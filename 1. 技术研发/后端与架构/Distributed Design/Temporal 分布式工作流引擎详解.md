---
type: reference
title: "Temporal.io — 分布式工作流引擎详解"
created: 2026-07-22
tags:
  - temporal
  - workflow-engine
  - distributed-systems
  - saga-pattern
  - event-sourcing
  - spring-boot
status: distilled
---

# Temporal.io — 分布式工作流引擎详解

> [!NOTE] 核心定位
> Temporal.io 是由 Uber 工程师开源的 Cadence 项目演进而来的**分布式持续执行工作流引擎**。它的核心理念是 **"Workflow as Code（代码即工作流）"**：工程师用普通的 Java/Go/Python 代码编写业务逻辑（包括循环、条件分支、异步等待），Temporal 在底层通过 **Event Sourcing（事件溯源）** 保证这段代码在任何服务器崩溃、节点漂移后都能从中断处精确恢复并继续运行。

---

## 一、核心设计哲学：持续执行（Durable Execution）

传统的分布式系统中，一旦进程崩溃，内存中的所有状态（局部变量、执行位置）都会丢失。你必须自己实现检查点（Checkpoint）、重试逻辑和 Saga 补偿。

Temporal 的设计目标是：**让开发者写代码时，假装服务器永远不会崩溃。**

### 工作原理：Event Sourcing（事件溯源）

```
[Workflow 代码执行]                    [Temporal Server]
      │                                      │
      │─── 调用 Activity（扣款）──────────► 记录事件: ActivityScheduled
      │                                      │
      │─── 服务器崩溃！内存丢失 ───────────  │
      │                                      │
      │ (服务器重启后)                        │
      │─── 加载 History Log ◄───────────── 从数据库读取所有历史事件
      │                                      │
      │─── 快进（Replay）代码 ──────────►   重放至崩溃前的执行位置
      │─── 发现 Activity 已记录完成 ────►   直接跳过，不重复执行
      │─── 继续往下执行下一步 ─────────►   从断点继续
```

**关键点**：Temporal 不保存内存变量的快照，而是保存所有的**事件历史（Event History）**。恢复时通过"重放（Replay）"代码来还原内存状态——重放过程中已完成的 Activity 会直接返回历史结果而不重新执行，保证幂等性。

---

## 二、核心概念

| 概念 | 说明 |
| :--- | :--- |
| **Workflow** | 定义业务流程的接口，实现类中写控制逻辑（循环/条件分支），**禁止**包含直接的网络 I/O 或随机数调用（因为 Replay 时需要结果确定性） |
| **Activity** | 定义具体的无状态操作单元（调用外部 API、读写数据库），**允许**包含所有 I/O，Temporal 保证其至少执行一次，并支持独立重试 |
| **Worker** | 运行在你的应用中的监听进程，从 Temporal Server 的 TaskQueue 中拉取任务并执行 Workflow/Activity |
| **TaskQueue** | Workflow 和 Activity 之间的虚拟通道，用于路由任务到特定类型的 Worker |
| **Signal** | 外部向运行中的 Workflow 发送异步消息（如"用户已支付，继续流转"）|
| **Query** | 外部查询运行中 Workflow 的当前状态（只读操作） |

---

## 三、核心特性

### 1. 可靠的分布式定时器（Durable Timers）

```java
// 等待 30 分钟，如果期间服务重启，计时依然有效，绝不丢失
Workflow.sleep(Duration.ofMinutes(30));
```

这行代码看似普通，但 Temporal 会把这个定时事件持久化到 Server 端数据库中，即使整个 Worker 集群全量重启，定时器也会精确地在 30 分钟后触发，继续运行后续代码。

### 2. 精细化的 Activity 重试策略

```java
ActivityOptions options = ActivityOptions.newBuilder()
    .setStartToCloseTimeout(Duration.ofSeconds(10))  // 单次执行超时
    .setRetryOptions(RetryOptions.newBuilder()
        .setInitialInterval(Duration.ofSeconds(1))   // 首次重试等待 1 秒
        .setMaximumInterval(Duration.ofMinutes(1))   // 最大重试间隔 1 分钟
        .setBackoffCoefficient(2.0)                  // 指数退避系数
        .setMaximumAttempts(5)                       // 最多重试 5 次
        .build())
    .build();
```

每个 Activity 可以独立配置重试策略，与 Workflow 的重试完全解耦。

### 3. Saga 分布式事务补偿

```java
@Override
public void startOrderProcess(String orderId) {
    // 使用 Saga 辅助类按序注册补偿回调
    Saga saga = new Saga(Saga.Options.newBuilder().setParallelCompensation(false).build());
    try {
        // 步骤 1：扣款，注册对应的补偿操作
        activities.deductMoney(orderId);
        saga.addCompensation(activities::refundMoney, orderId);

        // 步骤 2：扣库存，注册补偿操作
        activities.deductInventory(orderId);
        saga.addCompensation(activities::restoreInventory, orderId);

        // 步骤 3：发货
        activities.shipOrder(orderId);

    } catch (ActivityFailure e) {
        // 任何步骤失败，自动按逆序调用所有已注册的补偿（退款、还库存）
        saga.compensate();
        throw e;
    }
}
```

---

## 四、适合场景

> [!TIP] Temporal 的选型黄金准则
> 满足以下任一条件，优先考虑 Temporal：
> - 业务是**纯代码驱动**的微服务编排，无需给非技术人员展示可视化流程图
> - 任务运行时间很长（数小时、数天甚至数周），期间可能发生多次服务重启
> - 需要高度可靠的**分布式 Saga 事务补偿**（多步骤，任意步骤失败均需精准回滚）
> - 对高并发、高吞吐有强需求（Temporal Server 用 Go 开发，性能远超 Java 引擎）
> - AI Agent 任务编排，包含多轮 LLM 调用和工具调用的复杂循环流程

典型适用业务：跨系统金融转账链路、物流配送全链路状态追踪、AI Agent 长任务执行、系统批量初始化、SaaS 订阅生命周期管理。

---

## 五、Spring Boot 集成

### 1. Maven 依赖

```xml
<dependency>
    <groupId>io.temporal</groupId>
    <artifactId>temporal-sdk</artifactId>
    <version>1.24.0</version>
</dependency>
```

### 2. 定义 Activity 接口与实现

```java
// 接口：定义业务操作
@ActivityInterface
public interface OrderActivities {
    @ActivityMethod
    String deductMoney(String orderId, double amount);

    @ActivityMethod
    void shipOrder(String orderId, String txnId);

    @ActivityMethod
    void refundMoney(String orderId);   // 补偿 Activity
}

// 实现：包含真实的 I/O 操作
@Component
public class OrderActivitiesImpl implements OrderActivities {

    @Override
    public String deductMoney(String orderId, double amount) {
        // 调用支付服务，建议携带幂等键防止重复扣款
        return paymentClient.deduct(orderId, amount, "idempotent-" + orderId);
    }

    @Override
    public void shipOrder(String orderId, String txnId) {
        logisticsClient.createShipment(orderId, txnId);
    }

    @Override
    public void refundMoney(String orderId) {
        paymentClient.refund(orderId);
    }
}
```

### 3. 定义 Workflow 接口与实现

```java
// 接口
@WorkflowInterface
public interface OrderWorkflow {
    @WorkflowMethod
    void startOrder(String orderId, double amount);

    @SignalMethod
    void notifyPaymentConfirmed();  // 外部信号：用户支付确认

    @QueryMethod
    String getCurrentStatus();      // 外部查询：获取当前状态
}

// 实现（注意：禁止直接调用任何 I/O，必须通过 Activity 代理）
public class OrderWorkflowImpl implements OrderWorkflow {

    private String currentStatus = "INIT";

    // 配置 Activity 代理（含重试策略）
    private final OrderActivities activities = Workflow.newActivityStub(
            OrderActivities.class,
            ActivityOptions.newBuilder()
                    .setStartToCloseTimeout(Duration.ofSeconds(30))
                    .setRetryOptions(RetryOptions.newBuilder()
                            .setMaximumAttempts(3)
                            .build())
                    .build());

    @Override
    public void startOrder(String orderId, double amount) {
        this.currentStatus = "PAYMENT_PENDING";

        // 等待外部支付确认信号（最长等 1 小时）
        boolean confirmed = Workflow.await(Duration.ofHours(1), () -> "PAYMENT_CONFIRMED".equals(currentStatus));
        if (!confirmed) {
            this.currentStatus = "TIMEOUT_CANCELLED";
            return;
        }

        // Saga 补偿事务
        Saga saga = new Saga(Saga.Options.newBuilder().build());
        try {
            String txnId = activities.deductMoney(orderId, amount);
            saga.addCompensation(activities::refundMoney, orderId);

            activities.shipOrder(orderId, txnId);
            this.currentStatus = "SHIPPED";
        } catch (ActivityFailure e) {
            saga.compensate();
            this.currentStatus = "FAILED";
            throw e;
        }
    }

    @Override
    public void notifyPaymentConfirmed() {
        this.currentStatus = "PAYMENT_CONFIRMED";
    }

    @Override
    public String getCurrentStatus() {
        return this.currentStatus;
    }
}
```

### 4. 注册 Worker 并启动

```java
@Configuration
public class TemporalWorkerConfig {

    @Bean
    public WorkflowServiceStubs workflowServiceStubs() {
        return WorkflowServiceStubs.newLocalServiceStubs();
    }

    @Bean
    public WorkflowClient workflowClient(WorkflowServiceStubs stubs) {
        return WorkflowClient.newInstance(stubs);
    }

    @Bean(initMethod = "start")
    public WorkerFactory workerFactory(WorkflowClient client, OrderActivitiesImpl activitiesImpl) {
        WorkerFactory factory = WorkerFactory.newInstance(client);
        Worker worker = factory.newWorker("ORDER_TASK_QUEUE");

        // 注册 Workflow 实现类
        worker.registerWorkflowImplementationTypes(OrderWorkflowImpl.class);

        // 注册 Activity 实现类（Spring Bean 可直接注入）
        worker.registerActivitiesImplementations(activitiesImpl);

        return factory;
    }
}
```

### 5. 发起 Workflow 实例

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final WorkflowClient workflowClient;

    public void createOrder(String orderId, double amount) {
        OrderWorkflow workflow = workflowClient.newWorkflowStub(
                OrderWorkflow.class,
                WorkflowOptions.newBuilder()
                        .setTaskQueue("ORDER_TASK_QUEUE")
                        .setWorkflowId("order-" + orderId)   // 业务唯一 ID，天然幂等
                        .build());

        // 异步发起（不阻塞当前线程）
        WorkflowClient.start(workflow::startOrder, orderId, amount);
    }

    public void confirmPayment(String orderId) {
        OrderWorkflow workflow = workflowClient.newWorkflowStub(
                OrderWorkflow.class,
                WorkflowOptions.newBuilder()
                        .setWorkflowId("order-" + orderId)
                        .build());
        // 发送 Signal
        workflow.notifyPaymentConfirmed();
    }
}
```

---

## 六、Camunda vs Temporal 选型对比

| 维度 | Camunda | Temporal.io |
| :--- | :--- | :--- |
| **流程定义** | 拖拽 BPMN 可视化图（XML） | 纯 Java/Go/Python 代码 |
| **面向受众** | 开发 + 产品 + 业务运营 | 纯开发团队 |
| **人工审批** | ⭐⭐⭐⭐⭐ 天然内置 Tasklist | ⭐⭐ 需通过 Signal 自行实现 |
| **Saga 补偿** | 通过 BPMN Boundary Event 配置 | 代码级精确控制 |
| **可靠定时器** | 依赖引擎数据库，较可靠 | ⭐⭐⭐⭐⭐ 事件溯源，极其可靠 |
| **高并发性能** | 中等（Java 引擎） | 极高（Go 引擎 + 水平扩容） |
| **学习曲线** | 中等（需学 BPMN 建模） | 较陡（需理解 Replay 机制与代码约束） |

---

## 七、相关笔记

- [[Spring StateMachine 分布式落地与局限]]
- [[Camunda-工作流引擎详解]]
