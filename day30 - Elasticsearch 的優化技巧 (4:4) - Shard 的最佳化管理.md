# 喬叔教 Elastic - 30 - Elasticsearch 的優化技巧 (4/4) - Shard 的最佳化管理

**Elasticsearch 的優化技巧** 系列文章索引

- [(1/4) - Indexing 索引效能優化](https://ithelp.ithome.com.tw/articles/10252325)

- [(2/4) - Searching 搜尋效能優化](https://ithelp.ithome.com.tw/articles/10252695)

- [(3/4) - Index 的儲存空間最佳化](https://ithelp.ithome.com.tw/articles/10253058)

- [(4/4) - Shard 的最佳化管理](https://ithelp.ithome.com.tw/articles/10253348)

---

## 前言

這系列的文章主要的目的在於當我們開始使用 Elastic Stack 時，我們如何優化 Elasticsearch 的使用方式，包含 Indexing, Searching, Disk Usage, Shard Optimization 等四個主題，這篇以是 Shard Optimization 為主的介紹。

### 進入此章節的先備知識

- 已經有在使用 Elasticsearch，並且了解 Elasticsearch 的基本原理與操作方式。

### 此章節的重點學習

- 如何規劃屬於你的 Sharding Strategy 來管理 Elasticsearch Cluster 中的 Shards。
- 當 Elasticsearch Cluster 已經 oversharding 了，要如何修正。

---

## Shard 的最佳化管理方式 - Sharding Strategy

為了避免機器毀損時造成資料的遺失以及提升 indexing 或是 searching 的資料處理能力，Elasticsearch Cluster 在多個 nodes 並包含多個 shards 的分散式資料架構中，儲存 index 資料和這些資料的 replica，但是這些 shards 的數量，以及 replica 的數量，對於 cluster 的健康狀態及效能其實有很大的影響，最常遇到的一個問題就是 `oversharding` (過多的分片)，而這種 shard 數量太多的狀況可能會影響到 cluster 的穩定性，因此在管理 Elasticsearch Cluster 時，最好先了解你的資料，並規劃好 **Sharding Strategy** ，以下是規劃時建議要考量的項目。

- Search 在執行時，一個 Shard 會分配一個 Thread，數量太多反而會拖慢速度
- 每個 Shard 都有基本運作的成本，數量太多成本愈高
- 指定 Shard 在 Cluster 中的分配方式，以優化儲存硬體的資源
- 刪除整批資料時，以 Index 為單位來刪除，而不要從一批 Documents 來刪除
- 建議使用 Data Stream 或 Index Lifecycle Management (ILM) 來管理 time series 資料
- 一般會建議讓單一 Shard 的大小在 10 ~ 50 GB 左右
- 1GB 的 Heap Memory 大約能處理 20 個 Shards
- 一個 Index 有多個 Shard 時，避免大多數的 Shard 都被分配在同一台 Node 身上

底下分別是這些建議的個別說明。

### Search 在執行時，一個 Shard 會分配一個 Thread，數量太多反而會拖慢速度

大多數的 searching 在執行時，查詢的資料是會橫跨多個 shards，而每個 shard 在搜尋時會使用一個 CPU thread 在處理執行，也就是：

- 如果有多個 shard，可以讓查詢的處理並發在多個 shard 身上同時進行，以增加處理的效率。
- shard 數量愈多，node thread pool 的消耗也愈多，因此 thread pool 數量不足的話，反而會影響查詢的執行效率。

> 一個 Node 依照不同的功能分類有多種 Thread pool， 其中 **search** (count/search/suggest) 的 size 是 `int((# of allocated processors * 3) / 2) + 1` 並且 queue_size 預設是 `1000` ，而 **write** (index/delete/update/bulk) 的 size 是 `# of allocated processors` ，預設的 queue_size 也是 `1000`。( [官方文件 - Thread pools](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html) )



### 每個 Shard 都有基本運作的成本，數量太多成本愈高

每個 Shard 固定都會佔用一些 CPU 和 Memory 的資源，以同樣的資料量，Shard 數量愈多，overhead 也就愈高，佔用的總系統資源也會較多一些。

Shard 的另外的資源成本有一部份會是裡面的 sgement files，segment files 的 metadata 會被存放在 JVM heap memory 中，以提供搜尋時處理的加速，而 segment files 在經過 merge 後，也會將已標示為刪除的資料給移除，加上 segment files 數量變少，佔用的 heap memeory 也會較少。



### 指定 Shard 在 Cluster 中的分配方式，以優化儲存硬體的資源

預設是自動分配，而 Elasticsearch 也就會盡可能的平均分配，而我們可以使用 [shard allocation awareness](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#shard-allocation-awareness) 的機制來自己決定 shard 被分配的規則，並依照業務的需求來分配不同等級的硬體給不同的資料。

如果是 time-based 的資料，也可以把時間較舊、使用率較低的資料，分配到較次等的硬體上，以優化儲存硬體的成本，這部份也可以參考 Hot-Warm-Cold architecture 的運作機制來處理。



### 刪除整批資料時，以 Index 為單位來刪除，而不要從一批 Documents 來刪除

Document 的刪除，在 Elasticsearch 的運作上，是產生另外一筆 "標示刪除" 的記錄在 segment files 中，所以這些都還是會持續的佔用系統資源，一直到 segment merge 之後，才會真正的被移除。

因此如果會週期性的刪除一批舊資料時，最好能以 Index 為單位來刪除，而不是透過 delete_by_query 之類的批次刪除機制。



### 建議使用 Data Stream 或 Index Lifecycle Management (ILM) 來管理 time series 資料

Index Lifecycle Management 裡面可以定義 **自動 Rollover** 的機制，也就是當資料量成長到 **某個數量**、**某個時間**、**某個大小** 時，會自動產生新的 Index 來放新的資料，而太舊的資料也能設定自動刪除。

使用 ILM 是個很好來管理 Shard Strategy 的工具，因為他能很輕易的進行策略的調整：

- 因為 ILM 都會透過 Index Template 來管理 Indices，所以要改變 primary shard 數量時，直接改 index template 即可。
- 想要 shard 總數成長慢一點、單一 shard 的大小要大一點時，只要修改 Rollover 的配置規則。
- 當 shard 數量太多、舊資料要刪掉時，只要修改 delete phase 的規則。



### 一般會建議讓單一 Shard 的大小在 10 ~ 50 GB 左右

Shard 太大時，會讓 Cluster 進行 shard recover 時的成本太高。例如一個 node 死掉時，Cluster 會嘗試進行 rebalance ，這時要將某個 node 身上的 shard 搬到另一個 node 時，這個搬家所消耗的頻寬與處理的系統資源都會是成本。

> 不過這個還是要依照 Cluster 的硬體規格及 Node 的數量來進行全面的評估。



### 1GB 的 Heap Memory 大約能處理 20 個 Shards

一個 node 能處理的 shard 數量大約會和 JVM heap memory 大小成一定的比例，以官方統計的數字，一般是每 GB 的 heap memory 大約可以處理 20 個 shards，也就是 30GB 的 heap memory 可以處理 600 個 shards，不過這同樣的還是要依照 Elasticsearch Cluster 的硬體規格與使用狀況來評估。

若要查看 shard 數量，可以從 _cat API 來看

```
GET _cat/shards
```



### 一個 Index 有多個 Shard 時，避免大多數的 Shard 都被分配在同一台 Node 身上

當我們將一個 Index 設定有多個 primary shard 時，主要的目的就是為了 indexing 時能有更多的 Nodes 能分擔處理，但一個 Cluster 可能有許多的 Index ，在各種混合分配的情況下，若是 shard allocation 的機制剛好把這個 Index 把 shard 分到同一個 Node 身上的話，這樣就達不到我們要的目的。

因此可以透過 `index.routing.allocation.total_shards_per_node` 的設定，來限制每個 node 可以被分配存放這個 index 多少個 shard，設定方式如下：

```
PUT /my-index-000001/_settings
{
  "index" : {
    "routing.allocation.total_shards_per_node" : 5
  }
}
```



## 修正 oversharging (過多的 shards) Cluster 的方法

當 Cluster 已經因為太多的 shards 導致不太穩定時，我們可以透過以下的方式來進行調整或修正。

- 使用較長時間區間的 time-based indices 配置方式
- 刪掉空的或沒必要的 Indices
- 在系統較不忙的時段，執行 Force Merge 來合併 Segment Files
- 透過 Shrink API 將現有的 Index 的 shard 數量變少
- 透過 reindex API 來合併較小的 indices

底下分別是這些建議的個別說明。

### 使用較長時間區間的 time-based indices 配置方式

針對 time-based 資料，我們可以提高切 Index 的時間顆粒度，例如本來是 1天 切一個 Index，我們可以改成 1個月、或 1年 來切 Index。

如果是使用 Index Lifecycle Management 來處理 time-based indices 時，就可以透過提高 `max_age`, `max_docs`, `max_size` 的這些配置來達到同樣的目的。



### 刪掉空的或沒必要的 Indices

如果是使用 ILM 的 `max_age` 機制來進行 Index 的 rollover 時，有可能會產生出空的 index，這些空的 indices 也會佔用到一些系統資源，這樣的 Index 應該查詢出來並且刪除。



### 在系統較不忙的時段，執行 Force Merge 來合併 Segment Files

透過 Segment Files 的 merge 來減少佔用的系統資源與空間，將資源能提供給其他的任務。

也因為 forcemerge 的執行時很佔用系統資源，所以建議要在系統較不忙的時候來做這個動作。

```
POST /my-index-000001/_forcemerge
```



### 透過 Shrink API 將現有的 Index 的 shard 數量變少

如果 Index 已經不再會寫入新的資料時，可以透過 Shink API 來將 primary shard 的數量減少。詳細的設定方式可以參考先前的文章，或是 [官方文件 - Shrink index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-shrink-index.html)。

若是使用 ILM 的話，也可以在 warm phase 時設置 Shrink 的處理。



### 透過 reindex API 來合併較小的 indices

例如原先我們是以 **日** 來當作 time-based indices 的切割單位，若是 index 數量太多，因此造成 shard 數量太多，我們可以透過 reindex 的方式這些 indices 的資料合併到以 **月** 為單位的 index。

```
POST /_reindex
{
  "source": {
    "index": "my-index-2099.10.*"
  },
  "dest": {
    "index": "my-index-2099.10"
  }
}
```

這樣也會是一種減少 shard 數量的方式。



## 參考資料

- [官方文件 - How to size your shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html)
- [官方文件 - Thread pools](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)
- [官方文件 - Shrink index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-shrink-index.html)

