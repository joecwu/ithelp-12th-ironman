# 喬叔教 Elastic - 28 - Elasticsearch 的優化技巧 (2/4) - Searching 搜尋效能優化

**Elasticsearch 的優化技巧** 系列文章索引

- [(1/4) - Indexing 索引效能優化](https://ithelp.ithome.com.tw/articles/10252325)

- [(2/4) - Searching 搜尋效能優化](https://ithelp.ithome.com.tw/articles/10252695)

- [(3/4) - Index 的儲存空間最佳化](https://ithelp.ithome.com.tw/articles/10253058)

- [(4/4) - Shard 的最佳化管理](https://ithelp.ithome.com.tw/articles/10253348)

---

## 前言

這系列的文章主要的目的在於當我們開始使用 Elastic Stack 時，我們如何優化 Elasticsearch 的使用方式，包含 Indexing, Searching, Disk Usage, Shard Optimization 等四個主題，這篇以是 Searching 為主的介紹。

### 進入此章節的先備知識

- 已經有在使用 Elasticsearch，並且了解 Elasticsearch 的基本原理與操作方式。

### 此章節的重點學習

- Searching 的效能優化的各種技巧與建議。

---

## Searching 搜尋效能優化

這篇文章主要提供 Searching 的效能優化的各種技巧與建議：

- 與相關性計分無關的 Query，都使用 Filter 來處理
- 確保 Filesystem 有足夠的 memory cache
- 使用更快速的儲存硬體
- Document modeled
- 搜尋的欄位愈少愈好
- 依照 Aggregation 的需求 Pre-index 資料
- 盡量使用 `keyword` 來當作 identifiers 的型態
- Scripts 是昂貴的，應該盡量少用
- 使用日期時間當搜尋條件時，可以取整點，增加 Cache 利用率
- 將 filter 條件切割來提高 Cache 利用率
- 將不會再寫入的 Index 進行 Force-merge
- 將常會使用到 Terms Aggregations 的欄位，設定成 Eager Global Ordinals
- 預熱 filesystem cache
- 使用 index sorting 的設定，來加速 conjunctions 的搜尋
- 使用 `preference` 控制 `searching` request 的 routing 來增加 cache 使用率
- Replica 數量愈多不見得對搜尋愈有幫助
- 管理好使用 Elasticsearch 的方式，不要讓使用者擁有太大的彈性
- 使用 `Profile API` 來優化 Search Request
- 在 query 或 aggregation 處理需求量較高的環境中，安排特定的 Coordinating Node

以下會分別針對這些優化項目進行說明。

### 與相關性計分無關的 Query，都使用 Filter 來處理

因為 Filter 的處理不需要去計算 **相關性計分**，所以他的處理會比較快，也因此他的結果是適合被 cache 的，Elasticsearch 也就只會 cache filter 的結果，不會 cache 其他有相關性計分的 query，所以結論就是：預設請使用 filter，只有和相關性計分有關的查詢，才使用 query。

### 確保 Filesystem 有足夠的 memory cache

和 Indexing 時的建議一樣，Elasticsearch 使用時，由於使用 Lucene 進行許多 Segment files 的處理，會需要用到大量 file system 的 memory buffer，因此官方的配置建議上，會建議 JVM Heap size v.s OS filesystem 各配置 50% 的記憶體大小，因此請確保 Filesystem 擁有足夠的記憶體來替較常被使用的資料進行快取。

### 使用更快速的儲存硬體

Search 的處理有可能是 I/O bound 或是 CPU bound，如果你的 Search 是屬於 I/O bound，在官方的建議配置上，會建議基本上要使用 SSD 等級的硬碟來當成 Elasticsearch 的儲存硬體規格，而且使用 SSD 的配置，會讓整體的 C/P 值會較高。

若是你的 Search 是屬於 CPU bound，則應該將 node 配置較高等級的 CPU。

若是因資料量太大而有成本的考量，應該進一步再使用 Index Lifecycle Management 將 Indexing 完成的資料、或是較舊的資料，轉移到較便宜的磁碟硬體狀置上。

### Document modeled

儘量將你的 Document 在 Indexing 進入 Elasticsearch 時，就規劃成是 **針對 Searching 優化的結構**。

例如：避免使用 `join`、`nested` 的資料型態配合 `nested query` 會讓查詢速度慢好幾倍、`parent-child` 會讓查詢速度慢好幾百倍、`fuzzy`、`regex`…等查詢的效能也是非常的慢，所以若是能事先去正規劃、enrich raw log、透過 `ngram` 、 `bigram` 、 `shingle` …等各種 Analysis 套用在 multiple fields 中，能讓 `searching` 階段的處理盡量簡化，並且能達到同樣的效果，這樣搜尋速度會有非常明顯的改善。

### 搜尋的欄位愈少愈好

如果有使用 `query_string` 或 `multi_match` 這類查詢來同時查詢多個欄位時，優化的方式是使用 `copy_to` 在 `indexing` 時期就將這些會同時查詢的欄位合併到一個欄位中，並且 `searching` 時直接針對這個欄位進行搜尋，減少搜尋時的欄位數量，也會優化查詢的效率。

### 依照 Aggregation 的需求 Pre-index 資料

如果你的搜尋應用上常會針對一個欄位進行 `range` aggregation，而且都是一些固定的區間，例如：

```
PUT index/_doc/1
{
  "designation": "spoon",
  "price": 13
}

GET index/_search
{
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 10 },
          { "from": 10, "to": 100 },
          { "from": 100 }
        ]
      }
    }
  }
}
```

針對這種例子， **pre-index** 指的就可以將資料在 `indexing` 時，多增加一個欄位並使用 `keyword` 來存放這個分類的結果，如下：

```
PUT index
{
  "mappings": {
    "properties": {
      "price_range": {
        "type": "keyword"
      }
    }
  }
}

PUT index/_doc/1
{
  "designation": "spoon",
  "price": 13,
  "price_range": "10-100"
}
```

之後在使用時，就可以直接用這個欄位來進行 aggregation。

```
GET index/_search
{
  "aggs": {
    "price_ranges": {
      "terms": {
        "field": "price_range"
      }
    }
  }
}
```

這樣也能有效的提升 aggregation 的效率。

### 盡量使用 `keyword` 來當作 identifiers 的型態

不是所有的數值型態的資料都應該使用 `numeric` 的 data type。

Elasticsearch 針對 `numeric` 型態的欄位特別著重優化 `range` query 或 aggregation，而針對 `keyword` 欄位，會特別優化 `term` 或其他 `term-level` 相關的查詢。

因此如果你的 identifier 這類的資料是數值的型態，而且不需要進行 `range` query，那你應該考慮把他定義成 `keyword` 的型態。

> 如果你不確定會如何使用的話，就用 `multi-field` 把 `keyword` 和 `numeric` 都定義起來，也就是用空間換時間的方式，至少在 `searching` 階段能使用最合適的方式來進行最有效率的搜尋。

### Scripts 是昂貴的，應該盡量少用

不論是 scripts query 或是 scripted fields ，因為使用到 `script` 時，就沒辦法使用 Elasticsearch 的 index structure 或是相關的優化機制，所以如果 scripts 使用到的這些規則，若是能在 `indexing` 時期就先把資料預先算好並準備好，這樣也能有效的增加搜尋的效率。

### 使用日期時間當搜尋條件時，可以取整點，增加 Cache 利用率

這邊的原理，是因為 filter 的 cache 機制會依照 filter 的查詢條件來當成 cache key，一但 filter 的條件改變，這個 cache 自然就不會被 hit，以下面為例：

```
PUT index/_doc/1
{
  "my_date": "2016-05-11T16:30:55.328Z"
}

GET index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "my_date": {
            "gte": "now-1h",
            "lte": "now"
          }
        }
      }
    }
  }
}
```

如果我們現在的時間是 `16:31:29` ，我們進行了一次 search，過了一秒之後， `16:31:29` 這時再進行一次 search，如果第一次有產生 cache 的話，其實第二次的 filter 是無法利用到第一次的 cache 的，因為時間已經不同了，反之若是使用 rounded date，也就是直接 `/m` 取到分鐘為顆粒度的整數。

```
GET index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "my_date": {
            "gte": "now-1h/m",
            "lte": "now/m"
          }
        }
      }
    }
  }
}
```

以同樣上述的時間，產生出來的結果就會都是 `16:31:00` ，這樣就能提上 cache hit rate，而這個顆粒度也就取決於應用端可以接受的情境。



### 將 filter 條件切割來提高 Cache 利用率

如果我們的使用情境上有許多顆粒度很細、也就是 Cache hit rate 很低的查詢，例如我們總是要查詢 **最近一小時的資料** ，而且又想要愈即時、也就是顆粒度要很細的 filter，這個一般的查詢方式可能如下：

```
GET index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "my_date": {
            "gte": "now-1h",
            "lte": "now/"
          }
        }
      }
    }
  }
}
```

可以想像這個 cache hit rate 應該會極低，所以我們可以把這段時間切成三塊，讓其中一大塊的 cache rate 提高，並讓沒辦法 cache 的部份切成小塊而捨棄他的 cache 使用率：

```
GET index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "should": [
            {
              "range": {
                "my_date": {
                  "gte": "now-1h",
                  "lte": "now-1h/m"
                }
              }
            },
            {
              "range": {
                "my_date": {
                  "gt": "now-1h/m",
                  "lt": "now/m"
                }
              }
            },
            {
              "range": {
                "my_date": {
                  "gte": "now/m",
                  "lte": "now"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

這樣三塊分別是：

- 一個很精確的開始時間 ~ 開始時間之後的分鐘整點時間： `now-1h` ~ `now-1h/m`
- 開始時間之後的分鐘整點時間 ~ 最接近目前時間的分鐘整點時間：`now-1h/m` ~ `now/m`
- 最接近目前時間的分鐘整點時間 ~ 目前的時間：`now/m` ~ `now`

這種方式有利有弊，好處是讓中間那段有用分鐘整點來切齊的 cache hit rate 提高，但缺點是將 filter 切成三份，還是會增加一些 overhead，這部份就要依實際的使用情況來評估與調整了。

### 將不會再寫入的 Index 進行 Force-merge

如果是不會再寫入資料的 Index，例如 time-based indices 如果是已經 rotated，那麼不會再被寫入的 Index 應該要進行 segment files 的 force-merge，並且強制 merge 成只剩下一個 segment file，這樣也會提升搜尋的效率。

> 一般情況請不要針對一個還會持續寫入的 index 進行 force-merge ，這樣有可能會讓 performance 更差。

### 將常會使用到 Terms Aggregations 的欄位，設定成 Eager Global Ordinals

Global ordinals 是執行 terms aggregation 時會使用到的資料結果，預設是 lazy loading，因為 Elasticsearch 不知道你會針對哪些 keyword 欄位執行 terms aggregation。因此如果你有某個欄位明確的會頻繁執行 terms aggregation，可以進行以下的設定，將 `eager_global_oridnals` 設成 `true` ：

```
PUT index
{
  "mappings": {
    "properties": {
      "foo": {
        "type": "keyword",
        "eager_global_ordinals": true
      }
    }
  }
}
```

### 預熱 filesystem cache

可以使用 [`index.store.preload`](https://www.elastic.co/guide/en/elasticsearch/reference/current/preload-data-to-file-system-cache.html) 來告訴 OS 在 shard 起動時，要先將哪些 index 預先載入進 memory cache 中，另外載入的設定有以下幾種：

- `nvd` ：norms
- `dvd` ：doc values
- `tim` ：terms dictionaries
- `doc` ：postings lists
- `dim` ：points

例如：

```
PUT /my-index-000001
{
  "settings": {
    "index.store.preload": ["nvd", "dvd"]
  }
}
```

> **注意：** 若是預先載入太多的 indices 而導致 filesystem cache 不夠大來處理這些 indices 的話，`searching` 的效率是會下降的。

### 使用 index sorting 的設定，來加速 conjunctions 的搜尋

conjunctions 搜尋指的像是： `a AND b AND c ...` 這樣的搜尋，由於 conjunctions query 時的處理方式，是會將這些查詢條件，一個個去比對哪些文件"不符合條件"，若是一遇到不符合，就會跳過這個條件的文件，進行下一個條件的找尋。

宣告 Index sorting 的目的，主要是可以讓 符合 與 不符合 的文件先排在一起，不論是 `asc` 或 `desc` 都沒關係，只要讓他們先排在一起，一但使用 conjunctions 搜尋時，有某一個文件的條件不符合時，就能以較快的速度直接跳過這些不符合的文件，進入下一個條件的比較。

> 這邊要注意，這種小技巧只有對於資料內容差異較小的會較有效，也就是重覆的資料愈多愈有效。

### 使用 `preference` 控制 `searching` request 的 routing 來增加 cache 使用率

我們有 filesystem cache, request cache, query cache 等這些 cache 能優化 searching 的效能，不過這些都是 Node level 的 cache，如果我們有多份的 replica，同樣的 search request 若是導到不台，自然就沒辦法利用到另一台的 cache。

所以這邊的優化方式，是依照使用的情境，例如同一個使用者搜尋資料時的查詢條件應該會比較接近、或是相同地區的查詢條件會比較接近…等，我們就可以使用 `preference` 設定為 user id, session id, 甚至是 region id，來讓同樣的使用者或地區，能導到相同的 node，以增加 cache hit rate。

### Replica 數量愈多不見得對搜尋愈有幫助

replica 的數量還是要參考 primary shard 與 node 的數量來一併考量，如果 node 數量 4 個，primary shard 數量也是 4 個，並且 replica 是 0，這時 1 個 node 放 1 個 shard 的資料，這時 filesystem cache 的機制是最好的，如果 replica 設成 1，每一份 shard 都會有一份額外的 replica ，但這時 replica shard 也會佔用到 filesystem cache。

不過 replica 的數量另外一個最重要的目的是 availability，所以這會需要一併考慮，官方有個簡單的公式可做參考：

```
max(max_failures, ceil(num_nodes / num_primaries) - 1)
```

- `max_failures`: 代表 availability，也就是允許一口氣有幾個 node 壞掉時資料還是需要保留完整性。
- `num_nodes`: cluster node 數量。
- `num_primaries`: cluster 中，所有 primary shard 的數量。

### 管理好使用 Elasticsearch 的方式，不要讓使用者擁有太大的彈性

先前上課時最喜歡舉一個例子，如果大家使用過 Kibana ，可能會有過類似的經驗，在看 dashboard 時，調整時間時一不小心時間拉太長，拉到近1年，整個查詢就要等很久很久，最慘的是有可能把 cluster 的資源耗盡，又或著是我們提供給使用者的是 search box 讓使用者自己輸入 query_string 的字串，結果使用者輸了個超級複雜的查詢條件…

以上的例子都是我們開放讓使用者產生一些我們無法事先管理的 `searching request`，而這些 requests 是非常耗資源的，而甚至會影響到其他正常使用的狀況，這部份就會是應該要有良好的控管，確認我們適合提供的查詢方式，例如：

- 使用 alias + filter，限制只能查詢最近一段時間的資料，這可搭配 RBAC 來綁定在使用者的帳號上。
- 使用較有侷限的 UI 設計，讓使用者能產生的 `searching request` 都是在我們的掌握之中。

### 使用 `Profile API` 來優化 Search Request

可以使用 [Profile API](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/search-profile.html) 或是 Kibana Dev tools 的 **Search Profiler**，針對 search 底下運作的方式進行分析，也就可以針對某一個執行時間較長的查詢進行剖析或是調整。

![Query Profiler Visualization](https://i.imgur.com/7WRsPCQ.png)



### 在 query 或 aggregation 處理需求量較高的環境中，安排特定的 Coordinating Node

Corrdinator 在處理 aggregation 或是包含較複雜 sorting 處理的 query 時，會需要使用到較大量的 memory，因此將 Node 的身份進行有效的管理與分工，讓處理大量搜尋請求的任務，由專門的 Coordinating Node，有獨立的 memory 與系統資源，讓 Data node 在執行 searching 時，減少系統資源被其他處理佔用的情況。

> 也可以考慮將 Ingest Node 等專門的任務也都與 data node 中獨立出來，以確保處理 search request 的 node 的系統資源不會被其他處理佔用。



## 參考資料

- [官方文件 - Turn for search speed](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html)
- [Scaling Elasticsearch Part 2: How to Speed Up Search](https://dev.to/molly_struve/scaling-elasticsearch-part-2-how-to-speed-up-search-53of)