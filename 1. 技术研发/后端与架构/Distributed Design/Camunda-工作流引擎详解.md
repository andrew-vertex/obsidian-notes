---
type: reference
title: "Camunda — 可视化 BPMN 工作流引擎详解"
created: 2026-07-22
tags:
  - camunda
  - bpmn
  - workflow-engine
  - distributed-systems
  - spring-boot
status: distilled
---

# Camunda — 可视化 BPMN 工作流引擎详解

> [!NOTE] 核心定位
> Camunda 是一款面向**业务流程自动化**的企业级工作流平台，基于 **BPMN 2.0（Business Process Model and Notation）** 国际标准。它最大的差异化优势是将"业务设计"与"工程实现"解耦：产品经理和业务分析师可以通过拖拽画图设计流程，工程师则只需编写各节点的执行逻辑，无需关注流程编排本身。

---

## 一、BPMN 2.0 是什么？

BPMN 2.0 是由 OMG（Object Management Group）制定的业务流程图标准，流程图以 XML 格式存储（`.bpmn` 文件），描述了从流程触发到终止的完整路径。

一个典型的 BPMN 流程图包含以下基本元素：

| 元素 | 图形 | 描述 |
| :--- | :--- | :--- |
| **Start Event** | 细实线圆圈 | 流程的触发点 |
| **Service Task** | 带齿轮图标的矩形 | 自动执行的技术步骤，调用代码逻辑 |
| **User Task** | 带人形图标的矩形 | 需要人工操作的步骤（如填表、审批） |
| **Gateway** | 菱形 | 条件分支或并行分叉 |
| **End Event** | 粗实线圆圈 | 流程的终点 |

---

## 二、Camunda 7 vs Camunda 8 架构对比

| 维度 | Camunda 7 | Camunda 8（当前主流） |
| :--- | :--- | :--- |
| **核心引擎** | 嵌入式（JAR 包形式嵌入应用） | **独立部署的 Zeebe 引擎**（基于 Raft 共识协议） |
| **状态存储** | 应用的关系型数据库（MySQL/PostgreSQL） | Zeebe 自管理的分布式日志（Raft Log） |
| **通信方式** | 引擎直接调用 Java 代码 | **gRPC/REST 客户端轮询（Job Worker 模式）** |
| **水平扩容** | 困难（引擎与 DB 强耦合） | 简单（Worker 无状态，可任意扩容） |
| **适用规模** | 中小型单体或简单微服务 | 大规模分布式微服务集群 |

---

## 三、核心特性

### 1. 可视化流程 Modeler
使用 **Camunda Modeler**（桌面端 IDE）可视化地绘制 `.bpmn` 流程图，然后通过 REST API 或 SDK 将流程部署到 Zeebe 引擎。流程图即是运行代码，业务人员可直接参与流程设计，真正实现"所见即所得"。

### 2. 人工任务（Human Task / User Task）天然支持
Camunda 内置了 **Tasklist**（任务列表 Web 应用），自动处理：
- 任务分配给指定用户或用户组
- 超时未处理自动升级（Escalation）
- 催办与转办
- 配套审批表单配置（Form Builder）

### 3. 流程版本控制（Process Versioning）
每次部署新版本的 `.bpmn` 流程图都会创建一个新的版本号（v1, v2, v3...）。**旧实例继续跑旧版本流程，新实例自动使用新版本**，天然支持生产环境灰度发布，业务流程迭代完全不影响在途数据。

### 4. Cockpit 监控看板
Camunda Cockpit 提供实时的流程实例监控：
- 查看每个流程实例当前卡在哪个节点
- 统计各节点的平均耗时、失败率
- 手动重试失败的步骤（无需重跑整个流程）

---

## 四、适合场景

> [!TIP] Camunda 的选型黄金准则
> 满足以下任一条件，优先考虑 Camunda：
> - 业务流程需要大量**人工审批节点**（多级审批、转签、催办）
> - 非技术团队成员（产品/运营）需要**直接查看和理解**流程运行状态
> - 需要满足**合规审计（Compliance Audit）**要求，每个步骤的审批人、时间、决策必须留痕
> - 流程逻辑相对稳定，变化频率不高

典型适用业务：报销审批、采购招标、合同会签、员工入职离职、保险理赔审批、信贷审核。

---

## 五、Spring Boot 集成（Camunda 8 / Zeebe）

### 1. Maven 依赖

```xml
<dependency>
    <groupId>io.camunda</groupId>
    <artifactId>spring-zeebe-starter</artifactId>
    <version>8.5.0</version>
</dependency>
```

### 2. application.yml 配置

```yaml
zeebe:
  client:
    broker:
      gateway-address: 127.0.0.1:26500  # Zeebe 引擎 gRPC 端口
    security:
      plaintext: true                    # 开发环境关闭 TLS
```

### 3. 部署 BPMN 流程文件

在 Spring Boot 应用中，将 `.bpmn` 文件放在 `src/main/resources/` 下，应用启动时自动部署：

```java
@SpringBootApplication
@EnableZeebeClient
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

### 4. 编写 Job Worker 实现 Service Task 逻辑

在 BPMN 图中，将某个 Service Task 的 `Task Type` 设为 `payment-deduct`；在代码中对应编写：

```java
@Component
public class PaymentWorker {

    @JobWorker(type = "payment-deduct")         // 与 BPMN 中 Task Type 保持一致
    public Map<String, Object> handlePayment(
            final ActivatedJob job,
            @Variable String orderId,            // 从流程变量中提取参数
            @Variable Double amount) {

        // 执行业务逻辑
        String txnId = paymentService.deduct(orderId, amount);

        // 返回值会自动注入回流程变量，供后续节点使用
        return Map.of("txnId", txnId, "paymentStatus", "SUCCESS");
    }

    @JobWorker(type = "payment-deduct", autoComplete = false)  // 手动完成，适用于需要异步回调的场景
    public void handlePaymentAsync(final JobClient client, final ActivatedJob job) {
        String orderId = (String) job.getVariablesAsMap().get("orderId");
        // 发起异步操作，由回调时手动调用 client.newCompleteCommand(job).send()
    }
}
```

### 5. 触发（发起）流程实例

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final ZeebeClient zeebeClient;

    public void startOrderProcess(String orderId, double amount) {
        zeebeClient.newCreateInstanceCommand()
                .bpmnProcessId("order-process")         // BPMN 文件中定义的 Process ID
                .latestVersion()
                .variables(Map.of("orderId", orderId, "amount", amount))
                .send()
                .join();
    }
}
```

---

## 六、相关笔记

- [[Spring-State-Machine-分布式落地与局限]]
- [[Temporal-io-分布式工作流引擎详解]]
