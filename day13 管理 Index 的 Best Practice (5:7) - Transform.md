# 喬叔教 Elastic - 13 - 管理 Index 的 Best Practice (5/7) - Transform

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

這一系列的文章分別介紹了隨著時間不斷增長資料的 Index 該如何管理，除了透過 Index Lifecycle Management 移除舊的資料、透過 Rollup 把資料顆粒度變大、這邊要介紹的是透過 Transform 的功能，把資料進行轉換後再儲存。

### 進入此章節的先備知識

- Elasticsearch Query DSL。
- 一定程度的了解 Aggregation 的使用方式。
- 資料分析的基本觀念。

### 此章節的重點學習

- 如何使用 Transform 將 Index 資料轉換成另外的資料檢視維度並儲存至另個 Index 中。
- 如何在 Kibana 上建立 Transform 並使用他的結果。

---

## 什麼是 Transform

Transform 是 **Elasticsearch 7.2** 推出的強大新功能，這是在 `X-pack` 中的 `Basic License` ，所以 Open Source 版是沒有這個功能的，Transform 主要目的是將 Index 資料透過 **Pivot (樞杻)** 分析的方式運算後轉換成另外的資料檢視維度並儲存至另個 Index 中，這個資料轉換與整理的機制，背後就是用 Aggregation 來進行處理，但既然 Aggregation 就能做這這個 Transform 的處理，為什麼還要用 Transform 呢？下面會介紹什麼時候該用 Transform 。

### 何時使用 Transform 而不是用 Aggregation

- 如果你要取得的是 Aggregation 後的所有結果，而不是只有 top-N 的資料時。
- 如果你有使用一些資料，會定期的要撈取 **先前一段時間的訪問量(例如是 Unique User Visit Count)**、**某段時間的各類型的產品總銷售數字**、**某段時間某個商品被瀏覽的次數**…等，這種有特定功能、且每次要查的時候會耗較多運算資源的資料。
- 如果你在進行資料分析時，腦袋裡想到的解法都是要用許多 SQL 語法中的 `Group By ... Having ...`、`Sub Query + Where 或 Order By`…等在 Elasticsearch 要用到大量的 Bucketing + `bucket selector` …甚至你不知道要怎麼寫這個 Aggregation。
- 如果你在使用 `Pipeline Aggregation` 時，卻又要使用到排序，且遇到 Elasticsearch 目前並不支援這樣的操作。
- 如果你想將一些 Summary 的 Aggregation 結果另外存起來，減少每次查詢所要消耗的查詢資源。



## 使用 Kibana 建立 Transform

接下來直接進入 **Kibana** > **Stack Management** > **Transform** 的畫面，來建立一個 Transform。

### Create Transform

建立 Transform 時，第一步會要選擇資料的來源，這邊會需要先在 Kibana 建立好 Index Pattern。

![new transform](https://i.imgur.com/XAQdL8d.png)



接下來會要決定 `Group By` 的欄位會有哪些，以及資料分析時要有哪些 Aggregation 的運算。

![create transform](https://i.imgur.com/9Wxfdld.png)



在資料來源的檢視畫面上，可以切換 `Histogram charts` ，可以看到每一個欄位的資料分步狀況。

![creat transform - histogram](https://i.imgur.com/O2HxgZt.png)



在選擇好欄位後，底下會可以預覽這個 Transform 的 Aggregation 查詢的結果。

![create transform](https://i.imgur.com/Xh8QYqN.png)



確認資料彙總的方式沒問題後，接下來設定 Transform 的基本資訊，包含 Transfor ID, 產出結果存放目的地 Index 名稱，若是有選擇時間的 Group By 條件，也會要指定 Time Filter field name，再來指定 Date field 用來判斷資料是否有更新，最後是 Delay。

![transform details](https://i.imgur.com/gH3BXCL.png)



一切就緒後，就是執行 `Create and start` ，若好奇這個 UI 產生出來的 Create Transform API payload 長什麼樣，也可以選擇 `Copy to clipboard` ，並貼到 Kibana Dev Tools 去執行。

![create tranform](https://i.imgur.com/bv6PITw.png)



執行後，可以到 Transforms 頁面看建立的 Transform Job，或是到 Discover 查詢 Tranform 出來的資料內容。

![image-20200927225028081](https://i.imgur.com/H9yeJdp.png)



### 查看 Transform 的結果

建立好之後，在 Transforms 畫面可以看到這個 Transform job 正在執行，也可以看到他的 Status。

![image-20200927225117950](https://i.imgur.com/vh8aQTh.png)



若我們進入到 Discover 頁面，可以發現從 Kibana 建立的 Transform 自動也建立了 Index Pattern ，所以可以直接選擇這個 Index Pattern 來瀏覽資料。

![image-20200927225404534](https://i.imgur.com/qHw7WSU.png)



若直接從 `_search` API 來查看裡面的資料，每一個 Document 很單純的就是我們所指定彙總的結果。

![search transform data](https://i.imgur.com/yxWMCdu.png)



一但有了這個 Transform 後的資料，也可以直接用 Virtualize 以這個資料來源拉出想要分析的資料圖表。

![virtualize transform data](https://i.imgur.com/s3yY0HQ.png)



接下來也可以把多個 Virtualize 的圖表拉到 Dashboard 中，你會發現這個直接從 Transform 的結果當資料來源，執行速度比原先快上非常的多，而且還可以再進一步使用 Aggregation 來達到更多變化的應用！



## 注意事項

- Transform 是一個會定期執行、而且是使用 Aggregation 這樣很耗資源的查詢處理，所以在建立 Transform 時，要注意系統的資源與執行的資料範圍、適度的使用 `docs_per_second` 的節流設定，避免造成 Elasticsearch Cluster 服務的穩定性。
- 當使用 Transform 時，目的地的 Index 也需要先建立好 Mapping 以免造成 Dynamic Mapping 的結果不如預期，所以記得先 Create Index 或使用 Index Template。
- Transform 在執行時，為了有效的處理所有的 buckets 的分頁，會使 Composite Aggregation 來進行處理，而這個處理在進行中的時候，如果源頭的資料有新增、修改、刪除，不一定會被包含在這次 aggregation 的結果中。
- Transform 使用的是 Composite Aggregation ，因此在 Buckets 的數量限制上、及 terms query 的 `terms` 數量限制上，都可能會需要依照使用的情境來做對應的調整，相關的設定在 `max_page_search_size` 及 `index.max_terms_count`。
- 如果有自己調整 Transform 的 `Frequency` 時，要注意，這個值同時也是 Transform 發生錯誤時，retry 的機制使用的間隔時間。
- 使用 Transform 時，如果執行的當下，資料還在 indexing 的處理中、還無法被 search 出來時，這樣 Transform 執行的結果可能會沒有包含到這些資料，而且會標記成已經處理過，下一次資料沒有異動的話也不會再處理，所以要設定好 `sync.tim.delay` 的值，要有適度的時間等讓 indexing 的資料完成處理，並且可被搜尋到，特別是要注意是否有正確的對應到 `index.refresh_interval`的值。



## 參考資料

- [官方文件 - Transforms](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/transforms.html)