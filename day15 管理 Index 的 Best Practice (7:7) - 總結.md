# 喬叔教 Elastic - 15 - 管理 Index 的 Best Practice (7/7) - 總結

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

這個系列的文章總共介紹了各種 Index Management 的管理方式，最後這篇會融合各種方式，以全貌的方式來看 Elastisearch 的 Index 管理方式，如何應用前面介紹的各種機制，來建立完整的 Index Management。

### 進入此章節的先備知識

- Elasticsearch Index, Shard, Segment Files 的基本知識。
- 建議可先閱讀此系列文章的前面章節部份，或是也可以將這篇當成 overview，不清楚的部份再回頭去對照前面章節的詳細介紹。

### 此章節的重點學習

- 在 Elasticsearch 中，如何建立一個完整的 Index 管理機制。

---

## Elasticsearch Index Management Overview

進入 Index 的管理，有以下幾個重點：

- **Segment Files 的數量**：數量愈多對 **查詢的速度** 與 **磁碟的空間** 愈不好。
- **Shard 的數量**：愈多對 **Indexing 的速度** 愈好，但對 **查詢成本較高** ，單一 Shard 愈大對 **Cluster 的 Rebalance** 的成本愈高。
- **Index 的大小**：愈大對於 **查詢效率** 愈好，以 Time-based 資料來看的話，影響的是資料移轉到下一個階段的等待時間。
- **資料的新舊程度**：新的資料通常 **使用頻率** 較頻繁，會給較好的硬體資源、較舊的資料較少使用，可配置較差的硬體資源。
- **時間顆粒度**：當資料量很大時，在觀察過往的資料往往時間顆粒度會抓較大，並且觀察的彙總的結果 (例如：每天的 log 數量、每天的銷售金額、每天的觀看次數…等)。
- **Index 愈來愈多**：資源總是有限，太舊的資料會面臨刪除，可保留的是彙總的結果。
- **資料的安全性**：除了存取控制要妥善的限制之外，資料的備份也是非常重要的機制。



這邊提供一個 Index Lifecycle Management 的概觀圖：

![Elasticsearch Index Management Overview](https://i.imgur.com/Bk35OlJ.png)

隨著資料的進入，以下是主要的管理階段：

1. 使用 **Index Lifecycle Management** 來管理資料的 Hot Warm Cold Architecture：
   1. 新的資料大量寫入 Hot Nodes，所以會配置 **較多的 Primary shards**。
   2. 隨時間及資料量的成長，為了確保 Index 的資料量在有控制的範圍、以及讓 Hot data 進入 Warm data 以確保 Hot node 的硬體資源分配，會透過 **Rollover** 將 Index 進行 rotate ，產生新的 Index 來接新的資料，而原先的 Index 會進入下一個 Warm 的階段。
   3. 進入到 Warm data 的階段，會進入 read-only ，所以會透過 **Force Merge** 與 **Shink** 將 Segment Files 數量 與 Shards 數量進行最佳化，也可同時配合 **Compress** 進行儲存空間的優化。
   4. 時間過更久之後，資料進入 Code data 階段，會將 Index 進行 **Freeze** ，以減少 JVM heap 的使用量，提供較高的延遲反應，但還是能即時使用的服務狀態。
   5. 再更久的資料，可再確認已經被備份過之後進行 **刪除**。
2. 使用 Rollup 將資料以較大的時間顆粒度來儲存：
   1. Rollup 可以從 Hot, Warm, Cold 任何的 Index 當中把資料讀出，並且以較大的 **時間顆粒度** 進行彙總運算，並將結果儲存新的 Index 中。
   2. Rollup 是定時執行，同時 Rollup 的資料在透過 `_rollup_search` 查詢時，可**混搭 Live Data + Rollup Data**，結果會自動合併並去除掉重覆的，也因此當舊的資料被刪除後，存在 Rollup 的資料一樣能提供彙總後的查詢結果。
3. 使用 Transform 將資料以另外的分析結果來儲存：
   1. 將資料以 **Pivot** 的方式進行分析運算、並將結果儲存在獨立的 Index 中。
   2. 適合複雜的 Aggregation 的運算及彙總報表的定期處理工作。
   3. 也可以這個機制從查詢的彙總結果建立成 Index，並當作 Ingest 處理時 Enrich 資料的 Lookup Data Source。
4. 使用 Snapshot Lifecycle Management 來管理資料的備份。
   1. 設定 **定期的備份**。
   2. 設定備份的 **保存時間及數量**，確保備份所佔用的空間不會無限增長。



以上是 Elasticsearch 針對 Index 管理上的各種使用方式的搭配組合，建議搭配前面的文章進行細部的探討，透過這樣的概念進行資料的管理，可以得到較佳的資源使用與配置的規劃安排。



## 參考資料

- 請參考本章節的前面幾篇介紹