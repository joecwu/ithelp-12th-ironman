# 喬叔教 Elastic - 10 - 管理 Index 的 Best Practices (2/7) - 三溫暖架構 - Hot Warm Cold Architecture

**管理 Index 的 Best Practices** 系列文章索引

- [(1/7) - Shard 的數量與 Rollover & Shrink API](https://ithelp.ithome.com.tw/articles/10243037)

- [(2/7) - 三溫暖架構 - Hot Warm Cold Architecture](https://ithelp.ithome.com.tw/articles/10243650)

- [(3/7) - Index Lifecycle Management (ILM)](https://ithelp.ithome.com.tw/articles/10244575)

- [(4/7) - Rollup](https://ithelp.ithome.com.tw/articles/10245259)

- [(5/7) - Transform](https://ithelp.ithome.com.tw/articles/10245472)

- [(6/7) - Snapshot Lifecycle Management (SLM)](https://ithelp.ithome.com.tw/articles/10246076)

- [(7/7) - 總結](https://ithelp.ithome.com.tw/articles/10246673)

---

## 前言

前一天介紹了 Index 中 Shard 數量對效能的影響以及透過 Rollover 及 Shrink API 來管理 time-based 資料的方法，接下來要介紹 Elastic Stack 6.3 時陸續開始所推出針對這些隨時間增長的資料更全面的整體管理解決方案。

### 進入此章節的先備知識

- Elasticsearch Index, Shard, Segment File 的基本認識。
- Index Template。
- Rollover 與 Shrink 的機制。

### 此章節的重點學習

- Elasticsearch 中的 hot-warm-cold architecture。
- Elasticsearch Freeze API。

---

## 冷熱資料管理的基本觀念

針對 time-based 的資料，Elasticsearch 在 5.x 版的時候，就提出了 Hot-Warm 的架構，主要是針對 **常用的資料** 與 **時間較久也就是不常用的資料** 分開用不同的硬體來存放，以達到資源有效的利用，不過從 Elastic Stack 6.3 開始，陸續在接連的幾個版本中，針對 time-based 資料的管理機制，提供了更全面的解決方案，就像是以下的拼圖，拼湊得更加完整了，從這篇開始，我們會陸續的做相關的介紹。

![hot-warm-features-frozen-indices-transp.png](https://i.imgur.com/96yYghR.png)

## Elastic Cloud Deployment 的入口

當你要建立 Hot-Warm Architecture 時，若是使用 Elastic Cloud，在 Deployment 的階段就有這個配置可以選擇，但是明眼的一看，他只有 **Hot-Warm** 並沒有 **Cold**，這是因為實際上機器的配置，並沒有特別針對 Cold 的配置來安排機器，至少這是目前在 Elastic Cloud 上還沒看到的。

![Elastic Cloud Hot-warm architecture](https://i.imgur.com/nFqQfBL.png)

若是進入到 **Customize deployment** 後，可以看到實際的配置，是只有 **Hot** 與 **Warm** ，沒有 Cold 的配置。

![elastic cloud hot-warm architecture 2](https://i.imgur.com/gDOSrPX.png)



## Hot-Warm-Cold Architecture

這邊提到了 Hot Warm Cold 三種狀態，可想而知，就是除了分出 Hot Warm 之外，還要更細分出 Cold 的存放方式，所以使用上的資料管理順序，也就如下圖所示，新進來的資料就是會配置到 Hot Data Node，再來一段時間後，進入 Warm Data Node，再過一段時間後，最後進入 Code Data Node。

![hot-warm-cold architecture](https://i.imgur.com/Le5JDWC.jpg)

而這三種的定義與差異如下：

- **Hot:** 因為負責最新的資料，所以會負責處理 indexing 的資料，同時新的資料被使用的機率也最高，所以也會處理頻繁的 searching 請求。
- **Warm:** 當一份 Index 的資料成長到一定的量、或是已經過了一段時間，會將這份資料轉到 Warm data node，這時 Warn data node 是 `read-only` 也就是不會需要處理 indexing 的請求，只會專注在處理 searching 的請求。
- **Cold:** 當一份 Index 的資料經過一段較長的時間，判定會較少使用到時，會將他移到 Cold data node，並且針對這些資料進行 `Freeze` 的處理 (下面會介紹)，這時資料會被以最節省系統資源的狀態下被保存，還是可以提供查詢，但速度會較慢。

### Hot Warm Cold 的資源使用狀況

![hot-warm-cold architecture heap usage](https://i.imgur.com/dZueTUH.png)

從上圖可以看到，在這種架構下，各自針對 JVM heap 的使用狀況：

- Hot Node 因為會處理 Indexing 的請求，所以 JVM heap 會有一定比例拿來處理 Indexing。
- Warm Node 不處理 Indexing 所以只會有一半 JVM heap 拿來處理 query request，剩下的會來給 Lucene 用作暫存。
- Cold Node 不會處理 Indexing，而且針對 query request 所產生的 transient cache 也會一用完就盡快的釋放，減少 heap 的使用。



## Freeze Indices

沒錯，所謂的 code data 就是被 `freeze` 的 indices。

這邊解釋一下 Elasticsearch 為什麼要特別建立這個 Freeze 的機制。

一般的 Index，隨著搜尋的用法等不同，JVM heap需要的大小與 Index 存放的資料量比例，大約是 1:8 ~1:100 不等，因此這會限制了某個 Index 能存放的資料量的大小。

但是我們在較少用的舊資料上，我們希望能有更有效的資源使用，也就是一個同樣記憶體大小的 Node 能存放更多的資料、也願意犧牲一些查詢時的反應速度，因此 Elasticsearch 就設計的這個 Freeze 的機制，讓被 Freeze 的 Index 能盡量減少 heap 的使用。



### Freeze API

要把一個 Index freeze 或 unfreeze 使用的方式如下：

```
POST /my_index/_freeze
POST /my_index/_unfreeze
```

> 目前 Freeze Index 時，Elasticsearch 會在背後將 Index close 再重新打開，所以 cluster 的狀態會短暫進入 `紅燈` 直到 index 重新打開時，primary shard 被重新載入。

> 要注意的另一件事是，freezed Index 是 `read-only` ，也就是連 Segment file 的 `_forcemerge` 都不能操作，所以要 Freeze 之前，較好的 practice 是先記得將 Index 進行 segment file 的 force merge。
>
> ```
> POST /sampledata/_forcemerge?max_num_segments=1
> ```



### 搜尋被 Freeze 的 Index

為了避免 Freezed index 在無意識的情況下被存取到， Freezed Index 是會被指定為 `throttled` 的，也就是預設 Elasticsearch 的搜尋，是不會查到 Freezed  index 的資料，若是要查詢包含 Freezed index 裡的資料，需要在搜尋時帶上 `ignore_throttled=false` 的參數。

```
GET /sampledata/_search?ignore_throttled=false
{
 "query": {
   "match": {
     "name": "jane"
   }
 }
}
```



### 在 Kibana 搜尋到 Freezed Index 裡的資料

若是在 Kibana 裡，想要搜尋 Freezed Index 的資料的話，需要從以下的地方進入：

左滿 Menu 選單底下的 `Stack Management` > 找到 Kibana 區塊裡的 `Advanced Settings` > 找到 Search 區塊裡的 `Search in frozen indices` 並設定成 `On`。

![kibana search in frozen indices](https://i.imgur.com/DDrHDi3.png)



以上是 Hot-Warm-Cold Architecture 的介紹及原理，下一篇將會介紹如何透過 Index Lifecycle Management 來達到輕鬆設定、並直接利用 Rollover, Shrink, Hot-Warm-Cold Architecture 的機制來有效的管理我們的 Indices。



## 參考資料

- [官方文件 - Index Lifecycle Management](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/index-lifecycle-management.html)
- [官方 Blog - Creating frozen indices with the Elasticsearch Freeze index API](https://www.elastic.co/blog/creating-frozen-indices-with-the-elasticsearch-freeze-index-api)
- [Efficiency in Elasticsearch](https://coggle.it/diagram/XIpfGBc5Ajc-0zoI/t/efficiency-in-elasticsearch)

