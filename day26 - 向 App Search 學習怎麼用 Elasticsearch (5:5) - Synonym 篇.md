# 喬叔教 Elastic - 26 - 向 App Search 學習怎麼用 Elasticsearch (5/5) - Engine 的 Search 進階剖析篇

**向 App Search 學習怎麼用 Elasticsearch** 系列文章索引

- [(1/5) - 揭開 App Search 的面紗](https://ithelp.ithome.com.tw/articles/10250168)

- [(2/5) - Engine 的 Index Settings 篇](https://ithelp.ithome.com.tw/articles/10250655)

- [(3/5) - Engine 的 Mapping 篇](https://ithelp.ithome.com.tw/articles/10251102)

- [(4/5) - Engine 的 Search 基礎剖析篇](https://ithelp.ithome.com.tw/articles/10251530)

- [(5/5) - Engine 的 Search 進階剖析篇](https://ithelp.ithome.com.tw/articles/10251945)

---

## 前言

前面的章節介紹了 App Search 在執行 Engine 的搜尋時，我們如何取得 App Search 發送給 Elasticsearch 的 Search Request，搭配這系列先前的文章所介紹各種客制的 Analysis 以及應用在 Mapping 上的各種配置方法，我們剖析了 App Search 在執行 Search 時是如何運作的，這篇文章將會進一步探討在使用 App Search 的其他功能、像是 `Synonyms`, `Curations`, `Relevance Tuning` 時，Search Request 會有什麼樣的變化。

### 進入此章節的先備知識

- Elasticsearch Query DSL 的基本知識。
- 請先閱讀本系列先前的文章。

### 此章節的重點學習

- 在使用 App Search 的 `Synonyms`, `Curations`, `Relevance Tuning` 功能時，底層是如何使用 Elasticsearch 的。

---

## 準備測試資料

首先我們先增加幾筆資料，以協助接下來幾個使用案例的說明，這三筆資料請直接使用 App Search Data Importer 匯入即可。

```
[
  {
    "id": "park_rocky-mountain",
    "title": "Rocky Mountain",
    "description": "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m).",
    "visitors": 4517585,
    "location": "40.4,-105.58",
    "date": "1915-01-26T06:00:00Z"
  },
  {
    "id": "yangming-mountain",
    "title": "Yangming Mountain",
    "description": "Yangmingshan National Park is one of the nine national parks in Taiwan, located between Taipei and New Taipei City. The districts that house parts of the park grounds include Taipei's Beitou and Shilin Districts; and New Taipei's Wanli, Jinshan and Sanzhi Districts. The national park is known for its cherry blossoms, hot springs, sulfur deposits, fumaroles, venomous snakes, and hiking trails, including Taiwan's tallest dormant volcano, Qixing (Seven Star) Mountain (1,120 m).",
    "visitors": 123123,
    "location": "25.17,121.56",
    "date": "1985-09-15T16:00:00Z"
  },
  {
    "id": "Himalaya-mountain",
    "title": "Himalaya",
    "description": "The Himalayas, is a mountain range in Asia separating the plains of the Indian subcontinent from the Tibetan Plateau. The range has many of Earth's highest peaks, including the highest, Mount Everest, at the border between Nepal and China. The Himalayas include over fifty mountains exceeding 7,200 m (23,600 ft) in elevation, including ten of the fourteen 8,000-metre peaks. By contrast, the highest peak outside Asia (Aconcagua, in the Andes) is 6,961 m (22,838 ft) tall.",
    "visitors": 52700000,
    "location": "27.59,86.55",
    "date": "1900-01-01T00:00:00Z"
  }
]
```



## Synonym 同義字的查詢

針對探討 Synonym 同義字的查詢的執行方式，我們先建立一組同義字，這邊使用一個例子，將 `rocky` 和 `yangming` 這兩個字設成同義字：

![synonym setting](https://i.imgur.com/8RUBjmp.png)



接著我們到 Query Tester 執行查詢，並使用 `rocky` 來當查詢的關鍵字：

![synonym query tester](https://i.imgur.com/wvWQMjj.png)



接下來我們來看看 slowlog 幫我們印出來 Elasticsearch 收到的 Search Request 的內容是什麼：

![synonym request slowlog](https://i.imgur.com/dRvJTLR.jpg)

以下是 Formatted Search Request Payload： 

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
                        "query": "rocky",
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
                            "query": "rocky",
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
                    },
                    {
                      "bool": {
                        "should": [
                          {
                            "multi_match": {
                              "query": "rocky yangming",
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
                              "zero_terms_query": "NONE",
                              "auto_generate_synonyms_phrase_query": true,
                              "fuzzy_transpositions": true,
                              "boost": 0.75
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
        "query": "rocky",
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

我們可以發現這個 Request 和先前介紹基本 App Search 執行 Search 時產生的 Request 有一個塊 **新增加** 的一組 `multi_match` 查詢，並且使用 `bool query - should` 和原本的查詢包在一起：

![synonym additioinal request](https://i.imgur.com/0er1XJ3.png)

我們發現這個 query 的關鍵字，直接包含了 `rocky` 和 `yangming` ，這代表了一件事：

App Search 在處理 Synonym 時，**是在 Application 端進行的處理，不是使用 Elasticsearch 內的同義字字典機制**，也就是當 App Search 的 Search API 收到 `rocky` 的關鍵字時，在 Application 端，先發現 `rocky` 是包含在 Synonym 的定義中，所以直接將 Synonym 的 `rocky` 這組同義字設定展開，也就是 `rocky yangming`，並且另外帶入在 Elasticsearch 的查詢中，也就因此產生上面的這個查詢語句，並且由於是同義字的查詢，所以這部份的 boost 值設定為較低的 `0.75`。

> 這種做法的好處是，因為 App Search 的 同義字 是在 App Search 的後台讓使用者靈活的自行設置，所以在 Application 端處理的彈性較高，不用另外維護 Elasticsearch 參照到的同義字字典，同時為了能彈性的調整，所以這邊 Synonym 的執行方式選擇是 `searching` 時機的同義字比對，也就是在搜尋時將關鍵字參考到同義字字典後展開，查詢所有同義字有定義的詞，以查詢出包含這些詞的文件，而不是在 `indexing` 時期先參考好同義字字典，並先將同義字的相關的字詞都包含在 index 中。



## Curation

在使用 Curation 時，又是如何運作的呢？

以下我們透過 `mountain` 這個關鍵字為例，原始的 `mountain` 查詢結果如下：

![curation original request](https://i.imgur.com/TSL2fqg.png)

我們在先 App Search 建立一組新的 Curation 設定，針對 `mountain` 這個關鍵字。

![curation - create](https://i.imgur.com/YIFRdMX.png)

並且將原本分數最低的 `yangming mountain` ，拉到 promoted documents 中。

![curation - manage curation](https://i.imgur.com/y6Sbwh5.png)

這時我們再重新搜尋 `mountain` 時，這個 `yangming mountain` 的 Score 變成了 `1`，並排序在最上面。

![curation - new search](https://i.imgur.com/zpUITH6.png)

我們再透過 slowlog 來查詢底下發生了什麼事，這時我們發現有 **2筆的 logs**。

![curation slowlog](https://i.imgur.com/1lLYpBm.png)

原來 App Search 在處理 Curation 時，會將查詢結果分成兩部份來執行

1. 取得 Curated 項目
2. 執行其他文件的查詢 (不能包含 curated item)

以下我們分別查看各別的查詢內容為何：

###  取得 Curated 項目

```
{
  "from": 0,
  "size": 1,
  "timeout": "2000ms",
  "query": {
    "bool": {
      "must": [
        {
          "match_all": {
            "boost": 1
          }
        }
      ],
      "filter": [
        {
          "bool": {
            "must": [
              {
                "terms": {
                  "external_id": [
                    "yangming-mountain"
                  ],
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
  "highlight": {
    "fragment_size": 300,
    "number_of_fragments": 1,
    "type": "plain",
    "highlight_query": {
      "multi_match": {
        "query": "Mountain",
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

仔細看這部份的差異，主要就是以下這個：

![curation must promoted item](https://i.imgur.com/RntQr3y.png)

直接明確的透過 `terms query` 將 `external_id` 是 promoted 的這筆資料查出。

> 這邊雖然只是簡單的把一筆資料查出來，卻還是透過 Query 的方式來執行，我猜測有另個主要的目的就是同樣要使用 Highlighting 的機制，並且簡單的將最後的查詢結果能 combine 在一起。

### 執行其他文件的查詢 (不能包含 curated item)

這部份執行的目的，就是查詢出除了 promoted 的資料之外的資料。

```
{
  "from": 0,
  "size": 9,
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
                        "query": "Mountain",
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
                            "query": "Mountain",
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
      "filter": [
        {
          "bool": {
            "must_not": [
              {
                "terms": {
                  "external_id": [
                    "yangming-mountain"
                  ],
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
        "query": "Mountain",
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

從這邊產生出來的 Search Request ，可以發現主要是多增加了下面的這個 `filter - must_not` 的查詢。

![curation must_not promoted item](https://i.imgur.com/9TKboTa.png)

也是明確的宣告不要包含這筆已經另外處理的 promoted item。

最後 App Search 會將這兩個各別查詢的結果合併在一起，這部份就是 App Search Curation 底下運作的方式。



## Relevance Tuning

而 Relevance Tuning 的調整，對於查詢方式的影響是什麼，我們這邊直接來進行的實驗，將 `title` 欄位的 boost 從 `1` 調高到 `3`。

![relevance tuning - setting](https://i.imgur.com/CuphhyI.png)

接下來我們同樣透過 Query Tester 來執行搜尋，可以看到針對 `mountain` 這個關鍵字的回傳結果依照調整 relevanc boost 之後有些不一樣了。

![relevance tuning - query tester](https://i.imgur.com/zZypFHN.png)

直接查看 slowlog 看看看這個查詢的 payload。

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
                        "query": "mountain",
                        "fields": [
                          "description$string^1.0",
                          "description$string.delimiter^0.4",
                          "description$string.joined^0.75",
                          "description$string.prefix^0.1",
                          "description$string.stem^0.95",
                          "external_id^1.0",
                          "title$string^3.0",
                          "title$string.delimiter^1.2",
                          "title$string.joined^2.25",
                          "title$string.prefix^0.3",
                          "title$string.stem^2.85"
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
                            "query": "mountain",
                            "fields": [
                              "description$string.intragram^0.1",
                              "external_id.intragram^0.1"
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
                    },
                    {
                      "constant_score": {
                        "filter": {
                          "multi_match": {
                            "query": "mountain",
                            "fields": [
                              "title$string.intragram^0.3"
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
                        "boost": 0.3
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
        "query": "mountain",
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

這邊可以看到，針對 `title` 的欄位權重變成 **3倍**，因此 `title` 相關的 boosting 的值，也都對應的變成原本的 **3倍**。

![relevance tuning - normal fields](https://i.imgur.com/b0vjAxS.png)

而 intragram 使用的 `constant_score` 也另外加了另一組 `boost: 0.3` 的查詢。

![relevance tuning - intragram field](https://i.imgur.com/gq1s2CZ.png)

所以在畫面上的 Relevance Tuning 調整，就是直接反應到 Search Request 組成時，每個 `fields` 的 boost 配置。

## 總結

從本篇文章的探索，可以發現 App Search 在實作 `Synonyms`, `Curations`, `Relevance Tuning` 的機制時，是如何使用 Elasticsearch，有些是使用 Elasticsearch 原本就提供的功能、有的是配合一些進階的使用方式，提高 Application 端的管理方便性，也有不一定都會使用到 Elasticsearch 的功能而直接在 Application 端處理掉，這種做法都是有為了達到的好處及對應的取捨，會是我們在使用 Elasticsearch 進行進階的產品搜尋功能開發時很好的參考。

