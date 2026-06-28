# ElasticSearch 使用

- Rest High-Level Client - 推荐
    
    > RestClient 是 ElasticSearch 的基于 Rest API 的 Java 客户端，用于和 ElasticSearch 进行交互。它的本质是组装 DSL 语句，通过 HTTP 请求发送至 ElasticSearch 服务端实现交互。
    > 
    > 
    > RestClient 是低级别客户端，主要提供了 HTTP 通信客户端功能，而 Rest High-Level Client 是基于 RestClient 高级客户端，简化了与Elasticsearch的交互，提供了更高级和更抽象的 API，无需处理底层 HTTP 的细节。它支持同步和异步两种操作方式，支持索引管理、文档管理和检索等。
    > 
    - 使用
        - 引入依赖
            
            ```xml
            <dependency>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-high-level-client</artifactId>
            </dependency>
            ```
            
        - 操作索引
            
            通过 `org.elasticsearch.client.RestHighLevelClient#indices` 操作索引
            
        - 操作文档
            
            通过 `org.elasticsearch.client.RestHighLevelClient` 操作文档
            
    - 实战
        - 引入依赖
            
            ```xml
            <dependency>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-high-level-client</artifactId>
                <version>7.17.3</version>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.17.0</version>
            </dependency>
            ```
            
        - 通过 kibana 加入 sample 航班数据
            
            要求查询出起点城市为 Venice，终点国家为 CN，平均票价升序排序的前 10 条航班数据
            
            ![Untitled](ElasticSearch%20%E4%BD%BF%E7%94%A8/Untitled.png)
            
        - DSL 查询
            
            ```json
            GET /kibana_sample_data_flights/_search
            {
              "query": {
                "bool": {
                  "must": [
                    {
                      "term": {
                        "OriginCityName": {
                          "value": "Venice"
                        }
                      }
                    },
                    {
                      "term": {
                        "DestCountry": {
                          "value": "CN"
                        }
                      }
                    }
                  ]
                }
              },
              "from": 0,
              "size": 10,
              "sort": [
                {
                  "AvgTicketPrice": {
                    "order": "asc"
                  }
                }
              ]
            }
            ```
            
        - RestClient 查询
            
            新增测试类，使用测试类请求
            
            ```java
            @SpringBootTest
            public class SearcherTest {
            
                @Test
                public void query() {
                    // 示例化 RestHighLevelClient 客户端对象
                    try (RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://110.42.239.193:9200")))) {
                        // QueryBuilders 构建工具类提供了多种构建查询条件构造器
                        // 通过 布尔类型的查询构造器 BoolQueryBuilder 组织多条件 bool 检索
                        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
                        boolQueryBuilder.must(QueryBuilders.termQuery("OriginCityName", "Venice"));
                        boolQueryBuilder.must(QueryBuilders.termQuery("DestCountry", "CN"));
            
                        // 通过 SearchSourceBuilder 构建和配置搜索，例如设置查询条件、排序、分页、高亮，聚合等。
                        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
                        searchSourceBuilder.query(boolQueryBuilder);    // 设置查询条件
                        searchSourceBuilder.from(0);                    // 设置分页
                        searchSourceBuilder.size(10);
                        searchSourceBuilder.highlighter(new HighlightBuilder().field("OriginCityName").field("DestCountry").requireFieldMatch(false)); // 高亮显示配置
                        searchSourceBuilder.sort("AvgTicketPrice", SortOrder.ASC); // 设置排序
                        searchSourceBuilder.trackTotalHits(true);  // 开启命中统计，用于计算匹配的总命中数
            
                        // 构建查询请求并指定索引
                        SearchRequest searchRequest = new SearchRequest("kibana_sample_data_flights");
                        searchRequest.source(searchSourceBuilder); // 指定查询配置
            
                        // 指定请求并获取结果
                        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
            
                        long cnt = searchResponse.getHits().getTotalHits().value; // 获取命中数
                        SearchHit[] hits = searchResponse.getHits().getHits(); // 获取命中的结果集合
                        Arrays.stream(hits).forEach(hit -> {
                            System.out.println(hit.getSourceAsString());
                            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
                            System.out.println("高亮字段获取 OriginCityName：" + highlightFields.get("OriginCityName").getFragments()[0].string());
                        });
            
                    } catch (Exception e)  {
                        e.printStackTrace();
                    }
                }
            }
            ```
            
            过程：
            
            - 创建 SearchRequest 对象并指定索引名称；
            - 通过 SearchSourceBuilder 构建查询条件（通过SearchSourceBuilder#query(QueryBuilder) 方法指定查询条件，查询条件由 QueryBuilders 构造器类构建），设置结果处理方法（分页，排序，高亮）等；
            - 通过 SearchRequest#source(SearchSourceBuilder) 方法 将配置好的 SearchSourceBuilder 查询条件配置到 SearchRequest 对象中；
            - 将 SearchRequest 传入到 RestHighLevelClient#search(SearchRequest, RequestOptions) 方法中，用于发送请求。
            - 获取结果 SearchResponse 并解析
            - QueryBuilders 类
                
                > `QueryBuilders` 是 RestClient 的一个提供查询条件构造器的工具类，用于构建各种类型的查询，它包含很多静态方法，方法返回特定类型的查询构建器，这些构造器用于构建不同类型的查询条件。
                > 
                - Match Query：全文搜索
                    
                    ```java
                    MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("field", "value");
                    MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("value", "field1", "field2");
                    ```
                    
                - Term Query：精确匹配
                    
                    ```java
                    TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("field", "value");
                    ```
                    
                - Range Query：范围查询
                    
                    ```java
                    RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("field")
                                        .from("start_value")
                                        .to("end_value")
                                        .includeLower(true)  // 包含下界
                                        .includeUpper(false);  // 不包含上界
                    ```
                    
                - Bool Query：布尔查询，组合多个查询条件（must、should、must_not、filter）
                    
                    ```java
                    QueryBuilders.boolQuery()
                                 .must(QueryBuilders.matchQuery("field1", "value1"))
                                 .should(QueryBuilders.termQuery("field2", "value2"))
                                 .mustNot(QueryBuilders.rangeQuery("field3").from("start_value").to("end_value"))
                                 .filter(QueryBuilders.termQuery("field4", "value4"));
                    ```
                    
        - RestClient 聚合
            
            ```java
                /**
                 * GET /hotel/_search
                 * {
                 *     "query": {
                 *         "term": {
                 *             "city": {
                 *                 "value": "上海"
                 *             }
                 *         }
                 *     },
                 *     "size": 0,
                 *     "aggs": {
                 *         "by_brands": {
                 *             "terms": {
                 *                 "field": "brand",
                 *                 "size": 20
                 *             }
                 *         }
                 *     }
                 * }
                 */
                @Test
                public void aggs() {
                    try (RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://110.42.239.193:9200")))) 
            
                        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("city", "上海");
                        // 通过 AggregationBuilders 工具类构建 聚合查询
                        TermsAggregationBuilder termsAggregationBuilder = AggregationBuilders.terms("by_brands")
                                .field("brand")
                                .size(20);
            
                        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
                        searchSourceBuilder.size(0);
                        // 过滤条件的聚合
                        searchSourceBuilder.query(termQueryBuilder);
                        searchSourceBuilder.aggregation(termsAggregationBuilder); // 指定聚合查询
            
                        SearchRequest searchRequest = new SearchRequest("hotel");
                        searchRequest.source(searchSourceBuilder);
            
                        SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);
            
                        // 解析结果
                        Aggregations aggregations = response.getAggregations();
                        Terms byBrands = aggregations.get("by_brands");
                        List<? extends Terms.Bucket> buckets = byBrands.getBuckets();
                        buckets.stream().forEach(bucket -> {
                            System.out.println(bucket.getKeyAsString() + ":" +  bucket.getDocCount());
                        });
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            ```
            
            结果解析说明：Aggregations#get 获取的结果和具体的聚合类型保持一致
            
            ![Untitled](ElasticSearch%20%E4%BD%BF%E7%94%A8/Untitled%201.png)
            
        - RestClient 智能提示
            - 创建索引
                
                ```json
                PUT /hotel
                {
                    "settings": {
                        "analysis": {                                         // 自定义分词器
                            "filter": {
                                "py": {
                                    "type": "pinyin",
                                    "keep_full_pinyin": false,
                                    "keep_joined_full_pinyin": true,
                                    "keep_original": true,
                                    "limit_first_letter_length": 16,
                                    "remove_duplicated_term": true,
                                    "none_chinese_pinyin_tokenize": false
                                },
                                "analyzer": {
                                    "text_anlyzer": {
                                        "tokenizer": "ik_max_word",
                                        "filter": "py"
                                    },
                                    "completion_analyzer": {
                                        "tokenizer": "keyword",
                                        "filter": "py"
                                    }
                                }
                            }
                        }
                    },
                    "mappings": {
                        "properties": {
                            "id": {
                                "type": "keyword",
                                "index": false
                            },
                            "name": {
                                "type": "text",
                                "analyzer": "text_anlyzer",        // 使用自定义分词器
                                "search_analyzer": "ik_smart",
                                "copy_to": "all"
                            },
                            "address": {
                                "type": "keyword",
                                "index": false
                            },
                            "price": {
                                "type": "integer"
                            },
                            "score": {
                                "type": "integer"
                            },
                            "brand": {
                                "type": "keyword",
                                "copy_to": "all"
                            },
                            "city": {
                                "type": "keyword"
                            },
                            "starName": {
                                "type": "keyword"
                            },
                            "business": {
                                "type": "keyword",
                                "copy_to": "all"
                            },
                            "location": {
                                "type": "geo_point"
                            },
                            "pic": {
                                "type": "keyword",
                                "index": false
                            },
                            "all": {
                                "type": "text",
                                "analyzer": "text_anlyzer",
                                "search_analyzer": "ik_smart"
                            },
                            "suggestion": {                                 // 新增智能提示字段 completion 类型
                                "type": "completion",
                                "analyzer": "completion_analyzer"           // 参与智能提示自动补全的词条没有必要分词，只需要拼音分词器处理即可
                            }
                        }
                    }
                }
                ```
                
            - RestClient 智能提示查询
                
                ```java
                    @Test
                    public void testSuggest() {
                        try (RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://110.42.239.193:9200")))) {
                
                            // 通过 SuggestBuilders 工具类构建 completion 智能提示查询
                            CompletionSuggestionBuilder completionSuggestionBuilder = SuggestBuilders.completionSuggestion("suggestion")
                                    .prefix("7天")   // 
                                    .skipDuplicates(true)
                                    .size(10);
                
                            SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
                            searchSourceBuilder.suggest(
                                    new SuggestBuilder()
                                            .addSuggestion("my_suggestion", completionSuggestionBuilder)
                            );
                
                            SearchRequest searchRequest = new SearchRequest("hotel");
                            searchRequest.source(searchSourceBuilder);
                
                            SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);
                
                            // 解析结果
                            Suggest suggest = response.getSuggest();
                            CompletionSuggestion mySuggestion = suggest.getSuggestion("my_suggestion");
                            List<CompletionSuggestion.Entry.Option> options = mySuggestion.getOptions();
                            for (CompletionSuggestion.Entry.Option option : options) {
                                System.out.println(option.getText().string());
                            }
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                ```
                
- Spring Data ElasticSearch
- 实战
    - 索引库的创建
        
        业务表结构如下
        
        ```sql
        create table tb_hotel
        (
            id        bigint       not null comment '酒店id'
                primary key,
            name      varchar(255) not null comment '酒店名称',
            address   varchar(255) not null comment '酒店地址',
            price     int          not null comment '酒店价格',
            score     int          not null comment '酒店评分',
            brand     varchar(32)  not null comment '酒店品牌',
            city      varchar(32)  not null comment '所在城市',
            star_name varchar(16)  null comment '酒店星级，1星到5星，1钻到5钻',
            business  varchar(255) null comment '商圈',
            latitude  varchar(32)  not null comment '纬度',
            longitude varchar(32)  not null comment '经度',
            pic       varchar(255) null comment '酒店图片'
        );
        ```
        
        设计索引库
        
        > 考虑属性、属性类型、是否索引、分词和分词器以及多字段综合索引等因素。
        > 
        
        ```sql
        PUT /hotel
        {
          "mappings": {
            "properties": {
              "id": {                      // 建议将 ID 设置为 keyword 类型 且 关闭索引
                "type": "keyword",            
                "index": false
              },
              "name": {
                "type": "text",
                "analyzer": "ik_max_word",
                "copy_to": "all"          // 通过 copy_to 将当前字段的值复制到另一个字段，用于多字段综合搜索
              },
              "address": {
                "type": "keyword",
                "index": false
              },
              "price": {
                "type": "integer"
              },
              "score": {
                "type": "integer"
              },
              "brand": {
                "type": "keyword",
                "copy_to": "all"
              },
              "city": {
                "type": "keyword"
              },
              "star_name": {
                "type": "keyword"
              },
              "business": {
                "type": "keyword",
                "copy_to": "all"
              },
              "location": {                      // 设置 geo_point 类型的地理位置点 - 经纬度
                "type": "geo_point"
              },
              "pic": {
                "type": "keyword",
                "index": false
              },
              "all": {
                "type": "text",
                "analyzer": "ik_max_word"
              }
            }
          }
        }
        ```