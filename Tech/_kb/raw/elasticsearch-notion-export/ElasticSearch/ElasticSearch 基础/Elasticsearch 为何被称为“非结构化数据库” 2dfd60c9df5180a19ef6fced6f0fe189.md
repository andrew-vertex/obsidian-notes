# Elasticsearch 为何被称为“非结构化数据库”

Elasticsearch 常被称为“非结构化数据库”，本质原因是 **Schema-on-read**。

虽然它支持 schema 定义即映射 Mapping 存在，但不是强约束写入（即数据写入时不需要严格的结构定义）。关系型数据库是 **Schema-on-write**。两者约束时机不同。

---

## **1. 直觉结论**

- ES **不是无 schema，**而是**弱 schema、延迟约束、读时解释**。
- RDB 是 **强 schema、写时校验**。

---

## **2. 关系型数据库的 Schema-on-write**

### **定义**

> 在写入数据之前，必须严格符合 schema。
> 

### **特征**

- 建表即定义字段类型
- 写入时强校验
- 不符合直接失败

```sql
CREATE TABLE user (
  id BIGINT,
  age INT
);

INSERT INTO user VALUES (1, 'abc'); -- 直接报错
```

### **本质**

- schema 是 **写入门禁**
- 数据 = 已被强制结构化

---

## **3. Elasticsearch 的 Schema-on-read**

### **定义**

> 数据先写入，结构在查询和解析时才被真正“理解”。
> 

### **关键点**

- mapping ≠ 强制 schema
- mapping 是 **解析规则**
- 写入阶段宽松

```json
PUT index/_doc/1
{
  "age": "18"
}
```

- 未定义 mapping → 动态推断
- 已定义 mapping → 尽量解析，不轻易拒绝

---

## **4. 为什么 mapping 不是强 schema**

### **4.1 dynamic mapping 自动生成**

```json
PUT index/_doc/1
{
  "name": "张三",
  "age": 18
}
```

ES 自动生成：

```json
"name": "text",
"age": "long"
```

没有建表步骤。

---

### **4.2 类型不一致时的处理逻辑**

```json
PUT index/_doc/2
{
  "age": "18"
}
```

- 能解析 → 转换成功
- 不能解析 → 该字段索引失败，文档仍可写入

这在 RDB 中不允许。

---

### **4.3 mapping 是“解释方式”，不是“写入规则”**

- text → 倒排索引
- keyword → 精确匹配
- date → 时间解析

mapping 决定：

- 如何分词
- 如何索引
- 如何排序

不是决定：

- 能不能写

---

## **5. Schema-on-write vs Schema-on-read 对比**

| **维度** | **Schema-on-write** | **Schema-on-read** |
| --- | --- | --- |
| 代表系统 | MySQL | Elasticsearch |
| 校验时机 | 写入前 | 查询时 |
| schema 作用 | 写入约束 | 解析规则 |
| 类型不符 | 失败 | 尽量容错 |
| 结构变化 | 成本高 | 成本低 |
| 适合场景 | 交易系统 | 搜索分析 |

---

## **6. 为什么 ES 更适合“非结构化 / 半结构化”**

### **典型数据形态**

- 日志
- 文档
- JSON 事件流
- 多版本字段

```json
{
  "userId": 1,
  "extra": {
    "a": 1,
    "b": "x",
    "c": [1,2,3]
  }
}
```

关系型：

- 拆表
- JSON 字段
- 丧失索引能力

ES：

- 原生支持
- 任意层级
- 动态字段

---

## **7. Schema-on-read 的代价**

### **7.1 错误延后暴露**

- 写入成功
- 查询时报错或结果异常

### **7.2 mapping 冲突是灾难**

- 一旦字段类型确定，无法修改
- 不同文档写入不同类型 → index 报废

---

## **8. 工程实践建议**

### **8.1 生产环境必须“半结构化治理”**

```json
{
  "dynamic": "strict"
}
```

- 禁止未知字段
- 防止字段爆炸

---

### **8.2 明确区分 text / keyword**

```json
"orgName": {
  "type": "text",
  "fields": {
    "keyword": { "type": "keyword" }
  }
}
```

- 查询用 text
- 排序聚合用 keyword

---

### **8.3 ES 的定位认知**

- **不是数据库替代品**
- 是 **搜索 + 分析引擎**
- 是 **读优化系统**

---

## **9. 一句话总结**

- RDB：**先定义世界，再允许数据存在**
- ES：**先接受世界，再尝试理解它**

---

## **10. 拓展阅读关键词**

- inverted index
- dynamic mapping
- mapping explosion
- strict dynamic
- index template
- schema evolution

---