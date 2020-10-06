# 喬叔教 Elastic - 11 - 管理 Index 的 Best Practices (3/7) - Index Lifecycle Management (ILM)

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

先前我們介紹過了 Index, Shard, Segment Files, Hot-Warm-Cold Architecture，知道在不同的時期要有效率的使用 Index 可以進行什麼樣的配置，接下來要介紹的是全面的 Index Lifecycle 的機制，就會使用到之前介紹的各種機制，來進行 Index 生命週期的管理。

### 進入此章節的先備知識

- Elasticsearch Index, Shard, Segment File 的基本認識。
- Index Template 的基本認識。
- Rollover 與 Shrink 的機制。
- Hot-Warm-Cold Architecture。

### 此章節的重點學習

- Index Lifecycle Management (ILM) 的使用方式。
- 如何在 Elastic Cloud 中的 Kibana 來設定 ILM。
- ILM 設定中背後的原理。

---

## Index Lifecycle Management (ILM)

Index Lifecycle Management 顧名思意，就是用來管理 Index 的生命週期，而在 Elasticsearch Index 生命週期主要定義有四個段。

### Index Lifecycle Management 主要的 4 個階段

- **Hot:** 最新的資料，通常是用來放最新的資料。 `可以寫入、可以查詢`
- **Warm:** 資料進來後，不再寫入時，但還是會常常的查用，通常會放在這個階段。 `不能寫入、可以查詢`
- **Cold:** 資料已放蠻久的，不常使用到，但還是希望需要用到時能馬上就能用，但願意接受速度較慢一些，就會放在這個階段。 `不能寫入、可以查詢(但較慢)`
- **Delete:** 資源總是有限，空間也是有限，時間久了總是有些舊資料應該要從 Elasticsearch 中刪掉。

![ilm four phases](https://i.imgur.com/NmmfRFd.png)

上圖可以解釋當資料隨著時間變化，會不斷產新的 Index，並且隨著時間變化 (流水號數字愈大的是愈新的，數字愈小是時間愈久)，會逐漸移到下一個階段中。

在 ILM 中，你可以建立一個 **Policy** 來指定想要設定哪些階段、以及每個階段要進行的 Action (動作) 是什麼，而在 ILM 中可以執行的動作有以下這些。



### Index Lifecycle Management 中可以用的 Action 有哪些

- **Rollover:** 當原 index 達到 **一定的大小**、**資料筆數**、**資料存放一定時間** 時，自動建立新的 Index 來放新進來的資料，不會讓某 Index 一直無限的增長下去。這個動作可以針對 Index Alias 或 Data Stream 進行設定。
- **Shrink:** 將多個 Shard 的 Index 轉成較少 Shard 數量的 Index。
- **Force merge:** 將一個 Shard 中的 Segment Files 進行合併，可以釋放出被刪掉的文件在原先 `read-only` 的 Segment File 所佔用的空間，也能加快查詢的速度。
- **Freeze:** 將很少使用的 Index，以盡量不使用到 heap size 的方式來存放，
- **Delete:** 刪掉 Index。
- **Allocate:** 指定 Index replica 的數量，以及指定 Index 可以被放在哪些 shards 的規則。([Index-level shard allocation filtering](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/shard-allocation-filtering.html#index-allocation-filters))
- **Set Priority:** 指定 Index 的處理優先權，也就是當 node 重新啟動的時候，較高優先權的 Index 會先被 recover 而優先回到可被使用的狀態。
- **Unfollow:** 將 Cross Cluster Replication 機制中，會 follow 的 Index 給取消 follow。在 Rollover, Shrink 處理時會自動執行這個 Action。
- **Wait for snapshot:** 等到 Snapshot 完成後才能刪除 index。



### 每個階段可以進行的 Action 有哪些

| Index Lifecycle Management 的階段 | 此階段中可以執行的操作有什麼                                 |
| --------------------------------- | ------------------------------------------------------------ |
| **Hot**                           | `Force merge`, `Rollover`, `Set priority`, `Unfollow`        |
| **Warm**                          | `Allocate`, `Force merge`, `Read only`, `Set priority`, `Shrink`, `Unfollow` |
| **Cold**                          | `Allocate`, `Freeze`, `Set priority`, `Unfollow`             |
| **Delete**                        | `Delete`, `Wait for snapshot`                                |



## 透過 Kibana 來設定 Index Lifecycle Policies

我們從 Kibana 左側選單底下的 **Stack Management** 進入後，可以看到以下的畫面，左邊有個 **Index Lifecycle Policies** 可以進入 ILM 的管理畫面。

![kibana ilm](https://i.imgur.com/sFPKPO3.png)

點右上的 **Create Policy** 即可開始建立。

### Hot phase

在這個 Hot phase 的設定中，主要包含了前面介紹的一些設定 `Rollover`, `Index Priority`，不過在 Kibana 的畫可上，並沒有讓你設定 `Force merge`，畢竟這操作並不是適合用在一般的 Hot phase。

![index lifecycle policy - hot](https://i.imgur.com/PUXLup8.png)



### Warm Phase

在 Warm Phase 中，可以指定當 Hot Phase 中的 Index 發生 Rollover 時，是否直接把 Index 移到 Warm Phase 的階段。

若要移的話，會需要指定什麼樣的 attribute 是屬於 Warm Phase。

若使用 Elastic Cloud，他只有兩種 attribute - `data:hot` 和 `data:warm`，這也是依照硬體規格來配置過的。

除此之外可以指定 Shrink, Force merge, Index priority 等設定。

![index lifecycle policy - warm](https://i.imgur.com/DthWuFS.png)



### Cold Phase

在 Code Phase 的階段，也如同 Warm Phase 一樣，可以指定當 Rollover 發生多久之後 (若沒有啟動 Rollover，則是以 Index 建立的時間來判斷)，要把 Index 移到 Rollover，不過因為 Elastic Cloud 目前提供的配置中，沒針對 Cold Phase 規劃的硬體配置，因此還沒辦法直接選擇，相信不久的將來會推出的。

另外可以指定是否要啟動 Freeze，以及是否要指定 Index Priority。

![index lifecycle policy - cold](https://i.imgur.com/fXashHi.png)



### Delete Phase

Delete Phase 的設定很簡單，就是要在 index rollover 發生多久之後 (若沒有啟動 Rollover，則是以 Index 建立的時間來判斷)，要刪除這個 Index。

另外可以指定，刪除之讀是否確保這個 Index 已經有備份過，這個 Snapshot Policy 會在這系列文章的面後有所介紹。

![index lifecycle policy - delete](https://i.imgur.com/0d68Wfc.png)



## Index Lifecycle Policy 與 Index 之間的關係

當我們  Index Lifecycle Policy 設定好之後，在主畫面的 **Actions** 可以看到有兩個設定。

![index lifecycle policy vs index](https://i.imgur.com/xf6nwlU.png)

1. **View indices linked to policy:** 點下就會跳到 Index Management 的頁面，並且過濾出被這個 Indes Lifecycle Policy 所管理的 Indices 有哪些。
2. **Add policy to index template:** 這個可以指定要套用到哪個 Index Template，讓新的 Index 被建立時，自動套用這個管理機制。
   ![index lifecycle policy apply index template](https://i.imgur.com/EBtb8rg.png)



### 如何檢視 Index Lifecycle Policy 執行的狀況

當 Index Lifecycle Policy 建立起來之後，若想知道某個 Index, Data stream, Index Alias 所對應的 Index Lifecycle Policy 及執行的狀況、目前在哪個階段…等，可以透過 **Explain lifecycle API** 來查看：

```
GET my-index-000001/_ilm/explain
```

以下是這個 Explain API 回傳的結構：

```
{
  "indices": {
    "my-index-000001": {
      "index": "my-index-000001",
      "managed": true, 
      "policy": "my_policy", 
      "lifecycle_date_millis": 1538475653281, 
      "age": "15s", 
      "phase": "new",
      "phase_time_millis": 1538475653317, 
      "action": "complete",
      "action_time_millis": 1538475653317, 
      "step": "complete",
      "step_time_millis": 1538475653317 
    }
  }
}
```

若是有正在執行中的步驟，也都會有詳細的資訊回傳，若有錯誤發生，也都能看得到。

建議可以上 [官方文件 - Index Lifecycle Management Explain API](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/ilm-explain-lifecycle.html) 看更多詳細的介紹。

若有錯誤時，在排除完成後，也可以直接使用 Retry API 來進行重試。

```
POST /my-index-000001/_ilm/retry
```



## 參考資料

- [官方文件 - Index Lifecycle Management](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/index-lifecycle-management.html)
- [官方文件 - Index-level shard allocation filtering](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/shard-allocation-filtering.html#index-allocation-filters)
- [官方文件 - Index Lifecycle Management Explain API](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/ilm-explain-lifecycle.html)
- [这么简单的ES索引生命周期管理，不了解一下吗～](https://zhuanlan.zhihu.com/p/137810661)

