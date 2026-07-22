---
type: reference
title: "Spring State Machine — 分布式落地与局限分析"
created: 2026-07-22
tags:
  - state-machine
  - spring
  - distributed-systems
  - architecture
status: distilled
---

# Spring State Machine — 分布式落地与局限分析

> [!NOTE] 定位与背景
> Spring State Machine（SSM）是 Spring 官方提供的轻量级状态机类库，其核心设计面向**单机 JVM 内存**。它并非不能用于分布式，但需要额外的工程改造，且在复杂分布式长任务场景下存在明显局限，了解这些边界才是做出合理选型的基础。

---

## 一、单机 SSM 的根本缺陷

SSM 默认把状态机对象存储在**单个 JVM 进程的内存**中。在多节点集群下，这会产生致命的状态一致性问题：

```
节点 A（内存状态：待付款）          节点 B（内存状态：待付款）
     │                                   │
     │ ← 用户发来"付款成功"事件（被分发到 A）
     ▼
节点 A（内存状态：已付款）          节点 B（内存状态：待付款）← 完全不知情！
```

后续如果"发货"请求被负载均衡分发到节点 B，状态机认为订单还在"待付款"，就会出现业务逻辑错误。

---

## 二、SSM 在分布式应用中的落地改造方案

### 改造一：状态持久化（核心，必做）

将 JVM 内存中的状态强制落盘，节点间通过共享数据库实现状态共享。

SSM 提供了官方 `StateMachinePersister` 接口支持以下存储：

| 存储方案 | 适用场景 |
| :--- | :--- |
| **Redis**（`RedisStateMachineContextRepository`） | 高频状态更新，追求低延迟恢复 |
| **JPA（MySQL / PostgreSQL）** | 对事务一致性要求高，需要数据库级别的 ACID 保证 |
| **MongoDB** | 状态快照结构复杂、非关系型场景 |

**工作模式**：每次状态事件到来时，节点执行：
```
① 从数据库加载当前状态上下文（反序列化）
② 在内存中临时创建状态机，完成状态流转
③ 将新状态序列化并写回数据库
④ 内存中的状态机对象立即销毁（GC 回收）
```
节点本身保持无状态，状态的真相始终在数据库中。

---

### 改造二：分布式锁防并发冲突（必做）

在高并发下，两个请求可能同时到达不同节点，并发读取同一任务的状态并尝试同时流转，导致**状态覆盖**。

**解决方案**：在调用 `stateMachine.sendEvent()` 之前，必须先通过分布式锁锁住 `Task ID`：

```java
// 使用 Redisson 分布式锁示例
RLock lock = redissonClient.getLock("sm:lock:" + taskId);
boolean acquired = lock.tryLock(3, 5, TimeUnit.SECONDS);
if (!acquired) {
    throw new IllegalStateException("获取状态机锁失败，请稍后重试");
}
try {
    // 1. 从 Redis 加载状态
    stateMachinePersister.restore(stateMachine, taskId);
    // 2. 发送事件（内存流转）
    stateMachine.sendEvent(event);
    // 3. 持久化回 Redis
    stateMachinePersister.persist(stateMachine, taskId);
} finally {
    lock.unlock();
}
```

---

### 改造三（了解即可，已过时）：合奏机制（Ensemble/ZooKeeper）

> [!WARNING] 此方案已被淘汰，不建议在新项目中使用

SSM 提供了 `StateMachineEnsemble` 机制，利用 **ZooKeeper** 将多个节点的 JVM 内存状态机**实时广播同步**：
- 节点 A 状态变更 → 写入 ZooKeeper 节点 → ZooKeeper 推送 Watch 事件 → 节点 B 收到后强制更新自身内存状态机

**为什么被淘汰**：
1. **脆弱**：网络脑裂时，ZooKeeper 集群失联，所有节点的状态机会进入不一致的"孤岛"状态，后果比不用更糟。
2. **多余**：改造一（数据库持久化 + 无状态节点）已经彻底解决一致性问题，无需再同步内存状态。
3. **Spring Cloud Cluster 已归档**：该功能依赖的 `spring-cloud-cluster` 项目早已不再维护。

---

## 三、为什么复杂分布式场景不推荐 SSM？

### 1. 分布式定时器不可靠（致命缺陷）

若在 SSM 中定义了延迟触发逻辑（如"30 分钟内未支付则自动取消"），SSM 底层使用的是 **JVM 内存定时器**（`ScheduledExecutorService`）。一旦运行该定时器的节点重启或漂移，这个定时任务就会**彻底消失**，再也不会触发。

相比之下，Temporal.io 的 `Workflow.sleep(Duration.ofMinutes(30))` 是持久化在服务端数据库中的，即使集群全量重启，定时依然精准有效。

### 2. 频繁 DB 序列化/反序列化性能开销大

改造一要求每次状态流转都要进行一次完整的数据库读写与对象序列化。对于步骤非常密集的 Agent 任务或复杂编排，这种 per-step I/O 模式会在高并发下成为明显的性能瓶颈。

### 3. 缺乏 Saga 分布式事务补偿能力

当任务流程包含跨微服务调用（如：步骤 A 扣款成功 + 步骤 B 扣库存失败），SSM 没有内置的 Saga 补偿机制，补偿逻辑（如调用退款接口）需要完全由开发人员手写，工程量大且容易出错。

---

## 四、适合 SSM 的场景（轻量选型）

SSM 并非一无是处，以下场景使用它依然是性价比最高的选择：

> [!TIP]
> 如果你的业务只是简单的**单实体状态流转**（如订单从"创建"到"已付款"到"已完成"），完全不需要引入 SSM 框架，更不需要 Camunda/Temporal。使用**数据库状态字段 + 乐观锁**是最简单、最稳定的方案：
>
> ```sql
> UPDATE `order`
> SET status = 'PAID', version = version + 1
> WHERE id = 123 AND status = 'UNPAID' AND version = :old_version;
> -- 影响行数为 0 则说明并发冲突，需重试
> ```

- 步骤数量少（5 步以内）
- 没有超长等待或跨日执行的定时器需求
- 不涉及跨多个微服务的 Saga 补偿
- 团队对 Spring 生态更熟悉，不希望引入新的中间件

---

## 五、相关笔记

- [[Camunda-工作流引擎详解]]
- [[Temporal-io-分布式工作流引擎详解]]
