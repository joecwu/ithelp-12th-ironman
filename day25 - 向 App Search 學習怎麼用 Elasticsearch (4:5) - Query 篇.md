# 喬叔教 Elastic - 25 - 向 App Search 學習怎麼用 Elasticsearch (4/5) - Engine 的 Search 基礎剖析篇

**向 App Search 學習怎麼用 Elasticsearch** 系列文章索引

- [(1/5) - 揭開 App Search 的面紗](https://ithelp.ithome.com.tw/articles/10250168)

- [(2/5) - Engine 的 Index Settings 篇](https://ithelp.ithome.com.tw/articles/10250655)

- [(3/5) - Engine 的 Mapping 篇](https://ithelp.ithome.com.tw/articles/10251102)

- [(4/5) - Engine 的 Search 基礎剖析篇](https://ithelp.ithome.com.tw/articles/10251530)

- [(5/5) - Engine 的 Search 進階剖析篇](https://ithelp.ithome.com.tw/articles/10251945)

---

## 前言

前面的章節介紹了 App Search 如何使用 Elasticsearch 來建立 Engine 的 Index 及 Mapping，也介紹各種客制的 Analysis 以及應用在 Mapping 上的各種配置方法，接下來我們要研究 App Search 在執行 Search 時是如何運作的。

### 進入此章節的先備知識

- Elasticsearch Query DSL 的基本知識。
- 請先閱讀本系列先前的文章。

### 此章節的重點學習

- 如何查看 App Search 的 Query 的方式。
- 剖析 App Search Query 的基本使用方式。

---

## 如何取得 App Search 的 Query 內容

App Search 是使用 Elasticsearch 當 Search Engine & Database、並且對外提供 API 的應用程式，他並沒有公開源始碼，所以對我們來說他是一個黑盒子，要知道 App Search 是如何對 Elasticsearch 執行 Query 的，最簡單的方式還是從 Elasticsearch 下手，那就是 - **透過 Elatsicsearch 的 Request Log 來查看 App Search 送了 Request**，但 Elasticsearc 並不會將一般的 Request Log 都記錄下來，所以我們這邊要使用一個小撇步，透過修改 **Slowlog** 的配置，讓我們想辦法得到我們要的東西。

```
PUT .ent-search-engine-5f7deeaeac761042130bf192/_settings
{
  "index.search.slowlog.threshold.query.info": "0ms"
}
```

設定完成之後，再透過 Get Index API 來確認已修改完成：

```
GET .ent-search-engine-5f7deeaeac761042130bf192/_settings
```

結果如下：

![app search sloglog setting](https://i.imgur.com/bylns9W.png)

這時我們透過 App Search 的 Query Tester 進行搜尋，以下以搜尋 `divide` 為例：

![app search query tester](https://i.imgur.com/buCVimg.png)

這時我們去查看 Elasticsearch 的 log，會發現搜尋 `divide` 的 slowlog 的被印出來：

![elastic cloud logs](https://i.imgur.com/vmkSxJS.png)

其中 `source[]` 裡面放的，就是 App Search 執行搜尋的動作時，產生並發送給 Elasticsearch 的 **Search Request**。

##  App Search 執行查詢時的 Search Request

我們將 Log 中的 **Search Request** 排版後展開如下：

```
{
  "from": 0,
  "size": 10,
  "timeout": "2000ms",
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "must": [
              {
                "bool": {
                  "should": [
                    {
                      "multi_match": {
                        "query": "divide",
                        "fields": [
                          "description$string^1.0",
                          "description$string.delimiter^0.4",
                          "description$string.joined^0.75",
                          "description$string.prefix^0.1",
                          "description$string.stem^0.95",
                          "external_id^1.0",
                          "title$string^1.0",
                          "title$string.delimiter^0.4",
                          "title$string.joined^0.75",
                          "title$string.prefix^0.1",
                          "title$string.stem^0.95"
                        ],
                        "type": "cross_fields",
                        "operator": "OR",
                        "slop": 0,
                        "prefix_length": 0,
                        "max_expansions": 50,
                        "minimum_should_match": "1<-1 3<49%",
                        "zero_terms_query": "NONE",
                        "auto_generate_synonyms_phrase_query": true,
                        "fuzzy_transpositions": true,
                        "boost": 1
                      }
                    },
                    {
                      "constant_score": {
                        "filter": {
                          "multi_match": {
                            "query": "divide",
                            "fields": [
                              "description$string.intragram^0.1",
                              "external_id.intragram^0.1",
                              "title$string.intragram^0.1"
                            ],
                            "type": "best_fields",
                            "operator": "OR",
                            "slop": 0,
                            "prefix_length": 0,
                            "max_expansions": 50,
                            "minimum_should_match": "35%",
                            "zero_terms_query": "NONE",
                            "auto_generate_synonyms_phrase_query": true,
                            "fuzzy_transpositions": true,
                            "boost": 1
                          }
                        },
                        "boost": 0.1
                      }
                    }
                  ],
                  "adjust_pure_negative": true,
                  "boost": 1
                }
              }
            ],
            "adjust_pure_negative": true,
            "boost": 1
          }
        }
      ],
      "adjust_pure_negative": true,
      "boost": 1
    }
  },
  "_source": {
    "includes": [
      "date$date",
      "visitors$float",
      "description$string",
      "external_id",
      "location$location",
      "title$string",
      "engine_id"
    ],
    "excludes": []
  },
  "sort": [
    {
      "_score": {
        "order": "desc"
      }
    },
    {
      "_doc": {
        "order": "desc"
      }
    }
  ],
  "highlight": {
    "fragment_size": 300,
    "number_of_fragments": 1,
    "type": "plain",
    "highlight_query": {
      "multi_match": {
        "query": "divide",
        "fields": [
          "description$string.prefix^1.0",
          "description$string.stem^1.0",
          "title$string.prefix^1.0",
          "title$string.stem^1.0"
        ],
        "type": "best_fields",
        "operator": "OR",
        "slop": 0,
        "prefix_length": 0,
        "max_expansions": 50,
        "zero_terms_query": "NONE",
        "auto_generate_synonyms_phrase_query": true,
        "fuzzy_transpositions": true,
        "boost": 1
      }
    },
    "order": "score",
    "require_field_match": false,
    "encoder": "html",
    "fields": {
      "description$string.stem": {
        "fragment_size": 100,
        "no_match_size": 100
      },
      "description$string.prefix": {
        "fragment_size": 100,
        "no_match_size": 100
      },
      "title$string.stem": {
        "fragment_size": 100,
        "no_match_size": 100
      },
      "title$string.prefix": {
        "fragment_size": 100,
        "no_match_size": 100
      }
    }
  }
}
```

讓我們來剖析這個 Search Request 的使用方式，主要以下面幾個方向來分析。

### 一般查詢設定

- `timeout: 2000ms`: 在執行 search request 時，有明確宣告 2秒 的timeout 限制，避免過長時間等待的查詢。
- `_source`: 這邊有明確的指定 `includes` 的欄位有哪些，也就是回傳結果只有包含這些有宣告的欄位，其他欄位不會回傳。
- `sort`: 排序的方式使用 `_score` 遞減，也就是相關性計分的結果愈相關愈優先，分數同樣時，則不特別指定其他排序的規則，並以 `_doc` 的宣告來排序。

> `_doc` 的用方可參考官方文件的說明：
>
> ![sort _doc](https://i.imgur.com/3N3AziY.png)

### 關鍵字查詢

關鍵字的搜尋，只會被套用到 `string` 型態的欄位，其他型態的欄位內容值，不會影響搜尋的結果，其中查詢時透過 `bool query` 的 `should` 也就是 **OR** 的邏輯，將 `multi_match` 與 `constant_score` 的查詢結果組合在一起：

#### `multi_match query`

![multi_match query](https://i.imgur.com/Dy4Ek66.png)

這邊主要使用 `multi_match query` ，將 `string` 欄位先前定義的各種 `fields` 的特性，透過 `cross_fields` 的方式，分別以不同的比重進行查詢：

- 預設的文字 Analyzer: **1.0**
- delimiter: **0.4**
- joined: **0.75**
- prefix: **0.1**
- stem: **0.95**

此外有透過 `minimum_should_match: 1<-1 3<49%`  的設定，來依照查詢時輸入的關鍵字切出的 terms，決定搜尋時的精確度，這邊的設置是如果是 1~3 個 term，會需要 1個 term 有 match，如果是超過 3 個 term，會要有 49% 的 match 率。(參考：[官方文件 - minimum_should_match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html))

> 其他有些上面有宣告的設定，但是與預設值相同的，這邊就不特別解釋，需要查詢定義的話，請參考：[官方文件 - Multi-match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)

#### `constant_score query`

這邊查詢主要的目的是將 `intragram` 的查詢結果使用 constant score 的計分方式包含至最終查詢的結果。

而 intragram 的 boost 優先權重是 `0.1` ，此外這邊並沒有跨多欄位合併查詢的需求，所以 multi_match 的 type 指定使用 `best_fields` 的方式執行。

![constant_score query](https://i.imgur.com/RFjIwWm.png)



### Highlight 查詢的結果

針對查詢的結果，一併會包含 `highlight` 的標示，這邊使用的是 `plain` 的 highlighting 方式，整體的設置如下：

![image-20201010001024291](https://i.imgur.com/2zbgEZ2.png)

也因為使用 `plain` highlighting 的方式，這邊的 `highlight_query` 只針對 `prefix` 與 `stem` 的 fields 進行查詢，因為在回傳時要顯示時，這兩種分詞的方式會是較能明確顯示關鍵字在原始字串中的比對結果。



## App Search 執行查詢時的 Search Response

上面的 Search Request 執行後，Elasticsearch 的回傳結果如下：

```
{
  "took" : 15,
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
    "max_score" : null,
    "hits" : [
      {
        "_index" : ".ent-search-engine-5f7deeaeac761042130bf192",
        "_type" : "_doc",
        "_id" : "5f7deec0ac7610cca50bf195",
        "_score" : 0.38768208,
        "_source" : {
          "title$string" : "Rocky Mountain",
          "engine_id" : "5f7deeaeac761042130bf192",
          "description$string" : "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m).",
          "date$date" : "1915-01-26T06:00:00+00:00",
          "visitors$float" : 4517585.0,
          "external_id" : "park_rocky-mountain",
          "location$location" : "40.4,-105.58"
        },
        "highlight" : {
          "description$string.prefix" : [
            "Bisected north to south by the Continental <em>Divide</em>, this portion of the Rockies has ecosystems"
          ],
          "title$string.prefix" : [
            "Rocky Mountain"
          ],
          "title$string.stem" : [
            "Rocky Mountain"
          ],
          "description$string.stem" : [
            "Bisected north to south by the Continental <em>Divide</em>, this portion of the Rockies has ecosystems"
          ]
        },
        "sort" : [
          0.38768208,
          0
        ]
      }
    ]
  }
}

```

Search 的結果就是包含我們 `_source` 、 `sort` 以及 `highlight` 宣告的內容。

## 結語

這篇的內容，串接了先前 App Search Engine 的 Index Analysis 設定、Mapping 的設定，了解 App Search 對 Elasticsearch 執行 Search Request 時的使用方式，如何在 indexing 時期先透過 `fields` 的各種 anlaysis 處理特性先將文件進行處理，搜尋時再依照各種權重的組合，將查詢的結果合併在一起，能增加比對的廣泛度卻不用依賴 `searching` 時期非常耗資源的 fuziness 或是 prefix query，同時又會依照各種特性的權重混合計算，能讓**最相關的排在查詢結果的最上方**，不過要注意的是這樣的組合在不少使用情境下、特別是中文的處理中，還是會有缺點，像是可能查詢結果下方會出現非預期的結果，不過這已經是讓 App Search 這種 Out of Box Experience 的產品 **以一招打天下** 的使用配置上很不錯的配置了，值得我們的參考。



## 參考資料

- [官方文件 - Sort search results](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html#sort-search-results)
- [官方文件 - Multi-match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)
- [官方文件 - minimum_should_match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html)
- [官方文件 - Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/highlighting.html)

