# 喬叔教 Elastic - 29 - Elasticsearch 的優化技巧 (3/4) - Index 的儲存空間最佳化

**Elasticsearch 的優化技巧** 系列文章索引

- [(1/4) - Indexing 索引效能優化](https://ithelp.ithome.com.tw/articles/10252325)

- [(2/4) - Searching 搜尋效能優化](https://ithelp.ithome.com.tw/articles/10252695)

- [(3/4) - Index 的儲存空間最佳化](https://ithelp.ithome.com.tw/articles/10253058)

- [(4/4) - Shard 的最佳化管理](https://ithelp.ithome.com.tw/articles/10253348)

---

## 前言

這系列的文章主要的目的在於當我們開始使用 Elastic Stack 時，我們如何優化 Elasticsearch 的使用方式，包含 Indexing, Searching, Disk Usage, Shard Optimization 等四個主題，這篇以是 Disk Usage 為主的介紹。

### 進入此章節的先備知識

- 已經有在使用 Elasticsearch，並且了解 Elasticsearch 的基本原理與操作方式。

### 此章節的重點學習

- Elasticsearch Cluster 的管理時，針對儲存空間進行優化的各種技巧。

---

## Index 的儲存空間最佳化

儲存空間的利率反應的就是 **錢**，有效的管理資料儲存的方式，有可能可以省下超過一半以上的儲存成本，以下會介紹各種 Elasticsearch Cluster 儲存空間最佳化的技巧：

- 優化 Mapping 的設定 - disable 不需要被搜尋的欄位
- 優化 Mapping 的設定 - 不需計算 score 的欄位可以關掉 `norms` 
- 優化 Mapping 的設定 - 調整 `index_options` 到需要的層級即可
- 優化 Mapping 的設定 - 避免使用預設的 dynamic string mapping
- 優化 Mapping 的設定 - 減少 `_source` 裡儲存的資料
- 優化 Mapping 的設定 - 設置 `best_compression` 來提升資料壓縮率
- 優化 Mapping 的設定 - 使用較省空間的資料型態
- 減少 Shard 的數量，增加 Shard 的大小
- 透過 Force Merge 來減少 Segment files 數量太多所佔用的空間
- 將相似的文件透過 index sorting 排在一起以提升壓縮率
- 讓 Document 的欄位順序保持一致以提升壓縮率
- 使用 Rollup 的機制，將歷史資料的儲存顆粒度提升
- 使用 Hot Warm Cold Architecture 來分配合適的儲存硬體

以下會分別針對這些優化項目進行說明。

### 優化 Mapping 的設定 - disable 不需要被搜尋的欄位

```
PUT index
{
  "mappings": {
    "properties": {
      "foo": {
        "type": "integer",
        "index": false
      }
    }
  }
}
```

不論是 `text`, `keyword`, `integer` 等各種型態的欄位，如果這個欄位已經明確是不需要使用 query 的，這時可以把 index 設成 `false` ，這樣 **會節省不少的空間**。

> `index: false` 的欄位，雖然不能用在 query ，但是還是可以使用 aggregation。

### 優化 Mapping 的設定 - 不需計算 score 的欄位可以關掉 `norms` 

```
PUT index
{
  "mappings": {
    "properties": {
      "foo": {
        "type": "text",
        "norms": false
      }
    }
  }
}
```

`norms` 是用來處理 query 時，儲存相關性計分所需要使用到的資訊，基本上會一個文件的一個 mapping 宣告的欄位就會佔用 1個 byte (就算某件文件沒有這個欄位還是會佔用)，所以欄位數量多、資料筆數多時，這個儲存空間會很大。( [官方文件 - norms](https://www.elastic.co/guide/en/elasticsearch/reference/current/norms.html) )

所以如果這個欄位 **需要被搜尋，但是不用考慮相關性計分** 時，請把 `norms` 宣告成 `false` ，以節省儲存空間。

> `norms` 的設定可以使用 PUT mapping API 動態的關閉，但是關閉後就像是 Delete 文件一樣，並不會馬上省節空間，會等到下一次 Segment Files merge 時，才會真正節省到磁碟空間。

### 優化 Mapping 的設定 - 調整 `index_options` 到需要的層級即可

`index_options` 是用來控制 inverted index 中要存放哪些資訊的設置，這些資訊會影響到 **相關性計分計** 在計算時參考的資訊，或是某些功能能不能使用，例如 highlighting, proximity or phrase query  。( [官方文件 - index_options](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-options.html) )

```
PUT index
{
  "mappings": {
    "properties": {
      "foo": {
        "type": "text",
        "index_options": "freqs"
      }
    }
  }
}
```

例如 `freqs` 記錄的是詞頻，是相關性計分在計算是會使用到的，而 `positions` 記錄的是 term 的位置，是 phrase query 需要用的，如果我們確認這個欄位不會需要使用 phrase query，我們可以將他設為 `freqs` ，以減少記錄 `positions` 所使用的空間。

### 優化 Mapping 的設定 - 避免使用預設的 dynamic string mapping

Elasticsearch 針對 `string` 型態的欄位，預設的 dynamic mapping 規則如下：

```
{
  "string_fields": {
    "mapping": {
      "norms": false,
      "type": "text",
      "fields": {
        "keyword": {
          "ignore_above": 256,
          "type": "keyword"
        }
      }
    },
    "match_mapping_type": "string",
    "match": "*"
  }
}
```

也就是只要被判定是 `string` 型態的欄位，除了主要的型態是 `text` 之外，還會建立一個 field 並且指定型態為 `keyword`。

```
  "my_string_field": {
    "type": "text",
    "fields": {
      "keyword": {
        "type": "keyword",
        "ignore_above": 256
      }
    }
  }
```

這樣會產生並儲存兩種不同的 analyzed 結果。

如果很明確知道使用的情境，應該明確的定義好資料的型態及處理的方式，又或著是大部份的欄位不需要進行 text search，只有特定的的欄位才需要，也可以設定 dynamic template，將預設的 `string` 型態先指定成 `keyword` 的資料型態。

```
PUT index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```



### 優化 Mapping 的設定 - 減少 `_source` 裡儲存的資料

`_source` 儲的是文件的 JSON 原始資料，如果不需要存取原始資料，例如使用 Elasticsearch 對 description 進行搜尋，但搜尋的結果不用回傳 description，只要回傳 document id 或是 titile 等其他欄位，就可以把這些不需要回傳的欄位，從 `_source` 中排除，甚至可以將整份文件的 `_source` 都關閉。

> 這邊要注意，若是沒有存 `_source` ，會無法再使用 **reindex** 或是 **_update** 的操作。



### 優化 Mapping 的設定 - 設置 `best_compression` 來提升資料壓縮率

`_source` 和 `stored fields` 的儲存資料，可以透過 `index.codec` 的設定來調整資料壓縮的方式，設定成 `best_compression` 可以提高壓縮率，當然成本就是像 `stored fiels` 的處理的效能就會慢一點。 ( [官方文件 - codec](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-codec) )



### 優化 Mapping 的設定 - 使用較省空間的資料型態

如果你的資料是整數，而且已經確定資料的範圍大小，應該明確的使用 `byte`, `short`, `integer`, `long`, `unsign_long` ，而不要直接用預設的 dynamic mapping 一律判斷成 `long` 。

同樣的如果是有小數的數值，也應該明確的指定 `half_float`, `float`, `double`, `scaled_float`，而不要直接用預設的 dynamic mapping 一律判斷成 `float` 。



### 減少 Shard 的數量，增加 Shard 的大小

較大的 Shard 對於資料的儲存與查詢效率愈高，要優化這部份的做法有以下幾種：

- 建立 Index 時，就指定數量較少的 primary shard。
- time-series data 在使用 Rollover API 時，建立的 index 數量也要注意不要太多，造成這份資料的總 shard 數量太多。
- 可以搭配 Shrink API 當 indexing 處理完成後，減少 primary shard 的數量。

> 雖然應考慮讓 Shard 數量變少、Size 變大，但 Shard 的數量規劃，還是要注意以下兩點：
>
> 1. 單一 shard 愈大，會讓 cluster rebalancing 時成本較高。
> 2. shard 數量太少，也會限制資料被分散處理的能力。



### 透過 Force Merge 來減少 Segment files 數量太多所佔用的空間

Segment files 的單檔愈大、總數愈少，會對於空間的使用率愈好，甚至像是已刪除的文件，會是透過 **標示為刪除** 的方式來處理，並且會等到 segment files merge 時才會真正的移除，因此當 index 被 rollover 之後，或是不再需要被寫入時，應該將 index 進行 segment files 的 merge，而且可以透過 [_forcemerge API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html)  指定 `max_num_segments=1` ，只保留一份 segment file 即可。



### 將相似的文件透過 index sorting 排在一起以提升壓縮率

Elasticsearch 的 `_source` 在保存時，會將一匹文件一起進行壓縮以提升壓縮率，一般使用 Elasticsearch 的情境，像是用來處理 Logs，有蠻多文件其實是有不少欄位是一樣的，因此透過適當的 `index sorting` 設置，讓資料在 Indexing 時就會依照這個規則來排序，除了能提升搜尋時的效率，也可以提高資料保存時的壓縮率進而節省儲存空間。( [官方文件 - Index Sorting](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-index-sorting.html) )



### 讓 Document 的欄位順序保持一致以提升壓縮率

這部份和上一段的原理是一樣的，Elasticsearch 的 `_source` 在保存時，會將一匹文件一起進行壓縮以提升壓縮率，而 JSON 文件的欄位若是會不同的話，就算裡面的值是一樣，但因為 indexing 時欄位的排序不同，會造成這些相似的字串反而是一段一段的散落在不同的位置，也會減少壓縮率，不如排序相同的文件，讓相似的字串有機會比較長的合在一起。

例如：

```
{"a":123,"b":"bbb","c":333,"d":456}
{"a":123,"b":"bbb","c":222,"d":456}
```

這樣的相似字串就會是 `{"a":123,"b":"bbb","c":` 和 `,"d":456}`

而若欄位沒有照一樣的順序：

```
{"a":123,"b":"bbb","c":333,"d":456}
{"b":"bbb","a":123,"c":222,"d":456}
```

這樣的相似字串就會是 `"a":123,`, `"b":"bbb",` `"c":` 和 `,"d":456}` ，而且位置也會比較零散，這樣的壓縮率就會較不好。



### 使用 Rollup 的機制，將歷史資料的儲存顆粒度提升

Rollup 的概念是將原本一筆一筆的資料，透過 aggreate 的方式，以某種顆粒度較大的方式，將資料彙總起來，例如把資料依每分鐘存一筆，並且只保留這分鐘的平均值、加總、最大值、最小值…等彙總資訊，不僅讓檢示這樣的資料的時速度變快，也能將太細顆粒的資料、或是原始資料，隨著時間從 Elasticsearch 中刪除，減少佔用的儲存空間。

詳細的介紹可參考先前的文章 [喬叔教 Elastic - 12 - 管理 Index 的 Best Practice (4/7) - Rollup](https://ithelp.ithome.com.tw/articles/10245259) 。



### 使用 Hot Warm Cold Architecture 來分配合適的儲存硬體

儲存空間的利用，除了節省空間之外，不同的資料配置不同等級的儲存硬體也是一項重要的資源利用管理方式。

透過不同等級的 storage，例如：

- **Hot Data Node** 使用 最高等級的 SSD。
- **Warm Data Node** 使用 次等級的 SSD。
- **Cold Data Node** 使用 HDD。

並且將資料妥善的依照 Hot, Warm ,Cold 的分類方式進行管理，以達到儲存硬體的最佳配置。

詳細的介紹可參考先前的文章 [喬叔教 Elastic - 10 - 管理 Index 的 Best Practices (2/7) - 三溫暖架構 - Hot Warm Cold Architecture](https://ithelp.ithome.com.tw/articles/10243650)



## 參考資料

- [官方文件 - Tune for disk usage](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-disk-usage.html)
- [官方文件 - index_options](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-options.html)
- [官方文件 - codec](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-codec)
- [官方文件 - forcemerge API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html)  
- [官方文件 - Index Sorting](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-index-sorting.html)

