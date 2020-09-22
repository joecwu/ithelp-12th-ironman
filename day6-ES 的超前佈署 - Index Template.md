# 喬叔教 Elastic - 06 - ES 的超前佈署 - Index Template

## 前言

將 Document indexing 進入 Elasticsearch 時，比較好的做法是先依照合適的 Index settings 及 mapping 來設置，但是若資料量不斷隨著時間增加，Index 也應該要隨著時間產生新的，如何有效的管理這些動態新增的 Index，我們需要的就是 Index Template。

### 進入此章節的先備知識

- Index, Mapping, Alias 的基本設定與操作。
- Dynamic Mapping 的基本認識。

### 此章節的重點學習

- Index Template 的基本使用方式。
- Elasticsearch 7.8 推出的 Component Template 怎麼使用。
- 剖析 Elastic Stack 中預設的幾個 Index Template。
- Index Template Simlate API 的使用方式。
- 建議的 Index Template 設計方式。

---

## Index Template

從 [官方文件 - Index Template](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html#) 可以知道 Index Template 的基本用法，主要的目的就是預先建立好 `Tempalte`，而當新的 Index 要被建立時，若符合設定好的 `index_patterns`，則使用這個 `Template`裡的設定來建立 Index。

建立一個 Index Template 時，可以設定以下資訊：

- `index_patterns`: 這是指 index 或 data stream 的名字，可以使用萬用字元 `*` 來定義這個 pattern。
- `data_stream`: 這是 **X-Pack** 中的功能 (Basic license就能使用)，主要是針對 time-series 的資料的一整套 aliase, rollover 的管理機制，一般會在 Index Lifecycle Management 中搭配設定及使用，之後會有文章來介紹這種資料的管理方式。[官方文件 - Data Streams][https://www.elastic.co/guide/en/elasticsearch/reference/7.9/data-streams.html]
- `template`: 可以包含 Aliases, Mappings, Index Settings 的設定。
- `composed_of`: 這是 Elasticsearch 7.8 新增的功能，可以套用事先定義好的 Component Template，這個可以設定多個 Component Templates，若有重覆的設定值，會以"後面的蓋掉前面的"來進行合併。
- `priority`: 也是 Elasticsearch 7.8 新增的功能，指定 Index Template 的優先順序，數字愈大愈優先。(若沒有指定，會當成`0`，也就是最低優先權來處理)
- `version`: 讓使用者自己編寫的版本號。
- `_meta`: 也是 Elasticsearch 7.8 新增的功能，讓使用者自己存放任意的物件資料。

> 建立 Index Template 時，會檢查是否有同樣的 `priority` 而 `index_patterns` 有重疊的情況，若有衝突會直接拋出錯誤。

以下就是個簡單的基本例子：

```
PUT _index_template/template_1
{
  "index_patterns" : ["te*"],
  "template": {
    "settings" : {
        "number_of_shards" : 1
    },
    "aliases" : {
        "alias1" : {},
        "{index}-alias" : {} 
    },
    "mappings" : {
      "_source" : { "enabled" : false }
    }
  },
  "composed_of": ["template_component_1", "template_component_2"],
  "priority" : 0,
  "version": 5,
  "_meta": {
    "description": "test by Joe",
    "latest_modify_date": "2020-09-20"
  }
}
```



## Component Template

從 [官方文件 - Put Component Template](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/indices-component-template.html) 可以查到基本的用法，主要是建立**可被 Index Template 重覆使用**的一組設定樣版，而可以設定的內容如下：

- `template`: 可以包含 Aliases, Mappings, Index Settings 的設定。
- `version`: 讓使用者自己編寫的版本號。
- `_meta`: 讓使用者自己存放任意的物件資料。

簡單的例子：

```
PUT _component_template/template_1
{
  "template": {
    "settings" : {
        "number_of_shards" : 1
    },
    "aliases" : {
        "alias1" : {},
        "{index}-alias" : {} 
    },
    "mappings" : {
      "_source" : { "enabled" : false }
    }
  },
  "version": 123
}
```



## 剖析 Elastic 官方的內建的 Template

Elasticsearch X-pack 內建有兩個預設的 Index Template, `logs` 和 `metrics`，以下針對 `logs` 來做剖析。

### 內建的 Index Template - `logs`

從 `_index_template` API 可以直接 get 到這個內建的 `logs` index template.

![index template - logs](https://i.imgur.com/l8u94Rm.png)

這是個 Index Template 是 Elastic Agent 產生的 logs 預設會套用的設置，其中幾個重點：

- 會套用在符合 `logs-*-*`格式的index上
- 有 `priority: 100`很高的優先權，所以要注意如果你自己的 index template 其中`index_patterns`有同樣的格式的話，小心會被這個預設的配置給蓋掉。
- 有使用到 `logs-settings`和 `logs-mappings` 的 Component Templates.

以下是 `logs-settings`的 Component Template.

```
{
  "component_templates" : [
    {
      "name" : "logs-settings",
      "component_template" : {
        "template" : {
          "settings" : {
            "index" : {
              "lifecycle" : {
                "name" : "logs"
              },
              "codec" : "best_compression",
              "query" : {
                "default_field" : [
                  "message"
                ]
              }
            }
          }
        },
        "version" : 0,
        "_meta" : {
          "description" : "default settings for the logs index template installed by x-pack",
          "managed" : true
        }
      }
    }
  ]
}

```

主要設置的是 index settings

- 定義了這一系列 index 一致的管理方式 - `lifecycle`。
- `codec` 指定為 `best_compression`。
- 指定 `query` 的 `default_field`。

另外是 `logs-mappings`的內容

```
{
  "component_templates" : [
    {
      "name" : "logs-mappings",
      "component_template" : {
        "template" : {
          "mappings" : {
            "dynamic_templates" : [
              {
                "strings_as_keyword" : {
                  "mapping" : {
                    "ignore_above" : 1024,
                    "type" : "keyword"
                  },
                  "match_mapping_type" : "string"
                }
              }
            ],
            "date_detection" : false,
            "properties" : {
              "@timestamp" : {
                "type" : "date"
              },
              "ecs" : {
                "properties" : {
                  "version" : {
                    "ignore_above" : 1024,
                    "type" : "keyword"
                  }
                }
              },
              "data_stream" : {
                "properties" : {
                  "namespace" : {
                    "type" : "constant_keyword"
                  },
                  "type" : {
                    "type" : "constant_keyword",
                    "value" : "logs"
                  },
                  "dataset" : {
                    "type" : "constant_keyword"
                  }
                }
              },
              "host" : {
                "properties" : {
                  "ip" : {
                    "type" : "ip"
                  }
                }
              },
              "message" : {
                "type" : "text"
              }
            }
          }
        },
        "version" : 0,
        "_meta" : {
          "description" : "default mappings for the logs index template installed by x-pack",
          "managed" : true
        }
      }
    }
  ]
}
```

設置上的主要重點：

- 有使用到我們前一天介紹的 `dynamic_templates`，其中因為這一個 Index Template 的對象是 **Logs**，當有大量的文字的欄位時，不希望使用預設的 `text` + sub-fields `keyword`的配置，所以直接指定預設 `string` 的欄位就用 `keyword`的型態來處理。

  ```
  {
    "strings_as_keyword" : {
      "mapping" : {
        "ignore_above" : 1024,
        "type" : "keyword"
      },
      "match_mapping_type" : "string"
    }
  }
  ```

-  `date_detection`有設為 `false`，在這邊採取比較嚴謹的方式，不特別去嘗試判斷字串的欄位是否為日期格式，若要使用日期格式的規則，可由另外的 Component Template 來指定。

- 有一些 Logs general的欄位，直接在 mappings 中先定義出來，例如： `@timestamp`、`ecs`、`host`、`message`...等。



### 相關的 Index Template

我們從 `_index_template` 可以看到 `logs-`開頭的 Index Template 還有蠻多個

![logs related index templates](https://i.imgur.com/FMx2tSW.png)

找其中一個來看，以 `logs-endpoint-events.process`為例：

![image-20200919044815844](https://i.imgur.com/dRxbNSY.png)

- 從 `index_pattern`可以發現，他也符合 `logs-*-*`的格式，但他更針對某一子集來設置 `logs-endpoint.events.process-*`

- 他的 priority 設置到更高的 `200`，代表只要符合這個子集範圍的 Index，就直接以這個設置優先套用。

- 一些基本的 `index settings` 與 `mappings` 都直接在 `template`中有定義。

- 他也有設置 Component Template - `logs-endpoint.events.process-mappings`

  ```
  {
    "component_templates" : [
      {
        "name" : "logs-endpoint.events.process-mappings",
        "component_template" : {
          "template" : {
            "mappings" : {
              "dynamic" : false,
              "properties" : {
                "@timestamp" : {
                  "type" : "date"
                }
              }
            }
          }
        }
      }
    ]
  }
  ```

  這個 Component 很單純的設定了，只要是這種類型的 Logs，就是要把 `dynamic`設成 `false`。

  

## 使用 Simulate API 來驗證 Index Template 的效果

當 Index Template 及 Component Template 設計的愈複雜，想要驗證結果如何，可以透過兩個 simulate API 來驗證。

### `Simulate Index`

用途：當有個指定名字的 Index 要產生時，最終他被套用的 Template 結果為何，以及是否有其他因同規則發生重疊的 Template。

![simulate index example](https://i.imgur.com/5vfk4aJ.png)

### `Simulate Template`

用途：當建立這個 Template 後，若有一個 Index 因為這個 Index Template 而建立起來時，裡面的設定會長什麼樣子，以及是否有與其他同規則發生重疊的 Template。

![simulate template](https://i.imgur.com/7WnAKjc.png)



## Index Template 使用的建議

- 可以針對 Index Template 的 `index_pattern` 及 `priority` 建立結構化的管理方式，例如 `logs`與 `logs-xxxxx-*`這樣的子、從關係。
- `version` 一定要給，早期沒有 `_meta` 時可以考慮以日期來當版本號，不過有 `_mate` 後，會建議將此 index template 的基本描述及最後更新時間記錄在 `_meta`中，以便於維護及管理。
- 可共用的設置抽出成 `Component Template`，可以當作某種角色的定義，讓管理更容易。
- 在設計好之後，請多使用 `Simulate API` 來驗證這個 Index Template 的產生結果及影響範圍。
- Elastic 官方還有其他的 Index Template 及 Component Template，在使用前可以多參考這些 Template 的設計方式。



## 參考資料

- [官方文件 - Index Template](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html#)
- [官方文件 - Put Index Template](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/indices-put-template.html)
- [官方文件 - Put Index Template (legacy)](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/indices-templates-v1.html)
- [官方文件 - Put Component Template](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/indices-component-template.html)

