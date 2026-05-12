# Search After 原理

Search After是一种基于上一页最后一行的排序值来检索下一页数据的方法。它要求查询必须指定一个唯一的排序字段（或者多个字段组合成唯一），通常是用`_id`（因为唯一）或者时间戳加上其他字段来确保唯一性和稳定性。

**具体过程：**

1. 第一次查询：指定排序字段（比如`_id`），返回第一页数据。
2. 获取最后一行的排序值（比如`_id`的值），在第二次查询时，将这些排序值作为`search_after`参数，这样就可以获取下一页数据。

**注意**：使用Search After时，跳页只能一页一页地往下翻，不能跳转到任意页面。而且，如果两次查询之间数据有变动（比如新增或删除），可能会影响分页结果。

```mermaid
sequenceDiagram
    participant C as Client
    participant E as Elasticsearch

    C->>E: 1. 第一次查询，指定排序字段，如按_id升序
    E->>C: 返回第一页数据，并记录最后一条的排序值（如id=10）

    C->>E: 2. 第二次查询，使用search_after=[10]，同样排序规则
    E->>C: 返回第二页数据，并记录最后一条的排序值（如id=20）

    C->>E: 3. 第三次查询，使用search_after=[20]，同样排序规则
    E->>C: 返回第三页数据，...
```

**实现过程**

```java
# 第一次查询，按_id升序（确保唯一排序），获取排序边界值
POST /my_index/_search
{
  "size": 100,
  "query": {
    "match_all": {}
  },
  "sort": [
    {"_id": "asc"}
  ]
}

# 第二次查询，使用search_after，参数为上一页最后一条的_id
POST /my_index/_search
{
  "size": 100,
  "query": {
    "match_all": {}
  },
  "sort": [
    {"_id": "asc"}
  ],
  "search_after": [100]  # 假设上一页最后一条的_id是100
}
```

**核心机制原理图**

```mermaid
flowchart TD
    A[客户端请求<br>search_after: 100] --> B{ES协调节点}
    
    B --> C[向所有分片广播查询]
    
    subgraph D[分片1处理]
        D1[接收查询条件<br>+ search_after值]
        D2{在本地排序数组中<br>二分查找order_id=100}
        D3[定位到位置100]
        D4[从位置100开始取10条]
        D5[返回10条数据]
        D1 --> D2 --> D3 --> D4 --> D5
    end
    
    subgraph E[分片2处理]
        E1[接收查询条件<br>+ search_after值]
        E2{在本地排序数组中<br>二分查找order_id=100}
        E3[定位到位置95]
        E4[从位置95开始取10条]
        E5[返回10条数据]
        E1 --> E2 --> E3 --> E4 --> E5
    end
    
    subgraph F[分片N处理]
        F1[接收查询条件<br>+ search_after值]
        F2{在本地排序数组中<br>二分查找order_id=100}
        F3[定位到位置105]
        F4[从位置105开始取10条]
        F5[返回10条数据]
        F1 --> F2 --> F3 --> F4 --> F5
    end
    
    D5 --> G[协调节点<br>归并排序所有分片结果]
    E5 --> G
    F5 --> G
    
    G --> H[返回第1000页数据<br>共10条]
    H --> I[客户端记录最后一条<br>排序值用于下一页]
```