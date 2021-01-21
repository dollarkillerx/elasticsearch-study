# Aggregations

### 基础概念
- Buckets 桶
    - 满足某个条件的文档集合
- Metrics 指标
    - 为桶中文档计算得到的统计信息
- Pipeline 
    - 聚合其他聚合的Output
- Matrix 矩阵聚合
   
#### 简单聚合
基础数据
```json
        {
            "_index" : "company_search",
            "_type" : "_doc",
            "_id" : "1047347700",
            "_score" : 1.0,
            "_source" : {
              "ni.id" : "1047347700",
              "ni.entity_type" : 109006022,
              "created_at" : 1594847299,
              "updated_at" : 1599760969,
              "founded_on" : 995385600,
              "is_public_company" : false,
              "district_location_id" : "0303010106",
              "district_location_name" : "北京市海淀区",
              "city_location_id" : "0303010100",
              "city_location_name" : "北京市",
              "province_location_id" : "0303010000",
              "province_location_name" : "北京",
              "keywords" : [
                "金融",
                "互联网保险"
              ],
              "investors" : [
                {
                  "investor_id" : "2000384269",
                  "investor_name" : "建银国际",
                  "investment_count" : 1
                }
              ],
              "deal_count" : 1,
              "raised_to_date_normalized" : 1500,
              "max_deal_size" : 1500,
              "deals" : [
                {
                  "deal_id" : "47930",
                  "size" : 1500,
                  "size_range" : "1000-2500",
                  "post_money_valuation" : 122000,
                  "closed_on" : 1487779200,
                  "time" : "2017-02-23",
                  "year" : "2017",
                  "quarter" : "2017Q1",
                  "deal_type" : 105002300,
                  "is_pre_exit" : true,
                  "is_pre_ipo" : true,
                  "is_pre_ma" : true,
                  "investor_ids" : [
                    "1025744634"
                  ],
                  "district_location_id" : "0303010106",
                  "district_location_name" : "北京市海淀区",
                  "city_location_id" : "0303010100",
                  "city_location_name" : "北京市",
                  "province_location_id" : "0303010000",
                  "province_location_name" : "北京"
                }
              ]
            }
      }
       
```
Search Query: 
```
GET /company_search/_search
{
  "aggs": {
    "search1": {  # search1 为当前查询命名
      "terms": {
        "field": "province_location_name",
        "size": 3 # [可选] 返回多少个数据 ,default: 10
      }
    }
  }
}
```
Response:
```json
{
  "took" : 113,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 1.0,
    "hits" : [
      ...
    ]
  },
  "aggregations" : {
    "search1" : {
      "doc_count_error_upper_bound" : 0, // 文档上的错误上限为每个术语计数，见下文
      "sum_other_doc_count" : 379418,    // 当存在大量唯一的术语时，Elasticsearch 只返回最上面的术语; 这个数字是不属于响应的所有存储桶的文档计数总和
      "buckets" : [
        {
          "key" : "广东省",
          "doc_count" : 150885
        },
        {
          "key" : "北京",
          "doc_count" : 108858
        },
        {
          "key" : "上海",
          "doc_count" : 92354
        }
      ]
    }
  }
}
```
##### Order
``` 
GET /company_search/_search
{
  "aggs": {
    "search1": {
      "terms": {
        "field": "province_location_name",
        "size": 20,
        "order": {
          "_count": "desc"  # 通过返回_count进行排序  
        }
      }
    }
  }
```

##### 统计 四川省 下 不同城市的公司个数
Search Query:
``` 
GET /company_search/_search
{
  "query": {
    "match": {
      "province_location_name": "四川省"
    }
  },
  "aggs": {
    "city": {
      "terms": {
        "field": "city_location_name",
        "size": 10
      }
    }
  }
}
```
Response:
``` 
    "buckets" : [
        {
          "key" : "成都市",
          "doc_count" : 26208
        },
        {
          "key" : "绵阳市",
          "doc_count" : 1334
        },
        {
          "key" : "南充市",
          "doc_count" : 588
        }
    ]
```

##### 统计各个城市  公司平均融资次数
Search Query:
``` 
GET /company_search/_search
{
  "aggs": {
    "city": {
      "terms": {
        "field": "city_location_name",
        "size": 100
      },
      "aggs": {
        "deal_count": {
          "avg": {
            "field": "deal_count"
          }
        }
      }
    }
  }
}
```
Response:
``` 
    "buckets" : [
        {
          "key" : "北京市",
          "doc_count" : 108857,
          "deal_count" : {
            "value" : 2.3953778888194877
          }
        },
        {
          "key" : "上海市",
          "doc_count" : 89356,
          "deal_count" : {
            "value" : 2.3273388702038575
          }
    }]
```

##### 桶中的桶 , 统计 各个城市 公司融资平均次数 和 各个地区个数
Search Query:
``` 
GET /company_search/_search
{
  "aggs": {
    "city": {
      "terms": {
        "field": "city_location_name",
        "size": 100
      },
      "aggs": {
        "deal_count": {
          "avg": {
            "field": "deal_count"
          }
        },
        "investors": {
          "terms": {
            "field": "district_location_name",
            "size": 10
          }
        }
      }
    }
  }
}
```
Response:
``` 
     {
          "key" : "成都市",
          "doc_count" : 26208,
          "deal_count" : {
            "value" : 2.1375046834020233
          },
          "investors" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 966,
            "buckets" : [
              {
                "key" : "成都市武侯区",
                "doc_count" : 3780
              },
              {
                "key" : "成都市金牛区",
                "doc_count" : 1962
              },
              {
                "key" : "成都市锦江区",
                "doc_count" : 1908
              },
              {
                "key" : "成都市青羊区",
                "doc_count" : 1737
              },
              {
                "key" : "成都市成华区",
                "doc_count" : 1415
              },
              {
                "key" : "成都市双流区",
                "doc_count" : 610
              },
              {
                "key" : "成都市郫都区",
                "doc_count" : 545
              },
              {
                "key" : "成都市温江区",
                "doc_count" : 422
              },
              {
                "key" : "成都市龙泉驿区",
                "doc_count" : 397
              },
              {
                "key" : "成都市新都区",
                "doc_count" : 386
              }
            ]
          }
    
```
#### Filters
Search Query:
``` 
PUT /logs/_bulk?refresh
{ "index" : { "_id" : 1 } }
{ "body" : "warning: page could not be rendered" }
{ "index" : { "_id" : 2 } }
{ "body" : "authentication error" }
{ "index" : { "_id" : 3 } }
{ "body" : "warning: connection timed out" }

GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
```
Response:
```json
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "messages" : {
      "buckets" : {
        "errors" : {
          "doc_count" : 1
        },
        "warnings" : {
          "doc_count" : 2
        }
      }
    }
  }
}
```
匿名过滤器
``` 
GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : [
          { "match" : { "body" : "error"   }},
          { "match" : { "body" : "warning" }}
        ]
      }
    }
  }
}
```

##### 嵌套查询
``` 
PUT /products
{
  "mappings": {
    "properties": {
      "resellers": { 
        "type": "nested",
        "properties": {
          "reseller": { "type": "text" },
          "price": { "type": "double" }
        }
      }
    }
  }
}

PUT /products/_doc/0
{
  "name": "LED TV", 
  "resellers": [
    {
      "reseller": "companyA",
      "price": 350
    },
    {
      "reseller": "companyB",
      "price": 500
    }
  ]
}

GET /products/_search
{
  "query": {
    "match": {
      "name": "led tv"
    }
  },
  "aggs": {
    "reselers": {
      "nested": {
        "path": "resellers"
      },
      "aggs": {
        "min_price": {
          "min": {"field": "resellers.price"}
        }
      }
    }
  }
}
```