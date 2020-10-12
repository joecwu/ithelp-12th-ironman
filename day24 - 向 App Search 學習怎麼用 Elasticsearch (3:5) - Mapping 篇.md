# 喬叔教 Elastic - 24 - 向 App Search 學習怎麼用 Elasticsearch (3/5) - Engine 的 Mapping 篇

**向 App Search 學習怎麼用 Elasticsearch** 系列文章索引

- [(1/5) - 揭開 App Search 的面紗](https://ithelp.ithome.com.tw/articles/10250168)

- [(2/5) - Engine 的 Index Settings 篇](https://ithelp.ithome.com.tw/articles/10250655)

- [(3/5) - Engine 的 Mapping 篇](https://ithelp.ithome.com.tw/articles/10251102)

- [(4/5) - Engine 的 Search 基礎剖析篇](https://ithelp.ithome.com.tw/articles/10251530)

- [(5/5) - Engine 的 Search 進階剖析篇](https://ithelp.ithome.com.tw/articles/10251945)

---

## 前言

在前一篇介紹中，我們剖析了 Engine Index Settings 的各種設定，其中一大塊介紹到了 Analysis 的部份，包含了 Analyzer, Tokenizer, Token Filters 的設定，這篇文章會針對 Index Mapping 的設置來分析，了解 App Search 如用使用這些 Analysis 的客製化設定來定義他的 Mapping，以及 Mapping 中是否有其他的使用技巧。

### 進入此章節的先備知識

- Elasticsearch Mapping 的基本知識。
- 請先閱讀過前一篇文章。

### 此章節的重點學習

- App Search 如何定義 Engine 的 Mapping。
- 若是修改 App Search Engine 的 Schema 後，會發生什麼變化。

---

## 取得 App Search Engine 的 Mapping 設定

第一筆如同前面的介紹，要先取得 Engine 的 id：

```
GET .ent-search-actastic-engines_v9/_search
```

針對 `_source` > `name` 找到指定 Engine 名稱的 Document ，並取得他的 `id` 欄位。

接下來我們針對取得的 Engine Id - `5f7deeaeac761042130bf192` 透過 Get Mapping API 取得 mapping 的設定：

```
GET .ent-search-engine-5f7deeaeac761042130bf192/_mapping
```

以下是回傳結果：

```
{
  ".ent-search-engine-5f7deeaeac761042130bf192" : {
    "mappings" : {
      "dynamic" : "strict",
      "properties" : {
        "__st_expires_after" : {
          "type" : "date"
        },
        "__st_text_summary" : {
          "type" : "text",
          "analyzer" : "iq_phrase_shingle"
        },
        "date$string" : {
          "type" : "text",
          "fields" : {
            "delimiter" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_delimiter",
              "position_increment_gap" : 100
            },
            "enum" : {
              "type" : "keyword",
              "ignore_above" : 2048
            },
            "intragram" : {
              "type" : "text",
              "index_options" : "docs",
              "analyzer" : "iq_intragram"
            },
            "joined" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_bigram",
              "position_increment_gap" : 100
            },
            "prefix" : {
              "type" : "text",
              "index_options" : "docs",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "i_prefix",
              "search_analyzer" : "q_prefix"
            },
            "stem" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_stem",
              "position_increment_gap" : 100
            }
          },
          "index_options" : "offsets",
          "analyzer" : "iq_text_base",
          "position_increment_gap" : 100
        },
        "description$string" : {
          "type" : "text",
          "fields" : {
            "delimiter" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_delimiter",
              "position_increment_gap" : 100
            },
            "enum" : {
              "type" : "keyword",
              "ignore_above" : 2048
            },
            "intragram" : {
              "type" : "text",
              "index_options" : "docs",
              "analyzer" : "iq_intragram"
            },
            "joined" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_bigram",
              "position_increment_gap" : 100
            },
            "prefix" : {
              "type" : "text",
              "index_options" : "docs",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "i_prefix",
              "search_analyzer" : "q_prefix"
            },
            "stem" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_stem",
              "position_increment_gap" : 100
            }
          },
          "index_options" : "offsets",
          "analyzer" : "iq_text_base",
          "position_increment_gap" : 100
        },
        "engine_id" : {
          "type" : "keyword"
        },
        "external_id" : {
          "type" : "keyword"
        },
        "id" : {
          "type" : "keyword"
        },
        "location$string" : {
          "type" : "text",
          "fields" : {
            "delimiter" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_delimiter",
              "position_increment_gap" : 100
            },
            "enum" : {
              "type" : "keyword",
              "ignore_above" : 2048
            },
            "intragram" : {
              "type" : "text",
              "index_options" : "docs",
              "analyzer" : "iq_intragram"
            },
            "joined" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_bigram",
              "position_increment_gap" : 100
            },
            "prefix" : {
              "type" : "text",
              "index_options" : "docs",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "i_prefix",
              "search_analyzer" : "q_prefix"
            },
            "stem" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_stem",
              "position_increment_gap" : 100
            }
          },
          "index_options" : "offsets",
          "analyzer" : "iq_text_base",
          "position_increment_gap" : 100
        },
        "title$string" : {
          "type" : "text",
          "fields" : {
            "delimiter" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_delimiter",
              "position_increment_gap" : 100
            },
            "enum" : {
              "type" : "keyword",
              "ignore_above" : 2048
            },
            "intragram" : {
              "type" : "text",
              "index_options" : "docs",
              "analyzer" : "iq_intragram"
            },
            "joined" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_bigram",
              "position_increment_gap" : 100
            },
            "prefix" : {
              "type" : "text",
              "index_options" : "docs",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "i_prefix",
              "search_analyzer" : "q_prefix"
            },
            "stem" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_stem",
              "position_increment_gap" : 100
            }
          },
          "index_options" : "offsets",
          "analyzer" : "iq_text_base",
          "position_increment_gap" : 100
        },
        "visitors$string" : {
          "type" : "text",
          "fields" : {
            "delimiter" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_delimiter",
              "position_increment_gap" : 100
            },
            "enum" : {
              "type" : "keyword",
              "ignore_above" : 2048
            },
            "intragram" : {
              "type" : "text",
              "index_options" : "docs",
              "analyzer" : "iq_intragram"
            },
            "joined" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_bigram",
              "position_increment_gap" : 100
            },
            "prefix" : {
              "type" : "text",
              "index_options" : "docs",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "i_prefix",
              "search_analyzer" : "q_prefix"
            },
            "stem" : {
              "type" : "text",
              "index_options" : "offsets",
              "term_vector" : "with_positions_offsets",
              "analyzer" : "iq_text_stem",
              "position_increment_gap" : 100
            }
          },
          "index_options" : "offsets",
          "analyzer" : "iq_text_base",
          "position_increment_gap" : 100
        }
      }
    }
  }
}

```



## 剖析 Engine 的 Mapping

### 基本設定

我們先把一些自訂的欄位收起來，整體設定一覽如下：

![app search engine mapping - overview](https://i.imgur.com/m505WQU.png)

這裡有包含基本的 mapping 設定與一些 App Search 的欄位：

- `dynamic: strict`: 這指的是關掉 dynamic mapping，也就是如果嘗試 indexing 一個 mapping 不存在的欄位時，會回傳錯誤並且 indexing 失敗，這也是一般進入 Production 環境時、而且資料欄位的增長是能控制時，較嚴謹的使用方式。
- 一些 App Search 內部使用的欄位：`engine_id`, `external_id`, `id`,  `__st_text_summary` 和 `__st_expires_after`。
- 我們自己新增的欄位：這些欄位會以 `$` 後面帶上這欄位的型態，這邊的例子都是 `string` ，因為我們 import document 後並沒有特別改變 Schema 的設定，所以預設都是 `string`。

### String 類型的 Mapping 設定

接下來我們將自訂的某個 `string` 欄位的細部 Mapping 設定展開：

![app search engine mapping - string](https://i.imgur.com/y2Tim5a.png)

這裡才是這篇文章的重點，也就是 App Search 如何處理這些 **需要被搜尋** 的文字欄位：

- `index_options: offsets`: 這邊使用的 `offsets` 是 `index_options` 最詳細的設定。
- `analyzer`: 預設的 Analyzer 是 `iq_text_base` ，也就是 App Search Analysis 中，基本的文字處理 Analyzer。
- `position_increment_gap`: 這是處理 array 這種類型的多值的資料，拉開每個值之間的距離，提升這種多值的資料查詢的準確性。

接下來是 Fields 的設定，總共包含以下幾種，並且我們使用 `_analyze` API 來示範：

- `delimiter`: 使用的是 `iq_text_delimiter` Analyzer，也就是先前介紹過，使用 `whitespace` 最單純的空白切詞，會依非英數的符號再切詞、不會特別處理文法的部份、會做取字根的轉換、會去除掉 stop words，主要的目的是用來處理字串中有分隔符號的情境。

  ```
  POST .ent-search-engine-5f7deeaeac761042130bf192/_analyze
  {
    "field": "description$string.delimiter",
    "text": "red;yellow;紫;黑色"
  }
  ```

  拆出來的 tokens 如下：

  ```
  redyellow紫黑色, red, yellow, 紫, 黑色
  ```

- `enum`: 使用的是 `keyword` 的 type，也就是保留完整的字串內容成為一個 token。

  ```
  POST .ent-search-engine-5f7deeaeac761042130bf192/_analyze
  {
    "field": "description$string.enum",
    "text": "red;yellow;紫;黑色"
  }
  ```

  拆出來的 tokens 如下：

  ```
  red;yellow;紫;黑色
  ```

- `intragram`: 使用的是 `iq_intragram` Analyzer，也就是以 `ngram` 3~4 為單位來分詞，不會去處理 CJK 的字元，主要應該是做 partial match 使用。

  ```
  POST .ent-search-engine-5f7deeaeac761042130bf192/_analyze
  {
    "field": "description$string.intragram",
    "text": "red;yellow;紫;黑色"
  }
  ```

  拆出來的 tokens 如下：

  ```
  red, yel, yell, ell, ello, llo, llow, low
  ```

- `joined`: 使用的是 `iq_text_bigram`，這是將 icu 切出來一個個的 token，兩兩一組產生成新的 token，例如，依此讓搜尋時若有臨近的詞，在找尋文件時也相近時，能被找到，可以依此拉高這種相鄰詞的相關性分數。

  ```
  POST .ent-search-engine-5f7deeaeac761042130bf192/_analyze
  {
    "field": "description$string.joined",
    "text": "red;yellow;紫;黑色"
  }
  ```

  拆出來的 tokens 如下：

  ```
  redyellow, yellow紫, 紫黑色
  ```

- `prefix`: 使用的 `indexing` analyzer 是 `i_prefix` ，而 `search` 時的 analyzer 是 `q_prefix`，也就是 `indexing` 時要用 `edge_ngram` 拆分成各種大小一組一組的 tokens，但查詢時要用原來的字去比對，不要另外拆分。

  其中 `i_prefix` 的執行效果：

  ```
  POST .ent-search-engine-5f7deeaeac761042130bf192/_analyze
  {
    "analyzer": "i_prefix", 
    "text": "red;yellow;紫;黑色"
  }
  ```

  拆出來的 tokens 如下：

  ```
  r, re, red, y, ye, yel, yell, yello, yellow, 紫, 黑, 黑色
  ```

  若是使用 `q_prefix` 的執行效果：

  ```
  POST .ent-search-engine-5f7deeaeac761042130bf192/_analyze
  {
    "analyzer": "q_prefix", 
    "text": "red;yellow;紫;黑色"
  }
  ```

  拆出來的 tokens 如下：

  ```
  red, yellow, 紫, 黑色
  ```

- `stem`: 使用的是 `iq_text_stem` Analyzer，主要是有處理過子根的處理，讓 tokens 只保留字根。

  ```
  POST .ent-search-engine-5f7deeaeac761042130bf192/_analyze
  {
    "field": "description$string.stem",
    "text": "quickly;cats;紫;黑色"
  }
  ```

  拆出來的 tokens 如下：

  ```
  quick, cat, 紫, 黑色
  ```



## Engine Schema 改變時 Mapping 的變化

當我們進入 App Search 的 UI，透過 Schema 把原先預設的 `text` 欄位，改成我們實際資料的欄位：

![image-20201009003000700](https://i.imgur.com/sF7SWWm.png)

這時底下到底發生了什麼變化?

首先我們看一下 index 中的 document：

```
GET .ent-search-engine-5f7deeaeac761042130bf192/_search
```

```
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : ".ent-search-engine-5f7deeaeac761042130bf192",
        "_type" : "_doc",
        "_id" : "5f7deec0ac7610cca50bf195",
        "_score" : 1.0,
        "_source" : {
          "title$string" : "Rocky Mountain",
          "description$string" : "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m).",
          "visitors$float" : 4517585.0,
          "location$location" : "40.4,-105.58",
          "date$date" : "1915-01-26T06:00:00+00:00",
          "id" : "5f7deec0ac7610cca50bf195",
          "external_id" : "park_rocky-mountain",
          "engine_id" : "5f7deeaeac761042130bf192",
          "__st_text_summary" : "Rocky Mountain Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m). 4517585 40.4,-105.58 1915-01-26T06:00:00Z",
          "__st_expires_after" : null
        }
      }
    ]
  }
}
```

可以發現，這些文件被 reindex 處理過，欄位名稱都改變了，從原本都是 `$string` 結尾的欄位，各自對應有 `$float`, `$location`, `$date` 的型態描述。

再來我們看一下 mapping：

```
GET .ent-search-engine-5f7deeaeac761042130bf192/_mapping
```

![engine mapping after re-index](https://i.imgur.com/GDwdzR0.png)

我們可以發現，原先 `$string` 的這些所有欄位定義都還是存在，代表 index 沒有被重新建立，也因為 Elasticsearch Mapping 新增欄位後就不能刪掉這些定義，所以舊的定義都還是存在，但各自都擁有新的態型的欄位定義了，像是 `date$date`, `location$location`, `visitors$float`。



## 結語

從這篇文章的探討，我們可以知道 App Search Engine 的 Mapping 設定方式，特別是針對 `string` 型態的欄位定義了許多的 `fields` ，另外在修改 Engine Schema 時，會 re-index 文件但會保留原先型態欄位的定義，下一篇將會介紹這些資料被建立起來後，實際的查詢時會如何被使用。



## 參考資料

- [官方文件 - Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/mapping.html)

