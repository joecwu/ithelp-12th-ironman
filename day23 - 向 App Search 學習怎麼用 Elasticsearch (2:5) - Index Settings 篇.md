# 喬叔教 Elastic - 23 - 向 App Search 學習怎麼用 Elasticsearch (2/5) - Engine 的 Index Settings 篇

**向 App Search 學習怎麼用 Elasticsearch** 系列文章索引

- [(1/5) - 揭開 App Search 的面紗](https://ithelp.ithome.com.tw/articles/10250168)

- [(2/5) - Engine 的 Index Settings 篇](https://ithelp.ithome.com.tw/articles/10250655)

- [(3/5) - Engine 的 Mapping 篇](https://ithelp.ithome.com.tw/articles/10251102)

- [(4/5) - Engine 的 Search 基礎剖析篇](https://ithelp.ithome.com.tw/articles/10251530)

- [(5/5) - Engine 的 Search 進階剖析篇](https://ithelp.ithome.com.tw/articles/10251945)

---

## 前言

在前一篇介紹中，我們說明了 App Search 是如何使用 Elasticsearch 當成 NoSQL Database 來儲存 App Search 應用程式端的資料，以及我們建立了 Engine 之後，相關的 Index 又是如何被建立出來，這篇文章將會針對每個 App Search Engine 的 Index Settings 進行深入的介紹。

### 進入此章節的先備知識

- Elasticsearch Index Setting 的設定方式。
- Elasticsearch Analysis - Analyzer, Tokenizer, Token Filter 的基本知識。

### 此章節的重點學習

- 剖析 App Search Engine 的 Index Settings。
- 針對 App Search 使用的 Analysis - Analyzer, Tokenizer, Token Filter 進行剖析。

---

## 取得 App Search Engine 的 Index Settings

從上一篇文章的介紹，我們知道要取得 App Searc Engine 的方式如下：

從 `.ent-search-actastic-engines_v9` 找到 Engine 的 id。

```
GET .ent-search-actastic-engines_v9/_search
```

針對 `_source` > `name` 找到指定 Engine 名稱的 Document ，並取得他的 `id` 欄位。

```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
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
        "_index" : ".ent-search-actastic-engines_v9",
        "_type" : "_doc",
        "_id" : "5f7deeaeac761042130bf192",
        "_score" : 1.0,
        "_source" : {
          "id" : "5f7deeaeac761042130bf192",
          "created_at" : "2020-10-07T16:37:02Z",
          "updated_at" : "2020-10-07T16:37:24Z",
          "type_" : "Engine::IndexedEngine",
          "account_id" : null,
          "cluster_id" : "5f7de9caac7610ac216468a8",
          "key" : "ykZuaxzHjycZx6WnDdag",
          "loco_moco_account_id" : "5f7de9caac7610ac216468ab",
          "managing_application_id" : null,
          "slug" : "joe-test",
          "name" : "joe-test",
          "api_based" : true,
          "last_touched_at" : null,
          "stub" : false,
          "demo" : false,
          "sample" : false,
          "meta_data" : { },
          "source" : null,
          "queued_for_deletion" : null,
          "frito_pie_content_source_id" : null,
          "page_limit" : null,
          "document_count" : 1,
          "moving" : false,
          "deaggregation_requested" : false,
          "index_settings_override" : { },
          "index_create_settings_override" : { },
          "query_boosted_documents_enabled" : false,
          "query_boosted_queries_enabled_override" : false,
          "language" : "zh",
          "source_engine_ids" : [ ]
        }
      }
    ]
  }
}

```

以 `joe-test` 為例，Engine id 為 `5f7deeaeac761042130bf192` 。

接下來可以從這個 Engine id 組出這個 Engine 的 Index name： `.ent-search-engine-5f7deeaeac761042130bf192`

我們透過 GET Index Setting API 即可取得這個 Engine 的 Index Settings：

```
GET .ent-search-engine-5f7deeaeac761042130bf192/_settings
```

以下就是這個 Index Settings 的內容：

```
{
  ".ent-search-engine-5f7deeaeac761042130bf192" : {
    "settings" : {
      "index" : {
        "mapping" : {
          "total_fields" : {
            "limit" : "99999999"
          }
        },
        "indexing" : {
          "slowlog" : {
            "threshold" : {
              "index" : {
                "warn" : "10s",
                "trace" : "500ms",
                "debug" : "2s",
                "info" : "5s"
              }
            }
          }
        },
        "auto_expand_replicas" : "0-1",
        "provided_name" : ".ent-search-engine-5f7deeaeac761042130bf192",
        "creation_date" : "1602088641726",
        "analysis" : {
          "filter" : {
            "front_ngram" : {
              "type" : "edge_ngram",
              "min_gram" : "1",
              "max_gram" : "12"
            },
            "bigram_joiner" : {
              "max_shingle_size" : "2",
              "token_separator" : "",
              "output_unigrams" : "false",
              "type" : "shingle"
            },
            "bigram_max_size" : {
              "type" : "length",
              "max" : "16",
              "min" : "0"
            },
            "phrase_shingle" : {
              "max_shingle_size" : "3",
              "min_shingle_size" : "2",
              "output_unigrams" : "true",
              "type" : "shingle"
            },
            "zh-stop-words-filter" : {
              "type" : "stop",
              "stopwords" : "_english_"
            },
            "delimiter" : {
              "split_on_numerics" : "true",
              "generate_word_parts" : "true",
              "preserve_original" : "false",
              "catenate_words" : "true",
              "generate_number_parts" : "true",
              "catenate_all" : "true",
              "split_on_case_change" : "true",
              "type" : "word_delimiter_graph",
              "catenate_numbers" : "true",
              "stem_english_possessive" : "true"
            },
            "zh-stem-filter" : {
              "name" : "light_english",
              "type" : "stemmer"
            }
          },
          "analyzer" : {
            "i_prefix" : {
              "filter" : [
                "icu_folding",
                "front_ngram"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_intragram" : {
              "filter" : [
                "icu_folding"
              ],
              "tokenizer" : "intragram_tokenizer"
            },
            "iq_phrase_shingle" : {
              "filter" : [
                "icu_folding",
                "phrase_shingle"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_delimiter" : {
              "filter" : [
                "delimiter",
                "icu_folding",
                "zh-stop-words-filter",
                "zh-stem-filter",
                "cjk_bigram"
              ],
              "tokenizer" : "whitespace"
            },
            "q_prefix" : {
              "filter" : [
                "icu_folding"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_base" : {
              "filter" : [
                "icu_folding",
                "zh-stop-words-filter"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_stem" : {
              "filter" : [
                "icu_folding",
                "zh-stop-words-filter",
                "zh-stem-filter",
                "cjk_bigram"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_bigram" : {
              "filter" : [
                "icu_folding",
                "zh-stem-filter",
                "bigram_joiner",
                "bigram_max_size"
              ],
              "tokenizer" : "icu_tokenizer"
            }
          },
          "tokenizer" : {
            "intragram_tokenizer" : {
              "token_chars" : [
                "letter",
                "digit"
              ],
              "min_gram" : "3",
              "type" : "ngram",
              "max_gram" : "4"
            }
          }
        },
        "priority" : "150",
        "number_of_replicas" : "1",
        "uuid" : "ELd3uxHRRlmPbZW3hDEemg",
        "version" : {
          "created" : "7090299"
        },
        "routing" : {
          "allocation" : {
            "require" : {
              "data" : "hot"
            }
          }
        },
        "search" : {
          "slowlog" : {
            "level" : "debug",
            "threshold" : {
              "fetch" : {
                "warn" : "1000ms",
                "trace" : "200ms",
                "debug" : "300ms",
                "info" : "800ms"
              },
              "query" : {
                "warn" : "2000ms",
                "trace" : "500ms",
                "debug" : "1000ms",
                "info" : "1500ms"
              }
            }
          }
        },
        "number_of_shards" : "2",
        "similarity" : {
          "default" : {
            "type" : "BM25"
          }
        }
      }
    }
  }
}
```



## 剖析 Engine's Index Settings

針對整份 Index Settings 我們分成以下幾部份來看：

### Index 基本設定

我將 `slowlog`, `analysis` 這兩個部份另外切出去討論，剩下的基本設定如下圖：

![engine index settings](https://i.imgur.com/7Xfnv2a.png)

- `index.mapping.total_fields.limit: 99999999`: 這個是指定 index 最多的欄位數量，預設是 `1000`，這裡將這個值設提高到 `99999999`。
- `auto_expand_replicas: 0-1`: 這裡設定 replica 的數量依照 cluster 的 node 數量來決定，會配置 `0~1` 之間，也就是如果有超過 1 個node，就會配置 1 份 replica。
- `priority: 150`: 這個數字是決定 node 重啟動時，身上的 index recover 的優先順序，數字愈大會愈先被 recover。
- `routing.allocation.require.data: hot`: 這是因為使用的 Elastic Cloud Deployment 是 Hot-warm architecture，因此這邊會預設將 index 被安排在 `hot` node 身上。
- `number_of_shards: 2`: 在  App Search，其實不確定會進入的資料量有多少，但可以肯定的是並沒有使用 Rollover 的機制，因此這邊的設定不是預設的 `1`，而是較大的 `2`，相信這是 App Search 團隊評估一般使用 App Search 的使用情境後定出較合適的配置。
- `similarity.default.type: BM25`: 這是相關性計分 (score) 時使用的演算法，目的是計算搜尋的關鍵字與找尋的文件的相關性，會多參考到詞頻、文件的長度、所有文件的平均長度，不過 `BM25` 已是 Elasticsearch 的預設使用的 Similarity。

### Slow Log

這個的設定值的目的，是當 `indexing` 或是 `searching` 的處理，慢到某個程度的時候，要把這 request 給寫在 log 中，也就是協助我們分析 **跑太慢的 request** 到底是做了什麼事，App Search 分別依照這兩種類型有不同的設定：

#### Indexing

```
        "indexing" : {
          "slowlog" : {
            "threshold" : {
              "index" : {
                "warn" : "10s",
                "trace" : "500ms",
                "debug" : "2s",
                "info" : "5s"
              }
            }
          }
        },
```

這邊的定義是如果 `index` 的行為，分別依照不同的 Log severity 條件，達到不同的時間限制，就會記錄 Log，例如如果現在 Log 層級是開 `debug` 時，只要 `index` 的處理超過 2秒，就會產生 slowlog 的記錄。

#### Searching

```
        "search" : {
          "slowlog" : {
            "level" : "debug",
            "threshold" : {
              "fetch" : {
                "warn" : "1000ms",
                "trace" : "200ms",
                "debug" : "300ms",
                "info" : "800ms"
              },
              "query" : {
                "warn" : "2000ms",
                "trace" : "500ms",
                "debug" : "1000ms",
                "info" : "1500ms"
              }
            }
          }
        },
```

`searching` 的部份，有分成兩個階段 `query` and `fetch` ，針對這兩個階段有各自的設定值，任一階段超過某個時間，就會觸發 slowlog 的記錄。

### Analysis

這部份是 Index Analayzer, Tokenizer, Token Filters 的自定義設定，由於我這次選擇的語系是 `Chinese` 這當中也會有少部份是針對中文有特別定義的，設定分別如下：

#### Tokenizer

```
          "tokenizer" : {
            "intragram_tokenizer" : {
              "token_chars" : [
                "letter",
                "digit"
              ],
              "min_gram" : "3",
              "type" : "ngram",
              "max_gram" : "4"
            }
          }
```

這裡有定義了 `ngram` 的 tokenizer，將文字的內容依 3~4 個字元的大小，切成各自的 tokens，定義成 `instagram_tokenizer`。

#### Token Filters

```
          "filter" : {
            "front_ngram" : {
              "type" : "edge_ngram",
              "min_gram" : "1",
              "max_gram" : "12"
            },
            "bigram_joiner" : {
              "max_shingle_size" : "2",
              "token_separator" : "",
              "output_unigrams" : "false",
              "type" : "shingle"
            },
            "bigram_max_size" : {
              "type" : "length",
              "max" : "16",
              "min" : "0"
            },
            "phrase_shingle" : {
              "max_shingle_size" : "3",
              "min_shingle_size" : "2",
              "output_unigrams" : "true",
              "type" : "shingle"
            },
            "zh-stop-words-filter" : {
              "type" : "stop",
              "stopwords" : "_english_"
            },
            "delimiter" : {
              "split_on_numerics" : "true",
              "generate_word_parts" : "true",
              "preserve_original" : "false",
              "catenate_words" : "true",
              "generate_number_parts" : "true",
              "catenate_all" : "true",
              "split_on_case_change" : "true",
              "type" : "word_delimiter_graph",
              "catenate_numbers" : "true",
              "stem_english_possessive" : "true"
            },
            "zh-stem-filter" : {
              "name" : "light_english",
              "type" : "stemmer"
            }
```

App Search 定義了以下幾種 Token Filter：

- `front_ngram`: 使用 `edge_ngram` 切 1~12 的 gram，這應該是用來做 prefix matching。
- `bigram_joiner`: 這的作用是將 bigram 切出來的小單位的 token ，再兩兩一組的結合在一起成為新的 token。
- `bigram_max_size`: 針對 bigram token 的長度限定最長是 16。
- `phrase_shingle`: 這個定義了 `shingle` 的方式，將小單位的 token 每 2~3 為一組，組成 phrase 大小的 token。
- `zh-stop-words-filter`: 這雖然命名是 `zh` 的 stop word filter，但是可能是 ES 預設並沒有中文的 stop word，所以這邊還是以 `_english_` 為語系的設定。
- `delimiter`: 這是處理若某個 token 裡面有一些符號，會再依這些非字母或數字的符號將 token 切成斷成新的 tokens。
- `zh-stem-filter`: 這是將 token 轉成子根的 token filter，但在 `zh` 的語系中，一樣是使用 `light_english` 當成設定值。

### Analyzer

```
          "analyzer" : {
            "i_prefix" : {
              "filter" : [
                "icu_folding",
                "front_ngram"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_intragram" : {
              "filter" : [
                "icu_folding"
              ],
              "tokenizer" : "intragram_tokenizer"
            },
            "iq_phrase_shingle" : {
              "filter" : [
                "icu_folding",
                "phrase_shingle"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_delimiter" : {
              "filter" : [
                "delimiter",
                "icu_folding",
                "zh-stop-words-filter",
                "zh-stem-filter",
                "cjk_bigram"
              ],
              "tokenizer" : "whitespace"
            },
            "q_prefix" : {
              "filter" : [
                "icu_folding"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_base" : {
              "filter" : [
                "icu_folding",
                "zh-stop-words-filter"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_stem" : {
              "filter" : [
                "icu_folding",
                "zh-stop-words-filter",
                "zh-stem-filter",
                "cjk_bigram"
              ],
              "tokenizer" : "icu_tokenizer"
            },
            "iq_text_bigram" : {
              "filter" : [
                "icu_folding",
                "zh-stem-filter",
                "bigram_joiner",
                "bigram_max_size"
              ],
              "tokenizer" : "icu_tokenizer"
            }
          },
```

最後是 Analyzer 的部份，總共定義了以下幾種：

>  從命名規則來看， `i` 的前綴是給 `indexing` 用的，`q` 的前綴是給 `query` 使用。

- `i_prefix`: 使用了 `icu_tokenizer` 及前面介紹的 `front_ngram`，目的應該就是在 `indexing` 時，直接切出 prefix 比對用的 tokens。
- `iq_intragram`: 這裡使用了前面介紹的 `intragram_tokenizer` ，主要的目的應該是 partial match 使用。
- `iq_phrase_shingle`: 這裡使用 `icu_tokenizer` 但搭配前面介紹的 `phrase_shingle` ，目的應該是增強中文字若是有連接在一起的詞在搜尋時，這種 2~3 個中文字連接在一起的這種 **phrase**，若是有比對到，分數要能拉高。
- `iq_text_delimiter`: 這裡用的是 `whitespace` 最單純的空白切詞的 tokenizer，並搭配前面介紹的 `delimiter` token filter，會依非英數的符號再切詞、不會特別處理文法的部份、會做取字根的轉換、會去除掉 stop words，主要的目的是用來處理字串中有分隔符號的情境。
- `q_prefix`: 這是針對 query 時用的 prefix analyzer，也就是不用把 query 時的查詢字串特別的拆分，而是直接與 indexing 時被拆分的內容來比對即可。
- `iq_text_base`: 這是 text 類型基本使用的 analyzer，單純的使用 `icu_tokenizer`，並配合 `zh-stop-words-filter` 的 token filter，中規中舉。
- `iq_text_stem`: 這邊使用了 `icu_tokenizer` 並配合 `zh-stem-filter` 來將字根取出，主要的目的是讓字根相同的 token 也能互相被找出。
- `iq_text_bigram`: 這邊使用的是 `icu_tokenizer` 配合 `bigram-joiner` 和 `bigram_max_size` ，主要的目的是將相近的 token，兩兩一組的組合在一起，讓相鄰的查詢結果能提高查詢分數，這部份應該是針對 CJK (Chinese, Japenese, Korean) 語系的文字強化的處理。

## 結語

以上針對 App Search Engine 的 Index Settings 進行剖析，除了 Index Settings 上基本設定的參考、 Slowlog 的配置、這篇有一大塊的重點是介紹到 Analysis 的設定，而這些 Analysis 的實際用法，在下一篇的 Mapping 設定上，可以看到更明確的使用方式，這部份就待下一篇進行探討。

## 參考資料

- [官方文件 - Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/mapping.html)
- [官方文件 - Similarity](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/similarity.html)
- [官方文件 - Analysis](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)

