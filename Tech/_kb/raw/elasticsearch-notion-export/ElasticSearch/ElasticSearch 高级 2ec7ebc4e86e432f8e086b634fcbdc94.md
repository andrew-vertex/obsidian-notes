# ElasticSearch 高级

- ElasticSearch DSL 查询
    - DSL 查询类型
        - 地理查询
            
            ElasticSearch 地理查询，用于处理和搜索地理位置数据。其中 `geo_distance` 和 `geo_bounding_box` 是两种常见的地理查询方式，分别用于基于距离和基于边界框的查询。
            
            - `geo_distance`：基于地理距离查询，用于查找在指定点一定距离范围内（指定点的某个半径范围内）的文档，适用于查找附近的地点、用户位置等场景。
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled.png)
                
                ```json
                GET /indexName/_search
                {
                    "query": {
                        "geo_distance": {
                            "distance": "100km",    // 指定搜索的距离半径，单位包括km,m等
                            "FIELD": {              // 指定查询的字段并指定中心点的地理坐标（纬度 lat 和经度 lon） 
                                "lon": 120,
                                "lat": 30
                            }
                        }
                    }
                }
                ```
                
                示例
                
                ```json
                GET /hotel/_search
                {
                    "query": {
                        "geo_distance": {
                            "distance": "10km",
                            "location": {
                                "lon": 121.5,
                                "lat": 31.21
                            }
                        }
                    }
                }
                ```
                
            - `geo_bounding_box`：基于边界框的查询，用于查找指定矩形范围内的文档，适用于地理矩形范围内的搜索。
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%201.png)
                
                ```json
                GET /indexName/_search
                {
                  "query": {
                    "geo_bounding_box": {
                      "FIELD": {            // 指定边界框的两个对角点的地理坐标
                        "top_left": {          // 左上角的坐标（纬度 lat 和经度 lon）
                          "lat": 40.73,
                          "lon": -74.1
                        },
                        "bottom_right": {      // 右下角的坐标（纬度 lat 和经度 lon）
                          "lat": 40.01,
                          "lon": -71.12
                        }
                      }
                    }
                  }
                }
                ```
                
                示例
                
                ```json
                GET /hotel/_search
                {
                    "query": {
                        "geo_bounding_box": {
                            "location": {
                                "top_left": {
                                    "lon": 121,
                                    "lat": 31
                                },
                                "bottom_right": {
                                    "lon": 120,
                                    "lat": 30
                                }
                            }
                        }
                    }
                }
                ```
                
        - bool 查询
            
            bool 查询可以将多个查询条件的组合起来，实现多条件组合查询。通过 `must`、`filter`、`must not`、`should` 子句构建复杂的查询逻辑，其中 `must`、`should`  参与算分。注意：参与算分的条件（must, should子句）越多，查询的性能越差，因此要合理选择组合查询。
            
            | must（算分） | 查询条件必须匹配，相当于AND(逻辑与)。参与算分 | 文档必须满足的条件 |
            | --- | --- | --- |
            | filter | 查询条件必须匹配，通常用于过滤数据。不参与算分 | 过滤不满足条件的文档 |
            | must_not | 查询条件必须不匹配，排除在外，相当于NOT(逻辑非)。不参与算分 | 排除特定条件的文档 |
            | should（算分） | 选择性匹配，用于希望匹配的查询条件，相当于OR(逻辑或)，参与算分。（注：如果没有must，至少匹配一个should子句）。如果文档匹配 should 子句中的任意一个条件，它们会被认为是更相关的。should 子句通常用于增加匹配文档的评分。 | 可选条件，匹配条件会提升文档的相关性评分。 |
            
            示例：查询地点在“上海”，价格不高于300并且距离当前位置（31.2，121.5）不超过20km的酒店，心仪的酒店品牌为 "7天酒店"。
            
            ```json
            GET /hotel/_search
            {
                "query": {
                    "bool": {
                        "must": [
                            {
                                "term": {
                                    "city": "上海"
                                }
                            }
                        ],
                        "should": [
                            {
                                "term": {
                                    "brand": {
                                        "value": "7天酒店"
                                    }
                                }
                            }
                        ],
                        "must_not": [
                            {
                                "range": {
                                    "price": {
                                        "gte": 300
                                    }
                                }
                            }
                        ],
                        "filter": [
                          {
                            "geo_distance": {
                              "distance": "200km",
                              "location": {
                                "lat": 31.2,
                                "lon": 121.5
                              }
                            }
                          }
                        ]
                    }
                }
            }
            ```
            
            minimum_should_match 参数：用于控制在 bool 查询的 should 子句中必须匹配的最少查询条件数量。该参数允许更精细地控制查询的匹配逻辑，特别是在处理多个可选条件时。此外，minimum_should_match 还支持动态值，设置固定的百分比（必须匹配的条件数和总条件数的比值）根据 should 子句中的条件总数来计算必须匹配的条件数量。
            
            boost 参数：调整查询条件子句的权重影响文档相关性评分，用于增加或减少特定条件对最终评分的权重以影响最终评分，使得某些字段或条件在查询结果中更加重要。它作用于各种参与评分的查询子句。默认情况下，所有查询子句的权重相同，权重值都为1。
            
        - boosting 查询
            
            boosting 查询可以调整某些条件的评分权重，通过定义 `positive` 和 `negative` 查询条件实现该目的，并制定 `negative_boost` 参数来降低匹配 `negative` 条件的文档评分的程度，用于调整不同条件对评分的影响。
            
            ```json
            GET /my_index/_search
            {
              "query": {
                "boosting": {
                  "positive": {
                    "match": {
                      "my_text": "你好"
                    }
                  },
                  "negative": {
                    "match": {
                      "my_text": "世界"
                    }
                  },
                  "negative_boost": 0.2
                }
              }
            }
            ```
            
            positive: 指定匹配的查询条件，匹配这些条件的文档将获得正常的相关性评分。
            
            negative: 指定匹配的查询条件，匹配这些条件的文档将会被降低评分。
            
            negative_boost: 一个介于 0 和 1 之间的浮点数，用于指定在匹配 negative 查询条件的文档的相关性评分的降低程度。
            
        - function score 查询
            
            > Elasticsearch 的 function score 查询 通过函数对文档的相关性评分进行调整。它允许在标准查询的基础上，应用自定义的评分函数控制相关性算法以影响文档的相关性评分，更加灵活更加精确的控制搜索结果的排序。
            > 
            - 相关性算法
                
                在实现对搜索结果的排序时，ElasticSearch 通过 相关性算法 计算查询条件与匹配文档之间的相关性分数，根据相关性分数的分值进行排序。相关性分数的分值描述了 查询文档 和 查询条件 的匹配程度，分值越高，则说明文档越符合预期，排名越靠前。
                
                常见算法：
                
                - BM25 算法：ElasticSearch 5.x后默认使用的算法，它是一种基于概率的信息检索算法，通过考虑查询词在文档中的频率、文档的长度以及整个语料库中的文档频率来计算查询与文档之间的相关性分数。
                - TF-IDF 算法（`Term Frequency-Inverse Document Frequency`）：TF-IDF 算法通过 词项频率（TF）和 逆文档频率（IDF）来评估一个词在文档中的重要性。
                
                TF-IDF 算法会随着词频的增加而越来越大，而BM25 算法会随着词频的增加而增常曲线趋于平缓。
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%202.png)
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%203.png)
                
            - function score 的使用
                
                > function score query在基础查询（Query）检索匹配出文档的基础上，通过函数（Function）调整文档的相关性评分，控制搜索结果的排序。
                > 
                - 查询结构
                    
                    ```json
                    GET /indexName/_search
                    {
                      "query": {
                        "function_score": {
                          "query": {
                            // 基本查询
                          },
                          "functions": [              // 评分函数数组，通过评分还是影响原始算法
                            {
                              "filter": {             // 过滤器，用于过滤出评分函数适用的文档
                                     // 可选的过滤器
                              },
                              "weight": 1.2,          // 算分函数类型 weight，评分的权重常量值
                              "field_value_factor": { // 根据文档字段值调整评分
                                "field": "popularity",
                                "factor": 1.2,
                                "modifier": "sqrt",
                                "missing": 1
                              }
                            },
                            {
                              "random_score": {},     // 将随机值作为算分函数的结果，引入随机性，使得每次查询结果不同
                              "weight": 0.1
                            }
                          ],
                          "boost_mode": "multiply",  // 可选的加权模式；决定如何应用函数的评分结果
                          "score_mode": "sum"        // 可选的评分模式；当多个函数适用时，决定如何合并函数的评分结果
                        }
                      }
                    }
                    ```
                    
                    要素参数说明：
                    
                    - filter - 过滤条件，满足过滤条件的文档才会被算分函数影响
                    - 算分函数：用于计算出算法函数结果（function score），通过算分函数得到的结果即为 function score，function score将通过 boost_mode 加权模式来影响原始算法。
                        - weight：指定评分的权重常量值，即将指定的常量值作为算分函数的结果（function score）
                        - field_value_factor：将文档中的指定字段值作为算分函数的结果
                        - random_score：将将随机值作为算分函数的结果，用于引入随机性，使得每次查询结果不同
                        - script_score：自定义算分计算公式，将公式的结果作为算分函数的结果
                    - boost_mode：可选的加权模式，决定如何应用函数的评分结果，即如何将 function score 应用到原始算分上。
                        
                        ```
                        •	multiply: 默认值，函数评分（function score）与原始评分相乘。
                        •	replace: 使用函数评分替换原始评分。
                        •	sum: 原始评分与函数评分相加。
                        •	avg: 计算原始评分与函数评分的平均值。
                        •	max: 取原始评分与函数评分中的最大值。
                        •	min: 取原始评分与函数评分中的最小值。
                        ```
                        
                    - score_mode：可选的评分模式；当多个函数适用时，决定如何合并函数的评分结果 function score
                        
                        ```
                        •	multiply: 函数评分结果相乘。
                        •	sum: 函数评分结果相加。
                        •	avg: 计算函数评分结果的平均值。
                        •	first: 仅使用第一个函数的评分结果。
                        •	max: 取函数评分结果中的最大值。
                        •	min: 取函数评分结果中的最小值。
                        ```
                        
                - 查询示例
                    
                    查询出酒店名、品牌名、商圈包含 "外滩" 的文档并对品牌为 "7天酒店" 的文档进行算分加权。
                    
                    ```json
                    GET /hotel/_search
                    {
                        "query": {
                            "function_score": {
                                "query": {        // 定义原始查询条件匹配文档并得到原始算分 
                                    "match": {
                                        "all": "外滩"
                                    }
                                },
                                "functions": [    // 通过评分还是影响原始算法
                                    {
                                        "filter": {  // 过滤条件，符合条件的文档才会被影响算分
                                            "term": {
                                                "brand": {
                                                    "value": "7天酒店"
                                                }
                                            }
                                        },
                                        "weight": 2  // 算分权重常量值
                                    }
                                ],
                                "boost_mode": "multiply"
                            }
                        }
                    }
                    ```
                    
    - DSL 查询结果处理
        - 排序
            
            ElasticSearch 支持多种对搜索结果的排序方式，在不指定排序方式的情况下，默认是根据相关度算分（_score）排序。
            
            ElasticSearch 支持指定一个或多个字段进行排序，支持排序字段的类型包括keyword类型，数值类型，日期类型，地理坐标类型等。一旦指定排序字段，ElasticSearch 将会停用相关性算分，提升查询性能。
            
            ```json
            GET /indexName/_search
            {
              "query": {
                "match_all": {}
              },
              "sort": [             // 指定排序字段和排序方式(asc|desc)
                {
                  "FIELD": {        
                    "order": "asc"
                  }
                }
              ]
            }
            ```
            
            示例：酒店地理位置距离排序
            
            ```json
            GET /hotel/_search
            {
              "query": {
                "match_all": {}
              },
              "sort": [
                {
                  "_geo_distance": {
                    "location": {
                       "lat": 31.2,
                       "lon": 121.5
                    },
                    "order": "asc",
                    "unit": "km"
                  }
                }
              ]
            }
            ```
            
        - 分页
            
            ElasticSearch 默认只返回前10条数据，这是由于设置了默认的分页的参数。
            
            ElasticSearch 使用 from 和 size 参数实现分页。from 指定起始位置的偏移量即从第几条记录开始，默认为 0；size 指定返回的记录数，默认为 10。
            
            ```json
            GET /indexName/_search
            {
              "query": {
                "match_all": {}
              },
              "from": 1,
              "size": 5
            }
            ```
            
            - 深度分页问题
                
                [ElasticSearch 深度分页问题详情](ElasticSearch%20%E9%AB%98%E7%BA%A7/ElasticSearch%20%E6%B7%B1%E5%BA%A6%E5%88%86%E9%A1%B5%E9%97%AE%E9%A2%98%E8%AF%A6%E6%83%85%202fad60c9df518022af92cdd03c0d8de8.md)
                
                由于 ElasticSearch 底层结构采用倒排索引，这种结构无法直接分页，因此它的分页机制采用逻辑分页：它查询所有满足条件的文档，然后在根据 from 和 size 参数截取指定分页范围的文档。这种方式处理少量数据的分页时效率高，但是在处理深度分页时，可能会引入性能问题，尤其在分布式的场景下。
                
                ElasticSearch 支持分布式，当 ElasticSearch 采用集群部署时，同一索引的数据将分散到不同的分片中，此时当分页查询时，需要查询并排序出在每个分片上分页所有前面的文档；然后聚合不同分片上的结果，重新排序进行分页的截取。
                
                下图演示当按照 price 排序后，获取 from 为 990，size 为 10 的数据。
                
                - 从每个分片上获取检索并排序，得到前 1000 条文档
                - 将所有分片的上述结果聚合到内存中，在内存中重新排序出前 1000 条文档
                - 从这 1000 天文档中，截取从 990 开始的 10 条文档
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%204.png)
                
                上述过程中，如果搜索的页数过深或结果集过大，会对内存和cpu消耗过高，这就是深度分页问题（注：ElasticSearch 设定的结果集查询上限为 10000 即 from + size ≤ 10000）
                
                - 深度分页解决方案
                    - search after（推荐）：使用上次的查询结果的最后一条的排序值来查询 下一页的文档数据。
                        - 优点：没有查询上限，避免深度分页的性能问题。
                        - 缺点：只支持向后逐页查询，不支持随机翻页。
                        - 适用场景：没有随机翻页的搜索场景，例如滚动翻页。
                    - scroll（不推荐）：在初次查询时在内存中形成一个快照，后续的分页请求都会基于这个快照，避免每次分页查询都要进行聚合、排序和截取。
                        - 优点：没有查询上限。
                        - 缺点：快照消耗大量的内存，搜索的结果实时性差。
                        - 适用场景：海量数据的获取和迁移。
        - 结果高亮
            
            ElasticSearch 可以通过在查询请求中添加 `highlight` 选项设置搜索结果的高亮显示（为搜索结果的关键字添加标签标记），高亮显示可以帮助用户清晰地看到查询条件关键字和搜索结果的匹配情况。
            
            示例
            
            - 搜索查询设置高亮显示
                
                在查询请求中添加 `highlight` 部分来指定需要高亮显示的字段。注意，默认情况下，ElasticSearch 的搜索字段和高亮字段需要保持一致，可以通过 `"require_field_match": "false"` 关闭这一约束。
                
                ```json
                GET /hotel/_search
                {
                  "query": {
                    "match": {
                      "all": "如家"
                    }
                  },
                  "from": 0,
                  "size": 10,
                  "sort": [
                    {
                      "_geo_distance": {
                        "location": {
                           "lat": 31.2,
                           "lon": 121.5
                        },
                        "order": "asc",
                        "unit": "km"
                      }
                    }
                  ], 
                  "highlight": {                // 高亮显示配置
                    "fields": {
                      "name": {                 // 指定需要高亮显示的字段
                        "require_field_match": "false", 
                        "pre_tags": "<em>",
                        "post_tags": "</em>"
                      }
                    }
                  }
                }
                ```
                
            - 结果高亮
                
                ```json
                {
                  "took" : 4,
                  "timed_out" : false,
                  "_shards" : {
                    "total" : 1,
                    "successful" : 1,
                    "skipped" : 0,
                    "failed" : 0
                  },
                  "hits" : {
                    "total" : {
                      "value" : 30,
                      "relation" : "eq"
                    },
                    "max_score" : null,
                    "hits" : [
                      {
                        "_index" : "hotel",
                        "_type" : "_doc",
                        "_id" : "434082",
                        "_score" : null,
                        "_source" : {
                          "name" : "如家酒店·neo(上海外滩城隍庙小南门地铁站店)",
                          "address" : "复兴东路260号",
                          "price" : 392,
                          "score" : 44,
                          "brand" : "如家",
                          "city" : "上海",
                          "star_name" : "二钻",
                          "business" : "豫园地区",
                          "location" : {
                            "lon" : 121.498769,
                            "lat" : 31.220706
                          },
                          "pic" : "https://m.tuniucdn.com/fb2/t1/G6/M00/52/B6/Cii-U13eXLGIdHFzAAIG-5cEwDEAAGRfQNNIV0AAgcT627_w200_h200_c1_t0.jpg"
                        },
                        "highlight" : {
                          "name" : [
                            "<em>如家</em>酒店·neo(上海外滩城隍庙小南门地铁站店)"
                          ]
                        },
                        "sort" : [
                          2.3053783846826956
                        ]
                      },
                      .....
                    ]
                  }
                }
                ```
                
            - 高亮配置选项
                
                通过一些配置选项来自定义高亮显示的行为。
                
                - `pre_tags` 和 `post_tags` ：用于定义高亮标签，默认分别为 `<em>` 和 `</em>` ，建议使用与三方无关的自定义标签，保证不同平台的平台无关性。
                - `fragment_size` ：定义高亮片段的大小
                - `number_of_fragments` ：定义需要返回的片段数
                
                ```json
                POST /my_index/_search
                {
                  "query": {
                    "match": {
                      "content": "时间"
                    }
                  },
                  "highlight": {    
                    "fields": {
                      "content": {
                        "pre_tags": ["</strong>"],
                        "post_tags": ["</strong>"] // 自定义高亮标签
                      } 
                    }
                  }
                }
                ```
                
- Bucket 和 Metric 聚合分析
    
    > ElasticSearch 支持聚合分析，允许用户在数据上执行复杂的查询以获得统计信息和分析结果。聚合分析分为两类：桶（`bucket`）聚合和度量（`metric`）聚合以及管道（`pipeline`）聚合。
    > 
    > 
    > ElasticSearch 中可以参与聚合的字段类型主要包括数值型、日期型、关键词型keyword字段、布尔类型等，但是全文字段 text 类型不适合直接用于聚合，text 的索引方式为倒排索引，倒排索引结构无法进行高效的聚合计算。
    > 
    
    聚合分析的语法格式：
    
    ```json
    "aggregations": {                        // aggregations 聚合分析关键字 
      "<aggregation_name>": {                // 自定义聚合名称
        "<aggregation_type>": {              // 聚合的类型：terms, range, date_histogram, sum, avg等
          <aggregation_body>                 // 具体聚合的主体，包含特定聚合类型所需的详细定义
        },
        "meta": { <meta_data_body> }?,        // 可选的元数据
        "aggregations": { <sub_aggregation> }? // 可选的子聚合，可以在一个聚合内部定义其他聚合
      },
      "<aggregation_name_2>": {
        // 另一个聚合定义，可以包含多个同级的聚合查询，每个聚合都有自己的名称和定义
      }
    }
    ```
    
    - 聚合分析的类型
        - 桶聚合 - Bucket
            
            > 桶聚合是将文档分组到不同的桶中，每个桶代表一个满足特定条件的文档集合。桶聚合的作用是对数据进行分组。
            > 
            > 
            > 默认情况下，Bucket 桶聚合将会统计 Bucket 内的文档数量，记为 `_count`，并结果会按照 `_count` 降序排序，可以通过 order 属性修改排序字段和排序方式。
            > 
            > 默认条件下，桶聚合是对索引库中的所有文档做聚合分析，可以通过 `query 条件查询`对检索出的文档进行聚合分析即限定聚合的文档范围，
            > 
            - Terms - 术语：根据字段精确值进行分组，该字段通常为 keyword 类型，例如类型分组等。
                
                ```json
                {
                  "aggs": {
                    "by_category": {
                      "terms": {
                        "field": "category",         // 指定聚合字段d
                        "size": 10,                  // 指定聚合结果的返回数量
                        "order": {                   // 指定聚合结果的排序方式
                          "_count": "asc"
                        }
                      }
                    }
                  }
                }
                ```
                
            - Range - 范围：根据数值范围对文档进行分组，可以设置不同的区间进行分组。
                
                ```json
                {
                  "aggs": {
                    "sales_ranges": {
                      "range": {
                        "field": "sales",
                        "ranges": [
                          { "to": 100 },
                          { "from": 100, "to": 200 },
                          { "from": 200 }
                        ]
                      }
                    }
                  }
                }
                ```
                
            - Date Histogram - 日期直方图：根据时间间隔对文档进行分组。可以按照天，周，月等时间间隔分组。
                
                ```json
                {
                  "aggs": {
                    "sales_over_time": {
                      "date_histogram": {
                        "field": "date",
                        "calendar_interval": "month"
                      }
                    }
                  }
                }
                ```
                
            - Filter - 过滤器：使用多个过滤条件创建不同的桶，可以根据不同的条件创建多个组。
                
                ```json
                {
                  "aggs": {
                    "sales_filters": {
                      "filters": {
                        "filters": {
                          "electronics": { "term": { "category": "electronics" } },
                          "clothing": { "term": { "category": "clothing" } }
                        }
                      }
                    }
                  }
                }
                ```
                
            - Histogram - 阶段直方图：基于数值字段创建固定宽度的桶，可以按数值区间分组。
                
                ```json
                {
                    "aggs": {
                        "sales_histogram": {
                            "histogram": {
                                "field": "sales",
                                "interval": 50,
                                "extended_bounds": { // 统计边界
                                    "min": 0,
                                    "max": 1000
                                }
                            }
                        }
                    }
                }
                ```
                
        - 度量聚合 - Metric
            
            > 度量聚合是对文档数据进行基于字段值的数值汇总计算分析，计算结果通常为一个单一的值。度量聚合通常在桶聚合的基础上执行即在分组聚合的基础上作为子聚合执行，用于计算不同分组的统计信息。
            > 
            
            ```json
            {
              "aggs": {
                "sales_percentiles": {
                  "<metric_aggregation_type>": {
                    "field": "sales"
                  }
                }
              }
            }
            ```
            
            - Avg - 平均值：计算字段值的平均值
            - Sum - 求和：计算字段值的总和
            - Min - 最小值：计算字段值的最小值
            - Max - 最大值：计算字段值的最大值
            - Stats - 统计：计算一组统计值（平均值、最小值、最大值、总和和计数）
            - Extended - 扩展统计：计算扩展的统计值（例如方差、标准差）
            - Percentiles - 百分位数：计算字段值的百分位数
        - 管道聚合 - Pipeline
            
            > 管道聚合是对其他集合的结果进行聚合分析，它不直接作用于文档，而是作用于其他聚合的输出结果。
            > 
            - Derivative Aggregation：计算度量的导数
            - Cumulative Aggregation：计算度量的累积和
            - Moving Average Aggregation：计算度量的移动平均值
    - 桶聚合和度量聚合的结合
        
        首先使用桶聚合将文档分组到不同的桶中，然后对每个桶 Bucket 应用度量聚合来计算统计信息。
        
    - 聚合分析的性能优化
        
        `eager_global_ordinals` 选项：`eager_global_ordinals` 是一个用于优化聚合分析性能的选项，它主要作用于 keyword 类型的字段，表示指定字段将参与分组，提前对指定的字段进行预分组处理（如在文档操作时，将文档提前进行预分组），并将分组结果缓存，执行聚合查询时可以显著提高聚合查询的效率。
        
        全局序号是Elasticsearch用来对指定 keyword 字段的所有值进行排序的内部数据结构，默认情况下，全局序号是在第一次Bucket聚合计算时生成的，这会导致聚合查询产生一定的延迟。当启用 `eager_global_ordinals` 选项会在索引阶段预先计算并缓存全局序号（global ordinals），它可以避免第一次查询的延迟，使得术语聚合查询（terms aggregation）等操作的性能更加高效。
        
        `eager_global_ordinals` 的应用场景：对指定字段的聚合查询频繁时，可以预先计算全局序号减少查询时的延迟，提高查询的响应速度。适用于 高频聚合查询的大数据量的场景中。
        
        ```json
        PUT /my_tags
        {
          "mappings": {
            "properties": {
              "tags": {
                "type": "keyword",
                "eager_global_ordinals": true
              }
            }
          }
        }
        ```
        
    - 示例
        
        使用日期（按月）直方图将销售数据按月分组，然后对每个月的数据应用求和聚合来计算每月的销售总额。
        
        - 创建索引
            
            ```json
            PUT /my_sales
            {
                "mappings": {
                    "properties": {
                        "date": {
                            "type": "date"
                        },
                        "sales": {
                            "type": "double"
                        },
                        "category": {
                            "type": "keyword",
                            "eager_global_ordinals": true
                        }
                    }
                }
            }
            ```
            
        - 添加文档
            
            ```json
            POST /my_sales/_doc/1
            {
              "date": "2023-05-01",
              "sales": 100,
              "category": "electronics"
            }
            
            POST /my_sales/_doc/2
            {
              "date": "2023-05-02",
              "sales": 50,
              "category": "clothing"
            }
            
            POST /my_sales/_doc/3
            {
              "date": "2023-06-01",
              "sales": 50,
              "category": "other"
            }
            
            POST /my_sales/_doc/4
            {
              "date": "2023-06-03",
              "sales": 50,
              "category": "other"
            }
            ```
            
        - 聚合查询
            
            ```json
            GET /my_sales/_search
            {
              "size": 0,                  // 聚合查询返回的相关文档的页数，设置为0代表结果不显示文档数据，只包含聚合相关的结果
              "aggs": {
                "sales_per_month": {
                  "date_histogram": {     // 日期直方图桶聚合，根据date字段将文档按月分组
                    "field": "date",
                    "interval": "month",
                    "order": {
                      "monthly_sales.value": "asc"  // 指定排序字段和排序方式
                    }
                  },
                  "aggs": {               // 分组后的子查询
                    "monthly_sales": {    // 求和度量聚合，根号sales字段计算每个桶（即每个月）的销售总额
                      "sum": {
                        "field": "sales"
                      }
                    },
                    "cumulative_sales": { // 管道聚合，计算每个月的累计销售额
                      "cumulative_sum": {
                        "buckets_path": "monthly_sales"
                      }
                    }
                  }
                }
              }
            }
            ```
            
            ```json
            {
              "took" : 2,
              "timed_out" : false,
              "_shards" : {
                "total" : 1,
                "successful" : 1,
                "skipped" : 0,
                "failed" : 0
              },
              "hits" : {
                "total" : {
                  "value" : 4,
                  "relation" : "eq"
                },
                "max_score" : null,
                "hits" : [ ]
              },
              "aggregations" : {
                "sales_per_month" : {
                  "buckets" : [
                    {
                      "key_as_string" : "2023-06-01T00:00:00.000Z",
                      "key" : 1685577600000,
                      "doc_count" : 2,
                      "monthly_sales" : {
                        "value" : 100.0
                      },
                      "cumulative_sales" : {
                        "value" : 100.0
                      }
                    },
                    {
                      "key_as_string" : "2023-05-01T00:00:00.000Z",
                      "key" : 1682899200000,
                      "doc_count" : 2,
                      "monthly_sales" : {
                        "value" : 150.0
                      },
                      "cumulative_sales" : {
                        "value" : 250.0
                      }
                    }
                  ]
                }
              }
            }
            
            ```
            
- ElasticSearch Suggester 智能提示
    
    > ElasticSearch 的 Suggester 是一种用于自动补全和建议功能的机制，帮助用户在输入搜索查询时提供相关的建议。Suggester 可以实现自动补全、拼写纠正和搜索提示等功能，优化用户搜索体验。
    > 
    
    ElasticSearch 提供了三种类型的 Suggester。
    
    - Completion Suggester - 自动补全
        
        > `Completion Suggester` 是一种高效的自动补全机制，ElasticSearch 通过 Completion Suggester 实现基于**前缀查询**的实时自动补全（Auto-completion）功能的查询类型。
        > 
        > 
        > 注意参与自动补全查询的字段必须是 Completion 类型，字段值的内容一般是多个词条形成的词条数组。
        > 
        - 应用场景：常用于搜索框的智能搜索提示。
            
            Completion Suggester 可以快速提供基于**前缀**的建议，帮助用户输入查询词时获得即时提示。
            
        - 实现原理
            
            Completion Suggester 的底层原理基于 `前缀查找` 和  `FST 数据结构`。每当用户输入一个字符时，即时发送一个查询请求到 ElasticSearch，系统会基于 **前缀**  查找匹配项并返回建议。
            
            - 底层结构 - `FST` 数据结构（紧凑的有向无环图 DAG）：用于高效存储和前缀匹配的数据结构，它将单词的公共前缀共享，减少冗余节点，压缩存储空间和搜索的时间复杂度，它是 Completion Suggester 在大规模数据集上快速提供自动补全的关键。
                - 实现过程：当文档被构建索引时，包含 completion 的字段的内容会被解析并存储在一个 FST 数据结构中；当用户输入一个字符时，即时发送一个 `suggest` 查询请求，ElasticSearch 在 FST 中查找匹配该前缀的所有节点；最终的建议结果根据设置的权重进行排序，权重越高则排序越靠前。
        - 查询格式
            
            ```json
            POST /<index>/_search
            {
              "suggest": {
                "<suggest_name>": {          // suggest 查询自定义名
                  "prefix": "<prefix>",      // 输入的前缀
                  "completion": {            // suggest 类型
                    "field": "<field_name>", // 指定用于自动补全的 completion 类型字段
                    "size": <size>,          // 可选，返回建议的数量
                    "skip_duplicates": <true|false>  // 可选，是否开启去重
                  }
                }
              }
            }
            ```
            
        - 案例
            - 创建索引和定义映射
                
                定义 `completion` 类型字段的映射
                
                ```json
                PUT /my_suggest_index
                {
                    "mappings": {
                        "properties": {
                            "title": {
                                "type": "completion"
                            }
                        }
                    }
                }
                ```
                
            - 添加文档
                
                向索引中添加 `completion` 字段的文档：分别通过 `input` 和 `weight` 设置提示字段输入和提示权重
                
                ```json
                POST /my_suggest_index/_doc/1
                {
                  "title": {
                    "input": ["Elasticsearch", "Elastic", "Search Engine"], // 多字段输入：可以为一个文档的 completion 字段提供多个输入，用于提供更多的自动补全建议
                    "weight": 10 // 设置权重：每个输入项设置 weight，控制建议的排序。权重越高，优先级越高。
                  }
                }
                
                POST /my_suggest_index/_doc/2
                {
                  "title": {
                    "input": ["Elastic Stack", "Kibana", "Logstash"],
                    "weight": 5
                  }
                }
                ```
                
            - 使用 Completion Suggester 进行搜索
                
                > 使用 `suggest 查询` 来获取基于前缀的自动补全
                > 
                
                ```json
                POST /my_suggest_index/_search
                {
                  "suggest": {                     // 智能提示查询
                    "my_suggestion": {             // suggest 智能提示查询自定义名
                      "prefix": "Elastic",         // 用户输入的前缀，用于前缀查询
                      "completion": {              // 智能提示查询的类型
                        "field": "title",          // 指定用于自动补全的 completion 的字段
                        "skip_duplicates": true,   // 开启去重
                	      "analyzer": "ik_max_word", // 设置分词器
                        "size": 5                  // 获取结果的数量
                      }
                    }
                  }
                }
                ```
                
            - 解析响应结果
                
                ```json
                #! Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html to enable security.
                {
                  "took" : 0,
                  "timed_out" : false,
                  "_shards" : {
                    "total" : 1,
                    "successful" : 1,
                    "skipped" : 0,
                    "failed" : 0
                  },
                  "hits" : {
                    "total" : {
                      "value" : 0,
                      "relation" : "eq"
                    },
                    "max_score" : null,
                    "hits" : [ ]
                  },
                  "suggest" : {
                    "my_suggest" : [
                      {
                        "text" : "Elastic",
                        "offset" : 0,
                        "length" : 7,
                        "options" : [
                          {
                            "text" : "Elastic",
                            "_index" : "my_suggest_index",
                            "_type" : "_doc",
                            "_id" : "1",
                            "_score" : 10.0,
                            "_source" : {
                              "title" : {
                                "input" : [
                                  "Elasticsearch",
                                  "Elastic",
                                  "Search Engine"
                                ],
                                "weight" : 10
                              }
                            }
                          },
                          {
                            "text" : "Elastic Stack",
                            "_index" : "my_suggest_index",
                            "_type" : "_doc",
                            "_id" : "2",
                            "_score" : 5.0,
                            "_source" : {
                              "title" : {
                                "input" : [
                                  "Elastic Stack",
                                  "Kibana",
                                  "Logstash"
                                ],
                                "weight" : 5
                              }
                            }
                          }
                        ]
                      }
                    ]
                  }
                }
                ```
                
    - Term Suggester - 单词纠正
        
        > `Term Suggester` 提供基于输入的**单词**的拼写纠错和建议，它会基于每个单词编辑距离进行**模糊查询**，找到可能的拼写错误提供纠正建议，常用于查询的单词拼写错误纠正。
        > 
        - 查询格式
            
            ```json
            POST /<index>/_search
            {
              "suggest": {
                "<suggest_name>": {
                  "text": "<input_text>",    // 输入的内容
                  "term": {                  // suggest 类型
                    "field": "<field_name>", // 查询指定的字段
                    "size": <size>,          // 可选，返回建议的数量
                    "suggest_mode": "<suggest_mode>" // 可选，常用值：missing、popular、always
                  }
                }
              }
            }
            ```
            
            `suggest_mode` 选项：
            
            - missing：仅在输入的词在索引中不存在时才提供建议
            - popular：仅限索引中词频较高的词才能作为拼写建议
            - always：无论输入的词是否在索引中存在，都会提供拼写建议
        - 示例
            
            ```json
            POST /kibana_sample_data_ecommerce/_search
            {
              "suggest": {
                "my_suggest": {
                  "text": "Chandeer",
                  "term": {
                    "field": "customer_full_name",
                    "size": 10,
                    "suggest_mode": "always"
                  }
                }
              }
            }
            
            // 返回结果
            {
              "took" : 2,
              "timed_out" : false,
              "_shards" : {
                "total" : 1,
                "successful" : 1,
                "skipped" : 0,
                "failed" : 0
              },
              "hits" : {
                "total" : {
                  "value" : 0,
                  "relation" : "eq"
                },
                "max_score" : null,
                "hits" : [ ]
              },
              "suggest" : {
                "my_suggest" : [
                  {
                    "text" : "chandeer",
                    "offset" : 0,
                    "length" : 8,
                    "options" : [ // 建议选项
                      {
                        "text" : "chandler",
                        "score" : 0.875,
                        "freq" : 25
                      }
                    ]
                  }
                ]
              }
            }
            ```
            
    - Phrase Suggester - 短语纠正
        
        > `Phrase Suggester` 提供基于语言模型的**多词短语**的拼写纠正和建议，常用于基于整个短语提供更符合语义的短语的拼写纠正和建议。
        > 
        - 查询格式
            
            ```json
            POST /<index>/_search
            {
                "suggest": {
                    "<suggest_name>": {
                        "text": "<input_text>",
                        "phrase": {
                            "field": "<field_name>",
                            "size": <size> // 可选，返回建议的数量
                        }
                    }
                }
            }
            ```
            
        - 示例
            
            ```json
            POST /kibana_sample_data_ecommerce/_search
            {
              "suggest": {
                "my_suggest": {
                  "text": "eddie sammer",
                  "phrase": {
                    "field": "customer_full_name",
                    "size": 10
                  }
                }
              }
            }
            
            // 查询结果
            {
              "took" : 1,
              "timed_out" : false,
              "_shards" : {
                "total" : 1,
                "successful" : 1,
                "skipped" : 0,
                "failed" : 0
              },
              "hits" : {
                "total" : {
                  "value" : 0,
                  "relation" : "eq"
                },
                "max_score" : null,
                "hits" : [ ]
              },
              "suggest" : {
                "my_suggest" : [
                  {
                    "text" : "eddie sammer",
                    "offset" : 0,
                    "length" : 12,
                    "options" : [
                      {
                        "text" : "eddie samir",
                        "score" : 0.009672815
                      },
                      {
                        "text" : "eddie summers",
                        "score" : 0.0069724065
                      }
                    ]
                  }
                ]
              }
            }
            ```
            
    
    ElasticSearch 可以实现基于拼音字母的智能提示和自动补全功能，这可以通过拼音分词器或自定义分词器实现。
    
    - 拼音分词器
        
        [elasticsearch-analysis-pinyin](https://github.com/medcl/elasticsearch-analysis-pinyin) 是 ElasticSearch 拼音分词器插件，它可以实现对文档按照拼音进行分词。
        
        - 安装拼音分词器
            - 下载拼音分词器
                
                安装相应版本的[拼音分词器](https://github.com/infinilabs/analysis-pinyin/releases)
                
            - 上传至 ElasticSearch 插件挂载目录并解压
                
                ```bash
                unzip elasticsearch-analysis-pinyin-7.17.3.zip -d ./analysis-pinyin
                ```
                
            - 重启 ElasticSearch 容器
            - 测试
                
                ```bash
                GET /_analyze
                {
                  "text": ["分布式搜索引擎"],
                  "analyzer": "pinyin"
                }
                
                {
                  "tokens" : [
                    {
                      "token" : "fen",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 0
                    },
                    {
                      "token" : "fbsssyq",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 0
                    },
                    {
                      "token" : "bu",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 1
                    },
                    {
                      "token" : "shi",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 2
                    },
                    {
                      "token" : "sou",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 3
                    },
                    {
                      "token" : "suo",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 4
                    },
                    {
                      "token" : "yin",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 5
                    },
                    {
                      "token" : "qing",
                      "start_offset" : 0,
                      "end_offset" : 0,
                      "type" : "word",
                      "position" : 6
                    }
                  ]
                }
                
                ```
                
        - 拼音分词器问题
            
            拼音分词器默认的分词方式是每个字的全拼和所有字的拼音首字母的组合，因此存在不会分词并且不包含汉字的问题。
            
        - 分词器组成
            
            ElasticSearch 分词器（analyzer）由三部分组成，他们共同工作，将文本字段处理成合适搜索和索引的词项。
            
            - 字符过滤器（character filters）：在文本分词之前对文本进行预处理，例如删除字符，替换字符等。
                - html_strip：去除 HTML 标签
                - mapping：字符映射，将一个字符映射为另一个字符
                - pattern_replace：使用正则表达式进行模式替换
                
                ```json
                {
                    "char_filter": {
                        "my_html_filter": {
                            "type": "html_strip"
                        }
                    }
                }
                ```
                
            - 分词器（tokenizer）：将文本按照一定的规则切分为一个个词项（token)。它直接作用于文本并生成初步的词项列表。
                - standard：标准分词器；
                - ik：ik中文分词器
                - pinyin：拼音分词器
                
                ```json
                {
                    "tokenizer": {
                        "my_tokenizer": {
                            "type": "standard"
                        }
                    }
                }
                ```
                
            - 标记过滤器（token filter）：对分词器tokenizer生成的词项列表进一步处理，例如大小写转换、同义词处理、拼音处理等。
                - lowercase：将词项转换为小写
                - pinyin：拼音过滤器
                
                ```json
                {
                    "filter": {
                        "my_token_filter": {
                            "type": "lowercase"
                        }
                    }
                }
                ```
                
        - 自定义分词器
            
            自定义分词器就是对上述分词器的三个组成部分进行自定义设置的分词器。
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%205.png)
            
            - 创建自定义分词器
                
                在创建索引库时，通过 `settings` 来配置自定义的 analyzer 分词器（注：当前自定义分词器只对相应的索引有效）
                
                ```json
                PUT /my_index  
                {
                    "settings": {
                        "analysis": {
                            "filter": {                        // 自定义 token filter 配置
                                "py": {                        // 自定义 token filter 名称
                                    "type": "pinyin",          // 指定标记过滤器类型型为 pinyin，下面是各种属性设置
                                    "keep_full_pinyin": false,
                                    "keep_joined_full_pinyin": true,
                                    "keep_original": true,
                                    "limit_first_letter_length": 16,
                                    "remove_duplicated_term": true,
                                    "none_chinese_pinyin_tokenize": false
                                }
                            }, 
                            "analyzer": {                        // 自定义分词器配置
                                "my_analyzer": {                 // 自定义分词器名称
                                    "tokenizer": "ik_max_word",  // 自定义分词器 tokenizer 部分
                                    "filter": "py"               // 自定义分词器 token filter 部分
                                }
                            }
                        }
                    },
                    "mappings": {
                        "properties": {
                            "content": {
                                "type": "text",
                                "analyzer": "my_analyzer"        // 指定该字段使用索引配置的自定义分词器
                                "search_analyzer": "ik_smart"    // 自定义分词器建议在创建倒排索引时使用，建议不要在搜索时使用
                            }
                        }
                    }
                }
                ```
                
            - 测试
                
                ```json
                POST /my_index/_analyze
                {
                  "text": ["分布式搜索引擎"],
                  "analyzer": "my_analyzer"
                }
                
                {
                    "tokens": [
                        {
                            "token": "分布式",
                            "start_offset": 0,
                            "end_offset": 3,
                            "type": "CN_WORD",
                            "position": 0
                        },
                        {
                            "token": "fenbushi",
                            "start_offset": 0,
                            "end_offset": 3,
                            "type": "CN_WORD",
                            "position": 0
                        },
                        {
                            "token": "fbs",
                            "start_offset": 0,
                            "end_offset": 3,
                            "type": "CN_WORD",
                            "position": 0
                        },
                        {
                            "token": "分布",
                            "start_offset": 0,
                            "end_offset": 2,
                            "type": "CN_WORD",
                            "position": 1
                        },
                        {
                            "token": "fenbu",
                            "start_offset": 0,
                            "end_offset": 2,
                            "type": "CN_WORD",
                            "position": 1
                        },
                        {
                            "token": "fb",
                            "start_offset": 0,
                            "end_offset": 2,
                            "type": "CN_WORD",
                            "position": 1
                        },
                        {
                            "token": "式",
                            "start_offset": 2,
                            "end_offset": 3,
                            "type": "CN_CHAR",
                            "position": 2
                        },
                        {
                            "token": "shi",
                            "start_offset": 2,
                            "end_offset": 3,
                            "type": "CN_CHAR",
                            "position": 2
                        },
                        {
                            "token": "s",
                            "start_offset": 2,
                            "end_offset": 3,
                            "type": "CN_CHAR",
                            "position": 2
                        },
                        {
                            "token": "搜索引擎",
                            "start_offset": 3,
                            "end_offset": 7,
                            "type": "CN_WORD",
                            "position": 3
                        },
                    ....
                    }
                }
                ```
                
            - 建议
                
                为了避免在汉字搜索时搜索到同音字，在创建倒排索引时建议使用自定义分词器，在搜索时使用建议不要使用：用户用汉字搜索即用汉字分词搜索，用户用拼音字母即用拼音搜索，避免在用户用汉字搜索时，搜索到同音字。
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%206.png)
                
- ElasticSearch 索引别名
    
    > ElasticSearch 的 索引别名（Alias）是一个可以指向一个或多个索引（不推荐多个索引使用同一个别名）的逻辑名称，用于抽象化索引物理名称。
    > 
    > 
    > 通过 索引别名 使得客户端或应用程序无需直接操作具体的索引名称，而是使用别名进行索引操作。
    > 
    > 索引别名使索引的管理和维护过程更加灵活性和方便性。
    > 
    - 索引别名的应用场景
        1. 索引的升级和切换
        2. 数据迁移
        3. 索引的重建
    - 索引别名的操作
        - 创建别名
            
            > 为一个索引创建别名
            > 
            
            ```json
            POST /_aliases
            {
              "actions": [
                {
                  "add": {
                    "index": "index_1",
                    "alias": "my_index"
                  }
                }
              ]
            }
            
            // 或单一别名设置
            PUT /index_1/_alias/my_alias_index
            ```
            
        - 查看别名
            
            > 查看某个索引的所有别名
            > 
            
            ```json
            GET /index_1/_alias
            ```
            
        - 切换别名
            
            > 将别名从一个索引切换到另一个索引，先移除原始索引的别名，再为另一个索引新增别名
            > 
            
            ```json
            POST /_aliases
            {
                "actions": [
                    {
                        "remove": {
                            "index": "index_1",
                            "alias": "my_alias_index"
                        }
                    },
                    {
                        "add": {
                            "index": "index_2",
                            "alias": "my_alias_index"
                        }
                    }
                ]
            }
            ```
            
        - 设置带有过滤条件的别名
            
            > 为别名配置过滤条件，当通过该别名访问索引时，只返回满足条件的文档
            > 
            
            ```json
            POST /_aliases
            {
              "actions": [
                {
                  "add": {
                    "index": "index_2",
                    "alias": "my_alias_index", 
                    "filter": {
                      "term": {
                        "status": "active"
                      }
                    }
                  }
                }
              ]
            }
            ```
            
- ElasticSearch 重索引 reindex
    - 重索引 reindex 定义
        
        reindex - 重索引 是指将一个或多个索引中的数据重新索引（复制）到另一个索引中的过程。它通过 `_reindex` API 实现 从一个索引读取数据，并将其写入到另一个索引中。
        
    - 重索引的应用场景
        - 索引重建：变更索引的映射（mapping）或设置（settings）时，可以创建一个新的索引，使用重索引将旧索引的数据复制到新的索引中。
        - 数据迁移：将文档数据从一个索引迁移到另一个索引。
        - 索引拆分和索引合并
    - 利用 reindex 实现索引重建
        
        通过创建新的索引并重建索引 `reindex` 的方式进行索引映射的调整
        
        - 创建索引
            
            ```json
            // 创建旧索引
            PUT /old_index
            {
              "mappings": {
                "properties": {
                  "field1": {
                    "type": "text"
                  },
                  "field2": {
                    "type": "keyword"
                  }
                }
              }
            }
            
            // 向旧索引中新增文档数据
            POST /old_index/_search
            {
              "field1": "你好，世界",
              "field2": "世界，你好"
            }
            
            // 为旧索引设置别名 current_index
            POST /old_index/_alias/current_index
            
            // 创建新索引
            PUT /new_index
            {
              "mappings": {
                "properties": {
                  "field1": {
                    "type": "text"
                  },
                  "field2": {
                    "type": "text"
                  }
                }
              }
            }
            ```
            
        - 重索引：通过 `_reindex` 将数据从旧索引复制到新索引中
            
            ```json
            POST /_reindex
            {
              "source": { 
                "index": "old_index"
              },
              "dest": {
                "index": "new_index"
              }
            }
            
            // 查询验证重索引的数据
            GET /new_index/_search
            ```
            
        - 切换别名：别名切换至新索引，实现应用程序和数据管理的解耦
            
            ```json
            POST /_aliases
            {
              "actions": [
                {
                  "remove": {
                    "index": "old_index", 
                    "alias": "current_index"
                  }
                },
                {
                  "add": {
                    "index": "new_index",
                    "alias": "current_index"
                  }
                }
              ]
            }
            
            // 验证
            GET /current_index/_search
            ```
            
    - reindex 的 wait_for_completion 选项
        
        > 默认情况下，reindex 操作是以同步的方式运行的，一旦 reindex 操作开始，要等待操作运行完成，才能返回执行结果。`wait_for_completion=false` 选项一般用于重索引数据量较大的场景。
        > 
        
        通过设置参数 `wait_for_completion=false` 配置 reindex 操作为异步。当设置参数时，ElasticSearch 会立即返回一个 任务ID，这个任务对应后台异步执行的 reindex 操作。后续可以通过该 任务ID 监视 reindex 操作的进度和取消该任务。
        
        ```json
        POST /_reindex?wait_for_completion=false // 异步重索引
        {
          "source": {
            "index": "old_index"
          },
          "dest": {
            "index": "new_index"
          }
        }
        
        // 响应结果：任务ID（由节点ID和任务ID组成）
        {
          "task" : "g9A0oD1MS8GjmF3Zkjthvg:3480954" 
        }
        ```
        
        监视异步 reindex 操作的任务进度
        
        ```json
        GET /_tasks/g9A0oD1MS8GjmF3Zkjthvg:3480954
        ```
        
- ElasticSearch refresh 即时刷新
    - ElasticSearch 数据提交（commit）过程
        
        > ElasticSearch 数据提交（commit）分步分阶段处理，包含三个步骤：缓存 → 高速文件系统缓存 → 磁盘，它保证了数据的可用性和持久性。
        > 
        - ElasticSearch JVM 缓存 (In-Memory Buffer)
            
            文档被创建时，会先被写入到内存中开辟的缓存区，此时写入的文档对于其他线程是不可见的。
            
        - 操作系统文件系统高速缓存(Filesystem Cache)
            
            缓存中的文档数据将会被周期性（默认每隔一秒）的 `refresh` 刷新到文件系统的高速缓存中，此时这些文档对其他线程可见即其他线程可以查询到这些文档。
            
        - 磁盘 (Persistent Storage)
            
            文件系统高速缓存的文档数据将最终被 异步flush批量刷盘 到磁盘上，实现数据的持久化，默认情况下，每隔30分钟或当事务日志超过一定的大小触发。
            
    - 索引一致性问题
        
        在并发情况下，部分文档被线程写入，但是仍然处于内存缓冲区，导致其他线程不可见。
        
    - ElasticSearch refresh 操作和设置
        
        > ElasticSearch `refresh` 是一种即时刷新机制，用于将缓存中的最近文档变更信息写入到文件系统的高速缓存区，刷新后使得这些变更对其他线程的搜索可见。
        > 
        
        默认情况下，它会定期（默认每隔一秒）将内存中的缓冲区数据刷新到文件系统的高速缓存区中的段文件中。这使得新索引的数据在短时间内对搜索可见，一定程度上减少了可见性问题。
        
        - refresh 手动触发
            
            > refresh 支持手动触发
            > 
            
            ```json
            // refresh 指定文档操作
            PUT /my_index/_doc/1?refresh
            {
              "my_field": "你好，时间"
            }
            
            // refresh 指定索引
            POST /my_index/_refresh
            
            // 全局 refresh
            POST /_refresh
            ```
            
        - 刷新频率配置
            
            通过配置 `refresh_interval` 控制索引 `refresh` 的频率，`refresh` 时间间隔越短，`refresh` 频次越高，数据可见性提高，但是会影响IO性能。因此具体的设置需要权衡对可见性和性能的影响。
            
            ```json
            PUT /my_index/_settings
            {
              "refresh_interval": "2s"
            }
            ```
            
- ElasticSearch 集群
    
    > 单点的 ElasticSearch 存在很多问题，包括单点故障（高可用性问题）、海量数据存储、数据丢失风险（数据副本）、单点性能瓶颈等问题。因此建议采用多节点集群部署的方式，将索引拆分为多个分片存储到不同的节点上，同时配置副本分片将数据备份到不同到节点上。
    > 
    
    ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%207.png)
    
    - 集群部署
        
        > Docker 部署 ElasticSearch 集群
        > 
        > 
        > 在一台 CentOS 节点上使用 Docker 部署一个包含两个主节点 Master 的 ElasticSearch 集群，节点名称分别为 node1，node2。
        > 
        - Centos 调整系统参数 - `vm.max_map_count` 进程内存大小
            
            在 Centos 上安装启动 ElasticSearch 时，如果遇到  max virtual memory areas vm.max_map_count [65530] is too low 错误时，vm.max_map_count 表示一个进程最大拥有的内存区域大小，异常表示 ElasticSearch 需要更高的 `vm.max_map_count` 值，需要调整 max_map_count 值。
            
            - 编辑 sysctl.conf **文件**
                
                ```bash
                vim /etc/sysctl.conf
                ```
                
            - 在文件末尾添加参数
                
                ```bash
                vm.max_map_count=262144
                ```
                
            - 使配置生效
                
                ```bash
                sysctl -p
                ```
                
            - 验证配置是否生效
                
                ```bash
                sysctl vm.max_map_count
                ```
                
        - 创建挂载目录
            
            ```bash
            mkdir -p /mydata/elasticsearch/cluster/node1/data/
            mkdir -p /mydata/elasticsearch/cluster/node1/config/
            mkdir -p /mydata/elasticsearch/cluster/node1/logs/
            mkdir -p /mydata/elasticsearch/cluster/node1/plugins/
            ```
            
        - 编辑配置文件
            
            ```bash
            cat > /mydata/elasticsearch/cluster/node1/config/elasticsearch.yml <<- 'EOF'
            # 设置集群名称，集群内所有节点的名称保持一致
            cluster.name: "es-cluster"
            # 该节点是否允许作为主节点
            node.master: true
            # 该节点是否允许存储数据
            node.data: true
            network.host: 0.0.0.0
            # elasticsearch 对外提供服务端口
            http.port: 9200
            # 集群通讯端口
            transport.tcp.port: 9300
            # 集群中至少有数量的节点可用时，elasticsearch 才对外提供服务
            discovery.zen.minimum_master_nodes: 1
            # cors跨域
            http.cors.enabled: true
            http.cors.allow-origin: "*"
            http.cors.allow-headers: Authorization
            EOF
            ```
            
        - 创建 docker-compose.yml 文件
            
            ```bash
            cat > /mydata/elasticsearch/cluster/docker-compose.yml <<- 'EOF'
            version: '3'
            
            networks:
              es-cluster:
              
            services:
              es01:
                image: elasticsearch:7.17.3
                container_name: es01
                environment:
                  TZ: Asia/Shanghai
                  LANG: en_US.UTF-8
                  node.name: es01           # 设置集群中的 Elasticsearch 节点名
                  discovery.seed_hosts: es02   
                  cluster.initial_master_nodes: es01,es02 # 定义在集群启动过程中可能作为主节点候选的初始节点列表，用于完成主节点选举
                  TAKE_FILE_OWNERSHIP: true # 将 Elasticsearch 挂载数据文件的所有权更改为 Elasticsearch 用户，保证 Elasticsearch 拥有挂载目录的读写权限
                  ES_JAVA_OPTS: -Xms1024m -Xmx1024m
                  ELASTIC_PASSWORD: "123456" # Elasticsearch 账号密码
                volumes:
                  - /mydata/elasticsearch/cluster/node1/data:/usr/share/elasticsearch/data
                  - /mydata/elasticsearch/cluster/node1/logs:/usr/share/elasticsearch/logs
                  - /mydata/elasticsearch/cluster/node1/plugins:/usr/share/elasticsearch/plugins
                  - /mydata/elasticsearch/cluster/node1/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
                ports:
                  - 9201:9200
                  - 9301:9300
                networks:
                  - es-cluster
              es02:
                image: elasticsearch:7.17.3
                container_name: es02
                environment:
                  TZ: Asia/Shanghai
                  LANG: en_US.UTF-8
                  node.name: es02
                  discovery.seed_hosts: es01
                  cluster.initial_master_nodes: es01,es02
                  TAKE_FILE_OWNERSHIP: true # 将 Elasticsearch 挂载数据文件的所有权更改为 Elasticsearch 用户
                  ES_JAVA_OPTS: -Xms1024m -Xmx1024m
                  ELASTIC_PASSWORD: "123456" # Elasticsearch 账号密码
                volumes:
                  - /mydata/elasticsearch/cluster/node2/data:/usr/share/elasticsearch/data
                  - /mydata/elasticsearch/cluster/node2/logs:/usr/share/elasticsearch/logs
                  - /mydata/elasticsearch/cluster/node2/plugins:/usr/share/elasticsearch/plugins
                  - /mydata/elasticsearch/cluster/node2/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
                ports:
                  - 9202:9200
                  - 9302:9300
                networks:
                  - es-cluster
            EOF
            ```
            
        - 启动容器
            
            ```bash
            docker-compose up -d
            docker-compose ps
            ```
            
        - 检查集群状态
            
            ```bash
            curl -X GET "localhost:9201/_cat/nodes?v"
            ```
            
            master 为 * 的表示集群中的选举出来的主节点
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%208.png)
            
    - 集群主从
        
        在 Elasticsearch 集群中，节点可以扮演不同的角色，主要包含主节点（master node）和从节点（data node）。
        
        - 主节点 - master node
            - 作用：负责集群管理和协调任务，处理节点的加入和驱逐，决定分片分配，维护集群的状态更新。
            - 职责：负责集群管理和协调，确保集群的整体稳定性和一致性。
            - 配置：集群中至少需要一个主节点保证集群的正常运行，主节点的高可用性通过配置多个主节点实现，配置多个主节点以便在其中一个主节点失效时仍然可以选举新的主节点。
                
                ```yaml
                node.master: true
                node.data: false
                ```
                
        - 数据节点（从节点） - date node
            - 作用：主要负责数据存储，处理来自客户端的搜索请求和文档操作请求，负责分片的复制和恢复。
            - 职责：负责数据存储和处理，执行搜索和索引操作。
            - 配置：从节点通过分片和副本来保证数据的高可用性和容错能力。
                
                ```yaml
                node.master: false
                node.data: true
                ```
                
    - 集群监控
        
        通过 Kibana 监控 ElasticSearch 集群状态，包括内存占用，CPU负载等。
        
        - 创建挂载配置文件
            
            ```bash
            mkdir -p /mydata/kibana/cluster/config
            cat > /mydata/kibana/cluster/config/kibana.yml <<- 'EOF'
            server.name: kibana-es-cluster
            server.host: "0.0.0.0"
            elasticsearch.hosts: ["http://es01:9200", "http://es02:9200"]
            monitoring.ui.container.elasticsearch.enabled: true # 启用 Kibana 中的 Elasticsearch 监控 UI
            EOF
            ```
            
        - 启动 Kibana 容器
            
            ```bash
            docker run --network=cluster_es-cluster \
              -v /mydata/kibana/cluster/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
              -p 5602:5601 \
              --name kibana-es-cluster -d kibana:7.17.3
            ```
            
        - 打开 [Kibana](http://110.42.239.193:5602/app/home#/) 界面
            
            进入 `Management` 的 `Stack Monitoring` 查看集群的预览面板
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%209.png)
            
            点击 ElasticSearch 的 `Overview` 查看集群的总览，包含频率和延迟等信息。
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2010.png)
            
            查看 `Nodes` 节点的情况
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2011.png)
            
    - 集群分片
        - 分片（Shard）
            
            > 分片是 ElasticSearch 存储和管理数据的基本单元。在集群中，引入分片机制，使得索引数据可以分布在多个不同的数据节点上，实现水平扩展和负载均衡。一个索引可以被分解为多个分片，每个分片可以在不同的节点上独立存储和搜索。
            > 
            
            ElasticSearch 的分片机制是自动管理和组织分片，并对分片数据进行自动的平衡分配。
            
            ElasticSearch 分片分为主分片和副本分片，主分片是原始数据的存储组成，而副本分片是主分片的副本，为主分片提供数据冗余。
            
            - 主分片（Primary Shard）
                
                分片是原始数据的存储组成，用于读写操作数据。
                
                主分片一旦在索引创建时设定，则不允许变更主分片数量，则必须删除原先索引再重新创建
                
            - 副本分片（Replica Shard）
                
                副本分片是主分片的副本，用于数据复制和冗余，保证故障恢复和分散查询负载。
                
                副本分片数量可以实时的调整：
                
                ```json
                PUT /my_index/_settings
                {
                    "number_of_replicas": 2
                }
                ```
                
            
            在 ElasticSearch 7.x 版本后，如果不指定索引分片，它会默认创建为每一个索引创建一个主分片和副本分片。**默认情况下，集群中的同一索引的主分片和对应的副本分片不在同一个节点上。**
            
            - 配置
                - 默认情况
                    
                    默认情况下，集群中的同一索引的主分片和对应的副本分片不在同一个节点上。设置主分片和副本分片的数量。
                    
                    ```bash
                    PUT /my_index
                    {
                      "settings": {
                        "number_of_shards": 1, # 设置主分片数量
                        "number_of_replicas": 1 #设置每个主分片对应的副本分片数量
                      },
                      "mappings": {
                        "properties": {
                          "my_field": {
                            "type": "text"
                          }
                        }
                      }
                    }
                    ```
                    
                    ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2012.png)
                    
                - 自定义分片数量
                    
                    为一个索引设置3个主分片，为每个主分片设置1个副本分片 
                    
                    ```bash
                    DELETE /my_index
                    
                    PUT /my_index
                    {
                      "settings": {
                        "number_of_shards": 3,  # 设置主分片数量为 3
                        "number_of_replicas": 1 # 设置每个主分片对应的副本分片数量为 1 
                      },
                      "mappings": {
                        "properties": {
                          "my_field": {
                            "type": "text"
                          }
                        }
                      }
                    }
                    ```
                    
                    ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2013.png)
                    
    - 集群故障转移
        
        > ElasticSearch 集群中的主节点会监控集群中的节点的健康状态，如果检测到有节点宕机，会将该节点上的分片标记为不可用并将故障节点的主分片在其他节点上的对应的副本分片提升为主分片，保证高可用和数据安全。
        > 
        - 主节点故障转移
            
            当主节点宕机时，ElasticSearch 会从候选主节点中选举出新的主节点。
            
        - 主分片故障转移
            
            es01 故障后，Elasticsearch 定期检测节点的健康状态，检测到 es01 节点故障，会将该节点上的分片标记为不可用，并将其节点上的主分片在其他节点上的对应的副本分片提升为主分片。（如果索引主分片所在节点发生故障，Elasticsearch 会将其他节点上其对应的副本分片提升为主分片，保证了高可用）
            
            ```bash
            docker stop es01
            ```
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2013.png)
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2014.png)
            
        - 副本分片的故障转移
            
            同时，它也会在其他节点上创建缺失副本分片，实现分片平衡分配和分片的自动维护。
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2015.png)
            
            自动分配原理：当集群中的一个节点加入或离开时，Elasticsearch 会自动重新分配分片，Elasticsearch 会尝试将丢失的分片重新分配到集群中的其他健康节点，确保分片的均衡分布和高可用性，保证集群任意伸缩不出现故障。
            
        - 节点恢复
            
            当故障节点恢复运行时，分片会自动重新分配分片，保证分片的均衡分布。
            
            - 索引分片的转移
                
                通过 `/_cluster/reroute` 可以实现分片的重新路由（转移），将分片从一个节点移动到另一个节点。注意：主分片和它的副本分片不能在同一个节点上。
                
                ```json
                POST /_cluster/reroute
                {
                  "commands": [
                    {
                      "move": {                  // 转移
                        "index": "my_index",     // 指定索引
                        "shard": 0,              // 分片编号(指主分片)
                        "from_node": "node1",    // 当前所在节点
                        "to_node": "node0"       // 目标节点
                      }
                    }
                  ]
                }
                ```
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2016.png)
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2017.png)
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2018.png)
                
    - 集群状态管理和收集
        - 获取一系列获取集群和索引状态信息的API
            
            > _cat API 的设计目标是方便管理员快速获取集群的各种信息，返回的信息格式简洁明了。
            > 
            
            ```json
            GET /_cat
            ```
            
        - 查看集群健康状态
            
            获取所有索引的状态信息，包括健康状态、索引名称、文档数量、主分片和副本分片数量。
            
            ```json
            GET /_cat/health?v
            
            epoch      timestamp cluster    status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
            1719220327 09:12:07  es-cluster green           2         2     28  14    0    0        0             0                  -                100.0%
            ```
            
            - status 集群状态，red表示集群不可用，yellow表示基本可用但不可靠（单节点默认），green表示集群状态完全健康。
            - [node.total](http://node.total) 和 [node.data](http://node.data)：分别表示 节点总数 和 存储数据的节点数
            - shards：分片数
            - pri：主分片数
            - active_shards_percent：集群状态激活的分片百分比，加载分片数量达到指定百分比后，集群正常启动。
        - 查看集群的索引信息
            
            ```json
            GET /_cat/indices?v
            
            health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
            green  open   my_index                        QFeFp-T6R_6Nxm3F-GxMgg   3   1          0            0      1.3kb           678b
            ```
            
            - health：索引健康状态
            - status：索引激活状态
            - pri 和 rep： 主分片数和副本分片数
            - store.size 和 pri.store.size：索引存储的总容量和主分片的容量
        - 查看磁盘的分配情况
            
            查看每个节点的磁盘分配情况
            
            ```json
            GET /_cat/allocation?v
            
            shards disk.indices disk.used disk.avail disk.total disk.percent host       ip         node
                14       51.6mb   113.2gb     63.7gb      177gb           63 172.28.0.2 172.28.0.2 es01
                14       51.5mb   113.2gb     63.7gb      177gb           63 172.28.0.3 172.28.0.3 es02
            ```
            
            - shards：节点的分片数量
            - disk.indices、disk.used、disk.avail、disk.total：分别表示节点所有索引在磁盘所占空间、节点已经使用的磁盘容量、节点可用的磁盘容量，节点的磁盘容量
        - 查看集群的节点信息
            
            ```json
            GET /_cat/nodes?v
            
            ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
            172.28.0.2           12          98   5    0.25    0.28     0.31 cdfhilmrstw -      es01
            172.28.0.3           66          98   6    0.25    0.28     0.31 cdfhilmrstw *      es02
            ```
            
            - heap.percent：堆内存使用情况
            - ram.percent：运行内存使用情况
            - load_1m、load_5m、load_15m：过去1分钟、5分钟、15分钟设备的负载状态
            - cpu：CPU使用情况
            - master：是否为主节点
        - 集群的状态管理和收集
            
            ```json
            curl  -X GET http://localhost:9201/_cat/health?v >> dump.txt
            ```
            
    - 集群节点角色
        
        > ElasticSearch 集群中有不同的节点角色，每个角色有各自的职责划分。主节点负责集群管理，数据节点存储和处理数据，Ingest 节点进行数据预处理，协调节点负责请求的分发和聚合。
        > 
        > 
        > 在默认情况下，ElasticSearch 集群中的节点默认同时具备这四种角色，但是将节点角色进行合理配置，使得节点责任分明，可以优化 Elasticsearch 集群的性能和可靠性，同时不同角色使用不同的硬件设备以节约成本。
        > 
        - 集群节点分类
            - 主节点 - Master Node
                - 职责：主节点负责管理和协调集群的状态和操作，包括处理节点的加入和驱逐，分片的分配，索引库的操作，维护节点的健康状态等。
                - 配置：
                    
                    一个集群通常可以有多个候选主节点以保证高可用性，但同一时刻只有一个活跃的主节点。
                    
                    ```yaml
                    node.master: true # 默认为 true
                    ```
                    
            - 数据节点 - Date Node
                - 职责：数据节点负责存储和检索索引数据，处理来自客户端的数据相关的操作，比如搜索请求和文档操作请求
                - 配置：
                    
                    ```yaml
                    node.data: true # 默认为 true
                    ```
                    
            - Ingest 节点 - Ingest Node
                - 职责：Ingest 节点负责对文档数据进行预处理。在数据被索引之前执行一系列的通过 Ingest 管道（Ingest Pipeline）定义的预处理任务。
                - 配置：
                    
                    ```yaml
                    node.ingest: true # 默认为 true
                    ```
                    
            - 协调节点 - Coordinating Node
                - 职责：协调节点负责接收客户端请求并负载路由，分发请求到相应的数据节点，聚合结果并返回客户端。
                - 配置：
                    
                    每个节点都可以作为协调节点，但专门的协调节点可以优化集群的性能，尤其是在高查询负载的环境中。
                    
                    当上述三个参数配置都是 false 时，此时 节点为 专门的协调节点。
                    
                    ```yaml
                    node.master: false
                    node.data: false
                    node.ingest: false 
                    ```
                    
        - 集群节点分配
            
            集群部署时，建议每个节点有独立的角色，单独负责不同的职责，同时保证每个角色有多个节点，保证性能和  高可用性。  
            
            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2019.png)
            
    - 集群脑裂
        
        脑裂（split-brain）问题是分布式集群系统的常见问题，尤其在主节点选择过程中：默认情况下，每个节点（node.master: true）都可以是候选主节点，一旦 master 节点发生宕机，其他候选主节点会成为主节点。但是当主节点与集群发生网络分区时，不同分区的集群节点会从其他候选主节点中重新选择新的主节点，此时同一个集群出现多个主节点导致集群状态不一致，并且两个主节点在各自的网络分区中管理对应的数据节点，继续进行数据操作，造成数据不一致。当旧的主节点与集群网络恢复正常后，一个集群中会有两个主节点，并且发生了两边数据不一致的情况，这就是脑裂问题。
        
        解决方案：Elasticsearch 通过确保只有获得多数票（quorum）的节点才能当选为主节点来解决脑裂问题：只有选票超过 (集群候选节点数量+1)/2 的节点才可以当选为主节点，集群候选节点数量一般为奇数。这个规则保证了即使发生网络分区，也不会发生多个节点能够同时当选为主节点，此时只有一个分区可以获得超过半数的选票。
        
        多票数的配置参数为 discovery.zen.minimum_master_nodes，通过该指定集群中最少需要的主节点选票数量，以确保新主节点的选举遵循多数票规则。
        
        上述规则在 ElasticSearch 7.x后成为了默认规则，不再需要显式设置 minimum_master_nodes 参数，ElasticSearch自动管理这个参数，确保集群安全，因此ElasticSearch 7.x可以自动规避脑裂问题。
        
        ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2020.png)
        
- ElasticSearch 数据实时同步
    
    ElasticSearch 数据实时同步有很多应用场景，包括异构数据库（数据模型不同）同步、数据备份和数据分析等场景。
    
    MySQL 和 Elasticsearch 之间的数据同步问题通常泛指关系型数据库中的数据如何实时增量同步到 ElasticSearch 中，以便利用 Elasticsearch 的全文搜索和分析功能。
    
    - 常见的数据同步问题
        - 数据同步延迟：实时同步要求数据同步的及时性，避免同步延迟导致搜索结果不及时；
        - 数据一致性：在数据同步的过程中，需要保证异构数据源的数据一致性，避免数据丢失或重复；
        - 数据格式转换：异构数据源的数据模型不同，同步时需要对数据格式进行转换；
        - 错误情况处理：数据同步过程中可能会发生网络中断，数据冲突等情况，需要做好容错和回滚机制。
        
    - 常见的数据同步解决方案
        - 同步调用
            - 总结
                
                通过代码同步调用的方式：业务服务写入关系型数据库的同时，调用搜索服务更新ElasticSearch索引库的接口实现同步数据。
                
                优点：实现简单
                
                缺点：业务耦合；同步调用影响性能；无法保证事务问题；可扩展性差
                
        - Canal 监听 binlog 日志
            - Canal 简介
                
                > Canal 是阿里巴巴开源的一个工具，用于监听 MySQL 的 binlog 二进制日志（MySQL主从同步原理的关键），解析数据库增量日志，提供增量数据的订阅和消费，实现增量数据的实时同步，支持将数据同步至MySQL、Elasticsearch、HBase等异构数据库。
                > 
            - Canal 工作原理
                
                > Canal 是一个 基于 MySQL 二进制日志 binlog 的增量订阅和消费组件。
                > 
                
                Canal 通过模拟 MySQL 主库和从库的交互协议，将它自己伪装成MySQL的从库 slave，通过 MySQL binlog dump 协议订阅 binlog 日志，MySQL 主库收到dump请求会向canal推送binlog；Canal 解析 binlog 日志；提供数据的订阅和消费，提供实时的数据变更，推送到订阅其的目标端。
                
                ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2021.png)
                
            - Canal 使用
                
                > 以下配置和使用Canal，实现 MySQL数据实时同步至 ElasticSearch
                > 
                - 安装 Canal
                    
                    > 下载 [Canal](https://github.com/alibaba/canal/releases) 各个组件 `canal-server`、`canal-adapter`、`canal-admin`
                    > 
                    - `canal-server`: Canal 核心组件，负责连接MySQL主库并模拟成从库以监听 binlog 日志，接受数据并解析出binlog中的数据变更事件。其他客户端可以订阅其服务并获取数据变更事件进行处理（客户端可以是 MQ、ElasticSearch 等）。
                    - `canal-adapter`: Canal 拓展组件，也是 `canal-server` 的一个客户端组件，用于从 `canal-server` 中获取 `canal-server` 解析出的数据变更事件 并将这些事件进行转换并同步到数据源中，适用于异构的数据同步场景。
                        - 数据转换：将`canal-server` 输出的数据变更事件转换为目标系统需要的格式；支持自定义扩展以自定义数据转换逻辑，适应不同的业务需要
                        - 数据同步：将转换后的数据同步至目标系统
                    - `canal-admin`: Canal 管理组件，可视化界面用于集中管理和监控多个 `canal-server` 实例，提供集中配置管理、节点监控运维等功能。
                    
                    ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2022.png)
                    
                    ```yaml
                    version: '3'
                    
                    networks:
                      canal:
                    
                    services:
                      canal_admin:
                        image: canal/canal-admin:v1.1.5
                        container_name: canal_admin
                        restart: unless-stopped
                        volumes:
                          - /mydata/canal/canal-admin/conf/application.yml:/home/admin/canal-admin/conf/application.yml
                          - /mydata/canal/canal-admin/logs:/home/admin/canal-admin/logs
                        environment:
                          TZ: Asia/Shanghai
                          LANG: en_US.UTF-8
                          canal.adminUser: admin
                          canal.adminPasswd: 123456
                          spring.datasource.address: 110.42.239.193:3306
                          spring.datasource.database: canal_manager
                          spring.datasource.username: root
                          spring.datasource.password: 1qaz@WSX#EDC
                        ports:
                          - 8089:8089
                        networks:
                          - canal
                      canal_server:
                        image: canal/canal-server:v1.1.5
                        container_name: canal_server
                        restart: unless-stopped
                        volumes:
                          - /mydata/canal/canal-server/conf/canal_local.properties:/home/admin/canal-server/conf/canal_local.properties
                          - /mydata/canal/canal-server/logs:/home/admin/canal-server/logs
                        environment:
                          TZ: Asia/Shanghai
                          LANG: en_US.UTF-8
                          canal.register.ip: 110.42.239.193
                          canal.admin.manager: canal_admin:8089
                          canal.admin.port: 11110
                          canal.admin.user: admin
                          canal.admin.passwd: 6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9
                        ports:
                          - 11110:11110
                          - 11111:11111
                          - 11112:11112
                        depends_on:
                          - canal_admin
                        links:
                          - canal_admin
                        networks:
                          - canal
                      canal-adapter:
                        image: slpcat/canal-adapter:v1.1.5
                        container_name: canal_adapter
                        restart: unless-stopped
                        volumes:
                          - /mydata/canal/canal-adapter/conf/application.yml:/opt/canal-adapter/conf/application.yml
                          - /mydata/canal/canal-adapter/conf/es7:/opt/canal-adapter/conf/es7
                          - /mydata/canal/canal-adapter/logs:/opt/canal-adapter/logs
                        ports:
                          - 8081:8081
                        networks:
                          - canal
                    ```
                    
                    ```bash
                    docker-compose -f docker-compose.yml -p canal up -d
                    ```
                    
                - 配置 MySQL
                    
                    Canal 通过订阅 MySQL 的 binlog 日志实现数据同步，因此需要开启 MySQL 的 binlog 功能，并设置 binlog-format 模式为 ROW 模式。
                    
                    - 编辑配置文件并重启服务
                        
                        编辑配置文件 `/mydata/mysql/conf/my.cnf` ，`my.cnf` 是 MySQL 数据库服务器的配置文件，用于指定 MySQL 的各种参数。
                        
                        ```yaml
                        [mysqld]
                        pid-file        = /var/run/mysqld/mysqld.pid
                        socket          = /var/run/mysqld/mysqld.sock
                        datadir         = /var/lib/mysql
                        secure-file-priv= NULL
                        default-time-zone = 'Asia/Shanghai'           ## 指定MySQL服务器的默认时区
                        
                        log-bin         = /var/lib/mysql/tb-mysql-bin ## 启用binlog二进制日志并指定日志的基本路径和文件名前缀
                        binlog-do-db    = tb                          ## 指定记录到二进制日志的数据库
                        binlog_cache_size=1M                          ## 设置二进制日志缓存的内存大小
                        binlog_format=row                             ## 设置二进制日志格式，可选值 row、statement和mixed
                        expire_logs_days=0                            ## 设置二进制日志过期清理的天数，默认值为 0，表示不过期自动清理
                        slave_skip_errors=1062                        ## 设置主从复制跳过的错误，避免主从复制中断；错误代码1062表示主键重复
                        ```
                        
                        查看binlog是否启用
                        
                        ```sql
                        show variables like '%log_bin%'
                        +---------------------------------+-----------------------------------+
                        | Variable_name                   | Value                             |
                        +---------------------------------+-----------------------------------+
                        | log_bin                         | ON                                |
                        | log_bin_basename                | /var/lib/mysql/tb-mysql-bin       |
                        | log_bin_index                   | /var/lib/mysql/tb-mysql-bin.index |
                        | log_bin_trust_function_creators | OFF                               |
                        | log_bin_use_v1_row_events       | OFF                               |
                        | sql_log_bin                     | ON                                |
                        +---------------------------------+-----------------------------------+
                        ```
                        
                        查看binlog模式
                        
                        ```sql
                        show variables like 'binlog_format%';
                        +---------------+-------+
                        | Variable_name | Value |
                        +---------------+-------+
                        | binlog_format | ROW   |
                        +---------------+-------+
                        ```
                        
                    - 创建从库权限的账号
                        
                        创建一个拥有从库权限的账号，用于订阅 binlog 
                        
                        ```sql
                        CREATE USER canal IDENTIFIED BY 'canal';  
                        GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
                        FLUSH PRIVILEGES;
                        ```
                        
                    - 创建数据库和数据表
                        
                        ```sql
                        DROP DATABASE IF EXISTS tb;
                        CREATE DATABASE `tb` COLLATE utf8mb4_general_ci;
                        
                        DROP TABLE IF EXISTS `tb_hotel`;
                        CREATE TABLE `tb_hotel` (
                          `id` bigint NOT NULL COMMENT '酒店id',
                          `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '酒店名称',
                          `address` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '酒店地址',
                          `price` int NOT NULL COMMENT '酒店价格',
                          `score` int NOT NULL COMMENT '酒店评分',
                          `brand` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '酒店品牌',
                          `city` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '所在城市',
                          `star_name` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '酒店星级，1星到5星，1钻到5钻',
                          `business` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '商圈',
                          `latitude` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '纬度',
                          `longitude` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '经度',
                          `pic` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '酒店图片',
                          PRIMARY KEY (`id`) USING BTREE
                        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=COMPACT
                        ```
                        
                - 使用 `canal-server`
                    - 编辑 `canal-admin` 相关配置
                        
                        编辑配置文件 `/mydata/canal/canal-server/conf/canal_local.properties` ，指定 `canal-admin` 的地址、用户和密码等信息。
                        
                        ```sql
                        # tcp bind ip
                        canal.register.ip=110.42.239.193
                        
                        # canal admin config
                        canal.admin.manager=110.42.239.193:8089
                        canal.admin.port=11110
                        canal.admin.user=admin
                        canal.admin.passwd=6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9
                        ```
                        
                - 使用 `canal-adapter`
                    - 编辑适配器配置
                        
                        编辑 `/mydata/canal/canal-adapter/conf/c` 适配器配置文件，主要用于配置 `canal-server`、数据源和客户端适配器。
                        
                        ```yaml
                        server:
                          port: 8081
                        spring:
                          jackson:
                            date-format: yyyy-MM-dd HH:mm:ss
                            time-zone: GMT+8
                            default-property-inclusion: non_null
                        
                        canal.conf:
                          mode: tcp                                   # 指定客户端的模式，可选 tcp, kafka 或 rocketMQ
                          flatMessage: true                           # 控制是否以扁平化的 JSON 字符串形式投递数据，仅在kafka/rocketMQ模式下有效
                          zookeeperHosts:                             # 指定集群模式下的 zookeeper 地址，非集群模式下的该配置为空
                          syncBatchSize: 1000                         # 每次同步的批量大小
                          retries: 0                                  # 同步失败后的重试次数,0表示不重试,-1为无限重试
                          timeout:                                    # 同步的超时时间, 单位：毫秒
                          accessKey:                                  # 访问的密钥对
                          secretKey:
                          # 消费者配置
                          consumerProperties:
                            # canal tcp consumer
                            canal.tcp.server.host: 110.42.239.193:11111 # 设置 canal-server 地址
                            canal.tcp.zookeeper.hosts:                # 集群模式下的 zookeeper 地址
                            canal.tcp.batch.size: 500                 # 每次获取数据的批次大小
                            canal.tcp.username:                       # canal 的用户名和密码
                            canal.tcp.password:
                        #    # kafka consumer
                        #    kafka.bootstrap.servers: 127.0.0.1:9092
                        #    kafka.enable.auto.commit: false
                        #    kafka.auto.commit.interval.ms: 1000
                        #    kafka.auto.offset.reset: latest
                        #    kafka.request.timeout.ms: 40000
                        #    kafka.session.timeout.ms: 30000
                        #    kafka.isolation.level: read_committed
                        #    kafka.max.poll.records: 1000
                        #    # rocketMQ consumer
                        #    rocketmq.namespace:
                        #    rocketmq.namesrv.addr: 127.0.0.1:9876
                        #    rocketmq.batch.size: 1000
                        #    rocketmq.enable.message.trace: false
                        #    rocketmq.customized.trace.topic:
                        #    rocketmq.access.channel:
                        #    rocketmq.subscribe.filter:
                        #    # rabbitMQ consumer
                        #    rabbitmq.host:
                        #    rabbitmq.virtual.host:
                        #    rabbitmq.username:
                        #    rabbitmq.password:
                        #    rabbitmq.resource.ownerId:
                          # 源数据库配置
                          srcDataSources:
                            defaultDS:
                              url: jdbc:mysql://110.42.239.193:3306/tb?useUnicode=true
                              username: canal
                              password: canal
                          # 适配器列表
                          canalAdapters:
                          - instance: example                     # canal实例名 或 mq主题名，这里是 canal实例名 即 canal 同步任务单元名称
                            groups:                               # 分组列表
                            - groupId: g1                         # 分组id
                              outerAdapters:                      # 外部适配器
                              - name: logger                      # 日志打印适配器
                              - name: es7                         # ElasticSearch 同步适配器
                                hosts: 110.42.239.193:9200        # ElasticSearch 连接地址
                                properties:
                                  mode: rest                      # 连接模式模式，可选 transport(9300) 或者 rest(9200)
                                  cluster.name: elasticsearch     # ElasticSearch 集群名称
                                  security.auth: elastic:123456  # ElasticSearch 安全认证
                        ```
                        
                    - 编辑适配器转换配置
                        
                        定义转换配置，配置定义了数据源、目标、数据字段选择、同步策略等。它用于实现从源数据库 defaultDS 读取指定表的数据，然后通过 Canal 将数据同步到 ElasticSearch 的指定索引中。
                        
                        ```yaml
                        dataSourceKey: defaultDS        # 源数据源的 key, 对应 application.yml 配置中 srcDataSources 的一个值，表示数据从指定数据源读取
                        destination: example            # canal实例名或者MQ主题名
                        groupId: g1                     # 对应MQ模式下的groupId, 只会同步对应groupId的数据
                        esMapping:                      # ElasticSearch 映射配置部分
                          _index: hotel                 # 指定 ElasticSearch 索引名称
                          _id: _id                      # 指定 ElasticSearch 文档的 _id 字段，如果不配置该项，可以配置下面的pk项，ElasticSearch 则会自动分配 _id
                          _type: _doc                   # 指定 ElasticSearch 文档的类型
                          upsert: true                  # 指定更新文档时执行 "插入" 或 "更新" 的操作
                          # 用于数据映射的SQL查询语句，从源数据指定表和查询字段
                          sql: "SELECT id as _id, name, address, price, score, brand, city, star_name, business, CONCAT(latitude, ',', longitude) as location, pic FROM tb_hotel"
                        #  etlCondition: "where a.c_time>={}"   # ETL 全量同步 数据提取的过滤条件, 用于过滤数据，全量同步只同步创建时间（c_time）大于等于某个特定时间的数据。这个时间参数会在实际执行时填充
                          etlCondition: "where 1=1"     # 全量同步的过滤条件，指定全量同步过程中从源数据库提取数据的过滤条件 ## 全量同步命令 curl -X POST  http://110.42.239.193:8081/etl/es7/hotel.yml
                          commitBatch: 3000             #  提交批次大小
                        ```
                        
                    
                    上述两个配置文件协同工作，用于配置 `canal-adapter` 实现 MySQL 数据库到 ElasticSearch 的数据同步。
                    
                - 使用 `canal-admin`
                    
                    编辑配置文件 `/mydata/canal/canal-admin/conf/application.yml`，设置 `canal-admin` 的管理端元数据存储地址和可视化管理界面的登陆账号和密码。
                    
                    ```yaml
                    server:
                      port: 8089
                    
                    spring:
                      jackson:
                        date-format: yyyy-MM-dd HH:mm:ss
                        time-zone: GMT+8
                    
                    spring.datasource:
                      address: 110.42.239.193:3306
                      database: canal_manager
                      username: root
                      password: 1qaz@WSX#EDC
                      driver-class-name: com.mysql.jdbc.Driver
                      url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
                      hikari:
                        maximum-pool-size: 30
                        minimum-idle: 1
                    
                    canal:
                      adminUser: admin
                      adminPasswd: 123456
                    ```
                    
                    访问 [`canal-admin`](http://110.42.239.193:8089) 的可视化界面，输入账号密码 admin:123456
                    
                    - 查看和操作 `canal-server`
                        
                        ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2023.png)
                        
                    - 查看和管理 `canal 实例`
                        
                        > `canal 实例` 对应一个逻辑的同步任务单元，通常一个实例即一个同步任务单元对应于 MySQL 数据库实例。canal 实例配置指定数据抓取的数据库等信息，canal 一个任务单元实例和数据库建立连接，进行数据变更同步。
                        > 
                        
                        `canal 实例` 与 `canal-adapter` 中的 `canal.conf.canalAdapters[0].instance` 配置对应。
                        
                        ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2024.png)
                        
                        - 创建 canal 实例
                            
                            ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2025.png)
                            
                            ```yaml
                            #################################################
                            ## mysql serverId , v1.0.26+ will autoGen
                            # canal.instance.mysql.slaveId=0
                            
                            # enable gtid use true/false
                            canal.instance.gtidon=false
                            
                            # 指定需要数据同步的 MySQL 地址
                            canal.instance.master.address=110.42.239.193:3306
                            canal.instance.master.journal.name=
                            canal.instance.master.position=
                            canal.instance.master.timestamp=
                            canal.instance.master.gtid=
                            # 指定数据同步的数据库账号和密码
                            canal.instance.dbUsername=canal
                            canal.instance.dbPassword=canal
                            canal.instance.connectionCharset=UTF-8
                            # 指定订阅binlog的表的过滤正则表达式
                            canal.instance.filter.regex=.*\\..*
                            ```
                            
                - 同步数据
                    - 初次全量同步
                        
                        触发 `canal-adapter` 的 ETL 全量同步操作
                        
                        ```bash
                        curl -X POST  http://110.42.239.193:8081/etl/es7/hotel.yml
                        ```
                        
                    - 增量同步
            - 总结
                
                使用 Canal 监听 MySQL 的 binlog 二进制日志，一旦 binlog 变更则推送至订阅 Canal 的客户端。
                
                优点：服务零耦合
                
                缺点：开启 MySQL 的 binlog 日志功能，增加数据库负担；可靠性依赖于 Canal 组件；实现复杂。
                
        - MQ 异步通知
            - 总结
                
                基于消息队列实现异步通知：业务服务写入关系型数据库后，发布更新消息至消息队列MQ，搜索服务监听队列中的更新消息以实现数据同步。
                
                优点：低耦合
                
                缺点：引入消息中间件，依赖于 MQ 的可靠性
                
    - 架构设计
        
        Canal 负责监听 MySQL 增量数据，将监听到的数据变更事件推送至 MQ 消息队列，供下游服务订阅，不同的下游服务通过订阅关系来获取数据变更事件以同步数据，实现业务的解耦。
        
        ![Untitled](ElasticSearch%20%E9%AB%98%E7%BA%A7/Untitled%2026.png)
        
        引入消息队列的优势
        
        - 业务解耦：通过MQ使得业务解耦，不同异构服务通过订阅关系获取获取数据变更事件以同步数据，无需直接依赖，服务独立开发部署；
        - 缓冲同步压力：MQ 有缓冲作用，可以平衡 Canal 数据推送的高频次和下游系统处理能力之间的差异，防止下游服务因瞬间高并发数据写入而过载；
        - 可靠性：消息队列提供持久化机制，数据变更事件消息不会丢失，有很好的故障恢复机制；
        - 可扩展性：下游服务可以自由扩展即自由添加新的订阅者以扩展处理不同的数据同步需求。
        

[**Elasticsearch 评分机制与权重控制笔记**](ElasticSearch%20%E9%AB%98%E7%BA%A7/Elasticsearch%20%E8%AF%84%E5%88%86%E6%9C%BA%E5%88%B6%E4%B8%8E%E6%9D%83%E9%87%8D%E6%8E%A7%E5%88%B6%E7%AC%94%E8%AE%B0%2073e41e4416b24f45af83e0ba1f208b16.md)