# Elasticsearch 评分机制与权重控制笔记

Elasticsearch 的评分机制基于 Lucene BM25。权重控制不在 Mapping 中完成，而在查询阶段完成。

---

> 目标：搞清 **ES 为什么这么排序，我如何精确控制排序结果**
> 

---

## **一、ES 评分机制总览**

### **1️⃣ 本质结论**

- ES 的排序本质是 **相关性评分**
- 默认排序字段：_score
- _score 由 **Lucene 相似度算法** 计算
- 当前默认算法：**BM25**

---

## **二、BM25 评分机制原理**

### **1️⃣ BM25 公式拆解**

BM25 由三部分构成：

```
score = TF × IDF × FieldNorm
```

### **① TF（Term Frequency）**

- 关键词在字段中出现次数
- 出现越多，分越高
- **边际递减**（防止刷词）

### **② IDF（Inverse Document Frequency）**

- 关键词越稀有，权重越高
- 常见词（如“公司”“保险”）权重低

### **③ FieldNorm（字段长度归一化）**

- 字段越短，命中越集中，分越高
- 标题 > 正文

---

### **2️⃣ BM25 的直觉理解**

> **“稀有词 + 出现得多 + 字段短“**
> 

---

## **三、权重到底能不能在 Mapping 里设置？**

### **结论**

❌ **不能直接在 Mapping 中设置评分权重**

### **原因**

- Mapping 决定：
    - 类型
    - 分词器
    - 是否索引
- **不参与相关性计算**

### **Mapping 能“间接影响评分”的地方**

| **能力** | **作用** |
| --- | --- |
| 分词器 | 决定 TF |
| keyword / text | 决定是否参与评分 |
| norms | 控制字段长度归一化 |
| index_options | 控制是否记录词频 |

---

### **示例：关闭 norms（不考虑字段长度）**

```json
"title": {
  "type": "text",
  "norms": false
}
```

适合：

- 编码类字段
- ID、编号

---

## **四、权重控制的正确方式（重点）**

### **权重控制只在**

### **Query 阶段**

核心手段只有 4 类：

1. boost
2. bool should
3. function_score
4. script_score / script sort

---

## **五、boost：最基础的权重控制**

### **1️⃣ 定义**

- 给 **字段** 或 **查询条件** 加权
- 值 > 1 提升权重
- 值 < 1 降低权重

---

### **2️⃣ 字段级 boost 示例**

```json
{
  "match": {
    "title": {
      "query": "理赔",
      "boost": 3
    }
  }
}
```

解释：

- title 命中比分 × 3

---

### **3️⃣ 多字段权重对比**

```json
{
  "multi_match": {
    "query": "理赔",
    "fields": [
      "title^3",
      "content^1"
    ]
  }
}
```

---

### **4️⃣ boost 的特点**

✅ 简单

❌ 无法使用业务字段

❌ 无法做复杂规则

---

## **六、bool + should：结构化权重**

### **1️⃣ should 的本质**

- 命中越多 should 条件，得分越高
- 每个 should 都会贡献 _score

---

### **2️⃣ 示例**

```json
{
  "bool": {
    "must": [
      { "match": { "content": "理赔" } }
    ],
    "should": [
      { "match": { "title": { "query": "理赔", "boost": 3 } } },
      { "term": { "status": "HOT" } }
    ]
  }
}
```

解释：

- title 命中 → 分数上升
- status=HOT → 再加分

---

### **3️⃣ 典型场景**

- 推荐系统
- 标签加权
- 热门标识

---

## **七、function_score：业务权重核心武器**

### **1️⃣ 作用**

> 在**文本相关性基础上，**融入**业务数值**
> 

例如：

- 热度
- 点击量
- 发布时间
- 评分

---

### **2️⃣ 基础结构**

```json
{
  "function_score": {
    "query": { ... },
    "functions": [ ... ],
    "score_mode": "sum",
    "boost_mode": "multiply"
  }
}
```

---

### **3️⃣ 常用函数类型**

| **函数** | **作用** |
| --- | --- |
| weight | 固定加权 |
| field_value_factor | 使用字段数值 |
| decay | 时间衰减 |
| script_score | 自定义公式 |

---

### **4️⃣ field_value_factor 示例**

```json
{
  "function_score": {
    "query": {
      "match": { "content": "理赔" }
    },
    "functions": [
      {
        "field_value_factor": {
          "field": "clickCount",
          "factor": 0.1,
          "modifier": "log1p"
        }
      }
    ],
    "boost_mode": "sum"
  }
}
```

---

### **5️⃣ decay 示例（时间衰减）**

```json
{
  "gauss": {
    "publishTime": {
      "origin": "now",
      "scale": "7d",
      "decay": 0.5
    }
  }
}
```

---

### **6️⃣ function_score 的优缺点**

✅ 工业级

✅ 性能可控

❌ 结构复杂

❌ 不适合写复杂逻辑

---

## **八、script_score：完全自定义评分**

### **1️⃣ 本质**

- 用脚本直接返回 _score
- 可访问 _score、字段值、参数

---

### **2️⃣ 示例**

```json
{
  "script_score": {
    "query": {
      "match": { "content": "理赔" }
    },
    "script": {
      "source": """
        _score * Math.log(1 + doc['clickCount'].value)
      """
    }
  }
}
```

---

### **3️⃣ 使用建议**

❌ 大规模排序慎用

❌ QPS 高场景慎用

✅ 小数据

✅ 高精度业务规则

---

## **九、script sort：完全绕过相关性**

### **1️⃣ 特点**

- **不使用 _score**
- 按脚本返回的数值排序

---

### **2️⃣ 示例**

```json
{
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "source": "doc['priority'].value * 10 + doc['clickCount'].value"
        },
        "order": "desc"
      }
    }
  ]
}
```

---

### **3️⃣ 场景**

- 规则排序
- 非搜索型列表
- 后台管理系统

---

## **十、四种方式对比总结**

| **方式** | **是否参与** _score | **复杂度** | **性能** | **推荐度** |
| --- | --- | --- | --- | --- |
| boost | 是 | 低 | 高 | ⭐⭐⭐⭐ |
| bool should | 是 | 中 | 高 | ⭐⭐⭐⭐ |
| function_score | 是 | 中高 | 中 | ⭐⭐⭐⭐⭐ |
| script_score | 是 | 高 | 低 | ⭐⭐ |
| script sort | 否 | 高 | 低 | ⭐ |

---

## **十一、工程级推荐组合（重要）**

### **搜索推荐标准方案**

```
BM25（文本）
+ boost（字段权重）
+ function_score（业务字段）
```

### **示例组合**

- title^3
- content^1
- clickCount log 加权
- publishTime 衰减

---

## **十二、你该如何记在 Notion 里**

### **建议结构**

```
ES 搜索排序
├─ BM25 原理
├─ Mapping 能做什么 / 不能做什么
├─ boost 用法
├─ bool should
├─ function_score（重点）
│   ├─ field_value_factor
│   ├─ decay
│   └─ score_mode / boost_mode
├─ script_score
└─ 排序方案选型
```

---

## **十三、关键认知纠偏**

- ❌ 想在 Mapping 里配权重
- ❌ 用 script_score 解决所有问题
- ❌ 只靠 _score 排序

---

如果你愿意，下一步我可以直接帮你：

- 按你 **当前业务（理赔 / 组织架构 / ES 搜索）** 设计一套 **评分公式**
- 或把你现有 ES 查询 **重构成 function_score 最优版本**

你现在最困惑的是哪一段。