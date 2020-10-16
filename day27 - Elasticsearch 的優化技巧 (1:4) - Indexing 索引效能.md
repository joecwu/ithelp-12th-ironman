# 喬叔教 Elastic - 27 - Elasticsearch 的優化技巧 (1/4) - Indexing 索引效能優化

**Elasticsearch 的優化技巧** 系列文章索引

- [(1/4) - Indexing 索引效能優化](https://ithelp.ithome.com.tw/articles/10252325)

- [(2/4) - Searching 搜尋效能優化](https://ithelp.ithome.com.tw/articles/10252695)

- [(3/4) - Index 的儲存空間最佳化](https://ithelp.ithome.com.tw/articles/10253058)

- [(4/4) - Shard 的最佳化管理](https://ithelp.ithome.com.tw/articles/10253348)

---

## 前言

這系列的文章主要的目的在於當我們開始使用 Elastic Stack 時，我們如何優化 Elasticsearch 的使用方式，包含 Indexing, Searching, Disk Usage, Shard Optimization 等四個主題。

### 進入此章節的先備知識

- 已經有在使用 Elasticsearch，並且了解 Elasticsearch 的基本原理與操作方式。

### 此章節的重點學習

- Indexing 的效能優化的各種技巧與建議。

---

## Indexing 索引效能優化

這篇文章主要提供 Indexing 的效能優化的各種技巧與建議：

- Indexing 大量資料時，善用 bulk request
- 使用 multi-thread / multi-workers 來 indexing 資料進入 Elasticsearch
- 調低或暫時關閉 `refresh_interval`
- 指定 Routing 的方式，減少 Thread 的數量
- 第一批資料 indexing 進入 Elasticsearch 之前，先不要設定 Replica
- 關閉 java process swapping
- 確保 Filesystem 有足夠的 memory cache
- 使用 auto-generated ids
- 使用更快速的儲存硬體
- 調高 indexing buffer 大小
- 使用 cross-cluster replication 的配置讓 searching 的處理不會佔用 indexing 的資源
- 調整 Translog 的 Flush 設定，減少 Disk I/O

以下會分別針對這些優化項目進行說明。

### Indexing 大量資料時，善用 bulk request

大量資料要 Indexing 時，使用 bulk 減少 round-trip overhead，至於 bulk request 要多大才是合適的？這個在不同的 Elasticsearch Cluster 硬體規格、不同的 Indexing 文件大小，所以還是要依照使用情境進行 Benchmark ，找到最合適的批次處理大小，另外官方有建議，bulk request 的資料量太大的話，大量的 bulk 請求同時進入 Elasticsearch 時，可能會吃光 Elasticsearch 的記憶體，所以一般不建議一個 bulk request 處理數十 MB 以上的資料。

### 使用 multi-thread / multi-workers 來 indexing 資料進入 Elasticsearch

使用 multi tread/worker 來處理 indexing 絕對比單一 thraed 能提高處理的效率，不過就像 bulk request 的調效一樣，這個會是依照硬體配置有不同的最佳化配置方式，所以同樣的會建議進行 Benchmark 來保確當下情境下合適的 thread 數量，並注意若是 Elasticsearch 丟出 `TOO_MANY_REQUESTS (429)` 的錯誤時，就已達到上線，應該要調整配置。

### 調低或暫時關閉 `refresh_interval`

在 Elasticsearch 的 Indexing 生命週期中，當不斷的有資料 indexing 進入 Elasticsearch 時，一開始是寫在 memory buffer 中的，而這時還無法被搜尋到 (如下圖)：

![image-20201012074401270](https://i.imgur.com/4ROgeQF.png)

當 Elasticsearch 透過 `refresh` 的機制，將 In-memory buffer 的整理成 segment file，這時的狀態就是能被搜尋到的 (下圖中灰色的那塊，因為還沒進行 `fsync` 所以還沒被 commit)。

![image-20201012074416347](https://i.imgur.com/Q4ZNovs.png)

Elasticsearch 預設的 `refresh_interval` 是 `1s` ，也就是一秒會進行一次處理，若是大量的資料在進行 indexing 時，可以將這個值調大，或甚至先暫時關閉，以提升 indexing 的效率。

> 這個配置的調整，有時能將 indexing 的效能提升一倍，不過一樣是依照情境會有不一樣的效果，但基本上都能有一定的成效。

這個設定的配置在 `index.refresh_interval` ，可參考 [官方文件 - Index Module](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-refresh-interval-setting)。

### 指定 Routing 的方式，減少 Thread 的數量

當我們在 indexing 資料進入 Elasticsearch 時，Elasticsearch 的 Routing 機制預設會讓資料平均分配在各個 Shard 身上，以下圖為例，可以想像左邊是我們的 Coordinator node，當我們有兩批資料要 bulk indexing 進入 ES，而在沒有特別指定 Routing 的機制，所以每個資料被分配到不同的 shard 身上，這個例子我們有 4 個 Shards，所以每一批的處理，都會要分配到 4 個 Shards，這時 bulk 的處理就會要 4 個 Threads 來處理各個 Shard 的工作，2 批 bulk request 也就是總共有 8 個 Threads。

![img](https://i.imgur.com/MneJkIR.png)

如果我們指定第一批的資料只會 Routing 到 Shard 1, 2，而第二批的資料只會 Routing 到 3, 4，這樣 bullk 在處理時，就只會用到 2 個 Threads來進行這樣的操作。(如下圖)

![img](https://i.imgur.com/saAjVqk.png)

可以看到這種例子時，Threads 數量就減少一半了，這部份的配置方式，請參考 [官方文件 - Routing](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-routing-field.html)。

### 第一批資料 indexing 進入 Elasticsearch 之前，先不要設定 Replica

建議如果是一次性的 Indexing 一批資料進入 Elasticsearch，先將 `index.number_of_replicas` 設成 `0`，在我們 Indexing 資料時， 讓 Elasticseach 把資源用在處理 Indexing ，而不要在這個階段就去進行 Replica 的處理，等到資料都 indexing 完成之後，再把這個配置改回我們原先的預期配置，再讓 Elasticseach 進行 replication 的處理。

### 關閉 java process swapping

為了避免 JVM heap 被 swap 到 disk，而降低 Elasticsearch 的處理效率，這邊建議關閉 swapping，這部份請直接參考 [官方文件 - Disable swapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html)。

### 確保 Filesystem 有足夠的 memory cache

Elasticsearch 使用時，由於使用 Lucene 進行許多 Segment files 的處理，會需要用到大量 filesystem 的 memory buffer，因此官方的配置建議上，會建議 JVM Heap size v.s OS filesystem 各配置 50% 的記憶體大小，因此請確保 Filesystem 擁有足夠的記憶體來處理 indexing 的 request。

### 使用 auto-generated ids

Elasticsearch 在進行 Indexing 的處理時，會檢查文件的 id 是否已存在於目前的 shard 當中，如果是使用 Elasticsearch auto-generated ids 時，由於這個 Id 的組成有包含的時間，所以 Elasticsearch 可以確保他產生時不會與現存的資料有重覆的情況，因此可以省略這個 id checking 的機制，這樣會讓 indexing 的速度有所提升。

### 使用更快速的儲存硬體

Indexing 的處理是屬於 I/O bound，在官方的建議配置上，會建議基本上要使用 SSD 等級的硬碟來當成 Elasticsearch 的儲存硬體規格，而且使用 SSD 的配置，會讓整體的 C/P 值會較高。

若是因資料量太大而有成本的考量，應該進一步再使用 Index Lifecycle Management 將 Indexing 完成的資料、或是較舊的資料，轉移到較便宜的磁碟硬體狀置上。

### 調高 indexing buffer 大小

如果有大量的 indexing 的處理時，適時的調高 [`indices.memory.index_buffer_size`](https://www.elastic.co/guide/en/elasticsearch/reference/current/indexing-buffer.html)，這個值預設是 `10%` 的 JVM head size，這個 index buffer 是所有 active shard 共用的，所以若是有大量 indexing 處理時，也就會互相佔用這個空間，所以確保 indexing 時影響的 shard 數量，並且配置足夠的 index buffer size。

> 這個配置官方的建議是一個 shard 在 **512MB** 之內，若是超過的話效能一般不會有太明顯的改善。

### 使用 cross-cluster replication 的配置讓 searching 的處理不會佔用 indexing 的資源

如果是持續不斷的大量 indexing 資料進入 Elasticsearch 中，可以考慮使用 multi-cluster 的架構，將 indexing 的處理指向一個 Elasticsearch Cluster - A，而透過 cross-cluster replication 將資料 replica 到另一個 Elasticsearch Cluster - B，所有的 search 就指向 Elasticsearch Cluster - B，讓 searching 的各種請求處理，不會佔用到 indexing 的處理資源，確保 indexing 有獨立不受影響的資源配置。

### 調整 Translog 的 Flush 設定，減少 Disk I/O

Elasticsearch 因為避免 process crash 時資料的遺失，會預設在每 `5秒鐘` 、或是 memory size 達到 `512mb` …等條件達到時執行 fsync，而這個處理的 Disk I/O 成本較高，因此在大量 indexing 資料時，而且這時期又允許接受系統 crash 時資料遺失的風險，可以將 `index.translog.interval` 和 `index.translog.flush_threshold_size` 的配置調高，以提升 indexing 的效率。



## 參考資料

- [官方文件 - Tune for indexing speed](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html)

- [Scaling Elasticsearch Part 1: How to Speed Up Indexing](https://dev.to/molly_struve/scaling-elasticsearch-part-1-how-to-speed-up-indexing-2pel)