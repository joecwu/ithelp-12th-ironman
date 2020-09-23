# 喬叔教 Elastic - 04 - Index 建立前你該知道的 (1/5) - ES Index 如何被建立

### 進入此章節的先備知識

- 初步了解 Elasticsearch 的 Cluster 與 Shards。

### 此章節的重點學習

- Elasticsearch 的 Index 是如何被建立，在 Cluster 中是如何運作。

---

當我們建立好一個 Elastic Cloud 的 Deployment 之後，下一步就是要建立 Index，但是 Index 建立時的底層是如何運作的呢？

## Elasticsearch Index 的建立

假設有一組擁有 4 個 Nodes 的 Elasticsearch Cluster：

- index: `a`, shard: 2, replica: 1 (有 `a0`, `a1` 兩個深色的 primary shard, 另各有一份白底的 replica shard)
- index: `b`, shard: 3, replica: 1 (有 `b0`, `b1`, `b2` 三個深色的 primary shard, 另各有一份白底的 replica shard)

![es-node](https://i.imgur.com/6IZ7b0c.png)

當我們要建立一個新的 Index `c` 時，Elasticsearch 會執行以下幾個檢查的步驟：

### Shard 分配決策者 (Allocation Deciders)

1. 計算每個 node 身上的 shard 數量，盡可能的**以數量**來平均分配，決定新的 primary shard 要放在哪個 node 身上。
2. 檢查是否有些 filter 條件，例如當 node 有宣告 hot/warm architecture 的 attribute 時，會過濾掉不應該被存放的 nodes，例如新的資料只能被放在有宣告 `hot` attribute 的 node 身上。

![image-20200915024738494](https://i.imgur.com/7dhVeNt.png)

3. 檢查磁碟空間是否充足，如果已達到 `cluster.routing.allocation.disk.watermark.low` 設定的水位，則不會將新的 shard 放在該 node 身上。

![image-20200915023728618](https://i.imgur.com/W1r0NaJ.png)

4. 檢查是否有設定 Throttling，例如 `indices.recovery.concurrent_small_file_streams` 和 `indices.recovery.concurrent_file_streams` 的設置是否達到，而是否要暫緩目前 create index 的動作。

### Primary Shard 初始化

若 Shard Allocation Decider 一切順利，將會進行 Primary Shard 的初始化：

1. 在 Cluster 狀態中，標示這個 Index 的 Primary Shard 會被分派到哪個 node 身上，並標示狀態為 `Initializing`。
2. 該 Index 存在的 node 收到動工的通知後，開始建立空白的 Lucene index，並且回報 Cluster master node 處理完成。
3. Cluster master node 收到處理完成時，會將這個 shard 的狀態標示為 `started`，並且通知 cluster 中的大家這個狀態，而這個存在此 pirmary shard 的 node 也收到 master node 的通知時，就會將這個 shard 的狀態設定成 `started`，這時就能提供 indexing 的處理了。

### Replica Initialization

當我們 Primary Shard 正常運作之後，Cluster 會檢查目前 replica 的設定是否滿足，若是有需要執行 replica 的複制則進行下列步驟：

1. 決定 shard 應該放在哪一個 node 身上，通知這個 node 要做事，並且標示這個 shard 為 `Initializing`。
2. 收到通知的 node 為此 shard 建立空白的 Lucene index。
3. 一律從 primary shard 複制資料到 replica shard，並且在完成之後通知 master node。
4. Master node 將這個 shard 的狀態改為 `started`，並且通知 cluster 中的大家。
5. 收到 master node 通知時，存放 replica shard 的 node，將 shard 的狀態開啟提供服務。

到此階段，這個 Index 算是完成了被建立的這個流程，也開始能正常的提供服務了，下一步就是將要 indexing 進 Elasticsearch 的文件準備好吧！



## 參考資料

- [官方 Blog - Every shard deserves a home](https://www.elastic.co/blog/every-shard-deserves-a-home)