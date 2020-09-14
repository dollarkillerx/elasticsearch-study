# elasticsearch-study
elasticsearch-study



|Relational DB|Elasticsearch|
| ---- | ----|
| 数据库（database）	| 索引（indices）| 
| 表（tables）	| 类型（types）| 
| 行（rows）	| 文档(document)| 
| 字段（columns）	| 字段（fields）| 


### RESTFul API ES交互 
`curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`

被 < > 标记的部件：

|部件名|	作用|
| ----| ----|
|VERB	|适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。|
|PROTOCOL	|http 或者 https（如果你在 Elasticsearch 前面有一个 https 代理）|
|HOST	|Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。|
|PORT	|运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。|
|PATH	|API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。|
|QUERY_STRING	|任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)|
|BODY	|一个 JSON 格式的请求体 (如果请求需要的话)|


|method	|url地址|	描述|
| ---- | ---- | ---- |
|PUT	|localhost:9200/indices/types/document_ID|	创建文档（指定ID）|
|POST	|localhost:9200/indices/types	|创建文档（随机ID）|
|POST	|localhost:9200/indices/types/document_ID/_update	|修改文档|
|DELETE	|localhost:9200/indices/types/document_ID	|删除文档|
|GET	|localhost:9200/indices/types/document_ID	|通过ID查询文档|
|POST	|localhost:9200/indices/types/_search	|查询所有的文档|

作者：Summer2077
链接：https://www.jianshu.com/p/0b25cecd0599
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

- 获取服务状态
``` 
GET /_cat/health?v
```
- Add  (存在会覆盖)
``` 
// /index/type/document
// /db/table/row

POST /db/user/1  
{
    "username": "wangdashuai",
    "nickname": "dollarkiller",
    "motto": "坎坷之路,始抵群星"
}

POST /db/user/2  
{
    "username": "go",
    "nickname": "gopher",
    "motto": "fuck java"
}
```
- Find
``` 
GET /db/user/2
```
- Delete
``` 
DELETE /db/user/2
```
- Update  会覆盖
``` 
PUT /db/user/2
{
    "username": "go",
    "nickname": "gopher",
    "motto": "fuck java"
}
```
- Search
准备假数据
```
PUT /movies/movie/1
{
  "title": "The Godfather",
  "director": "Francis Ford Coppola",
  "year": 1972,
  "genres": [
    "Crime",
    "Drama"
  ]
}

PUT /movies/movie/2
{
  "title": "Lawrence of Arabia",
  "director": "David Lean",
  "year": 1962,
  "genres": [
    "Adventure",
    "Biography",
    "Drama"
  ]
}

PUT /movies/movie/3
{
  "title": "To Kill a Mockingbird",
  "director": "Robert Mulligan",
  "year": 1962,
  "genres": [
    "Crime",
    "Drama",
    "Mystery"
  ]
}

PUT /movies/movie/4
{
  "title": "Apocalypse Now",
  "director": "Francis Ford Coppola",
  "year": 1979,
  "genres": [
    "Drama",
    "War"
  ]
}

PUT /movies/movie/5
{
  "title": "Kill Bill: Vol. 1",
  "director": "Quentin Tarantino",
  "year": 2003,
  "genres": [
    "Action",
    "Crime",
    "Thriller"
  ]
}

PUT /movies/movie/6
{
  "title": "The Assassination of Jesse James by the Coward Robert Ford",
  "director": "Andrew Dominik",
  "year": 2007,
  "genres": [
    "Biography",
    "Crime",
    "Drama"
  ]
}
```
##### _search端点: 
URL： <index>/<type>/<_search>  index & type为可选
- /_search - 搜索所有索引和所有类型。
- /movies/_search - 在电影索引中搜索所有类型
- /movies/movie/_search - 在电影索引中显式搜索电影类型的文档。
##### 搜索请求正文和ElasticSearch查询DSL:
如果只是发送一个请求到上面的URL，我们会得到所有的电影信息。为了创建更有用的搜索请求，还需要向请求正文中提供查询。 请求正文是一个JSON对象，除了其它属性以外，它还要包含一个名称为 “query” 的属性，这就可使用ElasticSearch的查询DSL。
``` 
{
    "query": {
        //Query DSL here
    }
}
```
##### 关于分词：
term ：直接查询精确的
match：会使用分词器解析！（先分析文档，然后在通过分析的文档进行查询！）

查询字符串查询:  查询 coppola
req:
``` 
GET /_search
{
  "query": {
    "query_string": {
      "query": "coppola"
    }
  }
}
```
resp: 
``` 
{
  "took" : 50,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,    # 返回个数
      "relation" : "eq"
    },
    "max_score" : 0.9218686,
    "hits" : [        # 查询到的数据
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "1",
        "_score" : 0.9218686,
        "_source" : {
          "title" : "The Godfather",
          "director" : "Francis Ford Coppola",
          "year" : 1972,
          "genres" : [
            "Crime",
            "Drama"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "4",
        "_score" : 0.9218686,
        "_source" : {
          "title" : "Apocalypse Now",
          "director" : "Francis Ford Coppola",
          "year" : 1979,
          "genres" : [
            "Drama",
            "War"
          ]
        }
      }
    ]
  }
}
```
- `select * from summer.user where name = 'summer'`
``` 
GET /summer/user/_search
{
    "query": {
        "match": {
            "name": "summer"
        }
     }
}
```
- `select name,age from summer.user where nickname = 'dollarkiller'`
``` 
GET /summner/user/_search
{
     "query": {
         "match": {
             "nickname": "dollarkiller"
         }
     }
    "_source": ["name","age"]
}
```
- 排序 `select * from summer.user where nickname = 'dollarkiller' ORDER BY age`
``` 
GET /summner/user/_search
{
    "query": {
         "match": {
             "nickname": "dollarkiller"
         }
     }
    "sort": [
        { 
            "age": { "order": "desc" }
         }
    ]
}
```
- 分页 `select * from summer.user where nickname = 'dollarkiller' limit 10 offset 0`
``` 
GET /summer/user/_search
{
    "query": {
         "match": {
             "nickname": "dollarkiller"
         }
     }
     "from": 0,
     "size": 10
}
```
- must 多条件and `select * from user wher name='summer' and nickname='dollarkiller'`
``` 
GET /summer/user/_search
{
    "query": {
         "bool": {
            "must": [
                 "match": {
                    "nickname": "dollarkiller"
                 },
                 "match": {
                        "name": "summer"
                  }
            ]
        }
     }
}
```
- should   多条件 or `select * from user wher name='summer' or nickname='dollarkiller'`
``` 
GET /summer/user/_search
{
    "query": {
         "bool": {
            "should": [
                "match": {
                    "nickname": "dollarkiller"
                },
                "match": {
                    "name": "summer"
                }
            ]
        }
     }
}
```
- must_not `select * from user wher not nickname='dollarkiller'`
``` 
GET /summer/user/_search
{
    "query": {
         "bool": {
            "must_not": [
                "match": {
                    "nickname": "dollarkiller"
                }
            ]
        }
     }
}
```
- 过滤器
``` 
GET /summer/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "name":"summer"
        }
      ]
      , "filter": {
        "range": {
          "age": {
            "gte": 10,
            "lte": 20
          }
        }
      }
  }
}
```
- 查询短句 要求 顺序且连续
match_phrase的分词结果必须在text字段分词中都包含，而且顺序必须相同，而且必须都是连续的。
``` 
GET index/type/_search
{
    "query": {
        "match_pharse": {
            "tag": "  is man "  # 需要是连续且顺序的 不然会查不到
        }
    }
}
```
- 查询短句 顺序 连续无要求
``` 
POST index/type/_search
{
    "query": {
        "query_string": {
            "query": "  goes to school",
            "fields": ["tag"]
        }
    }
}
```


### 自动检测及动态映射Dynamic Mapping
参考: https://elasticsearch.cn/question/4930
``` 
{
    "mappings":{
        "dynamic_templates":[
            {
                "ik_fields":{
                    "path_match":"ik.*",
                    "match_mapping_type":"string",
                    "mapping":{
                        "analyzer":"ik_max_word",
                        "search_analyzer":"ik_max_word",
                        "type":"text"
                    }
                }
            },
            {
                "whitespace_fields":{
                    "path_match":"ws.*",
                    "match_mapping_type":"string",
                    "mapping":{
                        "analyzer":"whitespace",
                        "type":"text"
                    }
                }
            },
            {
                "standard_fields":{
                    "path_match":"sd.*",
                    "match_mapping_type":"string",
                    "mapping":{
                        "analyzer":"standard",
                        "type":"text"
                    }
                }
            },
            {
                "keyword_fields":{
                    "path_match":"kw.*",
                    "match_mapping_type":"string",
                    "mapping":{
                        "analyzer":"standard",
                        "type":"keyword"
                    }
                }
            },
            {
                "not_indexed_fields":{
                    "path_match":"ni.*",
                    "match_mapping_type":"string",
                    "mapping":{
                        "enabled":false,
                        "type":"object"
                    }
                }
            }
        ],
        "properties":{
            "ik":{
                "properties":{
                    "names":{
                        "type":"text",
                        "analyzer":"ik_max_word"
                    },
                    "primary_name":{
                        "type":"text",
                        "analyzer":"ik_max_word"
                    }
                }
            },
            "kw":{
                "properties":{
                    "entity_type":{
                        "type":"long"
                    },
                    "id":{
                        "type":"keyword"
                    },
                    "suggest_types":{
                        "type":"long"
                    }
                }
            },
            "ni":{
                "properties":{
                    "logo":{
                        "type":"object",
                        "enabled":false
                    }
                }
            }
        }
    }
}
```

``` 
{
    "mappings" : {
      "properties" : {
        "ik" : {
          "properties" : {
            "description" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "names" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            }
          }
        },
        "kw" : {
          "properties" : {
            "entity_type" : {
              "type" : "long"
            },
            "id" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "suggest_types" : {
              "type" : "long"
            }
          }
        },
        "ranking_score" : {
          "type" : "float"
        }
      }
    }
}
```
