---
title: 如何快速测试 Elasticsearch 的 Analyzer
s: test-elasticsearch-analyzer-efficient
date: 2019-05-16 14:32:36
tags:
  - Elasticsearch
  - Analyzer
---

使用 Elasticsearch 搭建搜索引擎的过程中，免不了会反复测试 Analyzer 的效果，如果每次都建立一个索引，配置索引的 Analyzer，插入 Document，无疑效率会很低。ES 提供了 `_analyze` 接口，可以[无需创建索引快速测试 Analyzer](https://avnpc.com/pages/test-elasticsearch-analyzer-efficient)，推荐搭配 Kibana 中的 Dev Tools 一同使用。

## 测试 Tokenizer

``` json
POST _analyze
{
  "tokenizer": "icu_tokenizer",
  "text":      "你好世界"
}
```

Response

``` json
{
  "tokens" : [
    {
      "token" : "你好",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "世界",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    }
  ]
}
```

## 测试 Character Filter

由于 Analyzer 必须指定一个 Tokenizer，因此可以使用[Keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-tokenizer.html)这个特殊的 Tokenizer, 即不做任何分词，从而可以看到 Character Filter 的效果。

``` json
POST _analyze
{
  "char_filter": [ "html_strip" ],
  "tokenizer": "keyword",
  "text":      "<p>Hello <b>World</b>!</p>"
}
```

Response

``` json
{
  "tokens" : [
    {
      "token" : """

Hello World!

""",
      "start_offset" : 0,
      "end_offset" : 26,
      "type" : "word",
      "position" : 0
    }
  ]
}
```

## 测试 Token filter

``` json
POST _analyze
{
  "tokenizer": "icu_tokenizer",
  "filter": [{
    "type": "stop", "stopwords": ["am"]
  }],
  "text": "I am ironman"
}
```

Response

``` json
{
  "tokens" : [
    {
      "token" : "I",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "ironman",
      "start_offset" : 5,
      "end_offset" : 12,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
```

## 测试已经创建的索引

而对于已经创建的索引，可以通过 `${index}/_analyze` 接口来调用某个已经创建好的 Analyzer，或者预览某个 Field 对于文本的分析结果。 如创建如下索引

``` json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "icu_tokenizer",
          "filter": [
            "my_stop_filter"
          ]
        }
      },
      "filter": {
        "my_stop_filter": {
          "type": "stop",
          "stopwords": [
            "am"
          ]
        }
      }
    }
  },
  "mappings": {
    "my_type": {
      "properties": {
        "title": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
}
```

调用这个索引中已经创建的 Analyzer `my_analyzer`

``` json
POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "<p>I am <b>Ironman</b>!</p>"
}
```

或者预览 `title` 字段的分析结果

``` json
POST my_index/_analyze
{
  "field": "title",
  "text": "<p>I am <b>Ironman</b>!</p>"
}
```

而对于已经索引的数据，可以通过

```plain
GET /${index}/${type}/${id}/_termvectors?fields=${fields_name}
```

来查看实际存储的数据， 如

``` json
POST _bulk
{ "index": { "_index": "my_index", "_type": "my_type", "_id": 1} }
{ "title": "<p>I am <b>Ironman</b>!</p>" }

GET my_index/my_type/1/_termvectors?fields=title
{
  "_index": "my_index",
  "_type": "my_type",
  "_id": "1",
  "_version": 1,
  "found": true,
  "took": 2,
  "term_vectors": {
    "title": {
      "field_statistics": {
        "sum_doc_freq": 2,
        "doc_count": 1,
        "sum_ttf": 2
      },
      "terms": {
        "I": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 0,
              "start_offset": 3,
              "end_offset": 4
            }
          ]
        },
        "Ironman": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 2,
              "start_offset": 11,
              "end_offset": 22
            }
          ]
        }
      }
    }
  }
}
```
