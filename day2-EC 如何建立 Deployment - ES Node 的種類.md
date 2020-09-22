# 喬叔教 Elastic - 02 - EC 如何建立 Deployment - ES Node 的種類

### 此章節的重點學習

- 了解 Elasticsearch Node 可扮演的角色有哪些、以及我們常提到的 Node 的種類有哪些，其中扮演的是哪些角色。

---

## 建立 Elastic Cloud (EC) Deployment 時的選擇

![create deployment](https://i.imgur.com/eNDS0vP.png)

當我們在 Elastic Cloud (EC) 上建立一個 Deployment 時，我們會面對下列幾個選擇：

- Deployment 的名稱。
- 指定的 Cloud Platform。
- 這些 Elastic Stack 佈置在的 Region。
- Elastic Stack 的版本。
- 是否要把這個 Deployment 的 Monitor 資料傳到另個統一收集 Monitoring 資料的 Deployment 來分開存放。
- 是否要限制能存取這個 Deployment 的 Traffic Filter 設置：例如只能有特定的 IP 存取、或是只能有特定的 AWS VPC Enpoint 來存取。
-  `Optimize your deployment`也就是你的 Deployment infra 上的配置方式。

最後一項 `Optimize your deployment`，一般會是大家思考最久的問題，而我們這篇會來先解釋這個選擇要考慮的項目。

![setup deployment](https://i.imgur.com/So4dWxB.png)

## Elasticsearch Node 可扮演的角色

在進入說明之前，大家需要先知道 Elasticsearch Node 可扮演的角色有以下這些：

- **Master-eligible role:** 是否可以被選擇當作 Cluster 的 Master 節點，以負責以下幾項任務：
  - 維護 Cluster 的狀態，例如 Node 的加入或移除。
  - 負責處理建立或刪除 Index 的請求，並指派給相關的 Node 進行實際操作的執行。
  - 決定 shard 要被分配到哪個 node 身上。
- **Data role:** 是否負責儲存資料，以及執行資料相關的操作 CRUD 或是執行 search、aggregation。
- **Ingest role:** 是否負責執行 [Ingest pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/current/pipeline.html) 所定義的工作 (Ingest pipeline 是在資料匯入至 Elasticsearch 之前，可在進行 Index 前進行資料修改、轉換…等 ETL (Extract, Transform, Load) 操作，也就是提供了部份 Logstash 的功能)。
- **Machine learning role:** 是否負責處理 Machine learning 的工作，這項功能是包含在 xpack 中。通常這個 role 會安排獨立的 Node 來扮演。
- **Transform role:** 是否執行 [Transform](https://www.elastic.co/guide/en/elasticsearch/reference/current/transforms.html) 的任務，這項功能是包含在 xpack 中。
- **Remote cluster client:** 是否能負責存取 Remote Cluster。

## Elasticsearch Node 的種類

通常一個 Node 我們可以同時扮演多種角色，以下的 Node 種類，是依照功能特性與使用情境來特別分類說明：

- **Default node:** 預設開啟一個 Node 的時候，可扮演的角色是 `Master-eligible`, `Data`, `Ingest`, `Remote cluster client node`, `Transform`。
- **Dedicated master-eligible node:** 如果 Cluster 規模較大、或是非常重視 Cluster 的穩定性，可安排某個 Node 只擔任 `Master-eligible` 的角色，這個 Node 就不會因為大量的 indexing, searching, ingesting, transforming 等操作讓資源被吃光而導致 Cluster 不穩定。
- **Voting only node:** 在 [Master Node 的選舉 ](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/modules-discovery-quorums.html)時，有時可能我們的 Master-eligible node 是雙數，可能會造成無法順利完成選舉，因此可以安排只負責投票，但不參與選擇的 `Voting only node`。(在這種模式下，Node 本身也會是 `master-eligible`的角色，因為必須是 `master-eligitble`才能參與選舉，而 `Voting only node`會宣告本身不要被選上)
- **Dedicated Data node:** 只單純負責儲存資料，專心的負責 CRUD, search, aggregation 這些很需要 CPU, Memory, I/O 資源的任務，不會讓其他的任務佔用資源。
- **Dedicated coordinating node (Client node):** 不扮演任何的角色，只單純的接受 indexing request 或是 searching request。當有複雜的 sorting、多層的 aggregation 查詢、大量的 bulk request 時，接收到 request 的這個稱之為 Coordinator 的 Node，會需要大量的記憶體來處理這些request，這時可評估安排 Dedicated coordinating node ，並將這類 request 的流量導入到這個 node 身上來進行處理，可以避免因過度消耗資源而影響到 cluster 的穩定性或是其他操作的性能。
- **Dedicated ingest node:** 通常有大量的 ETL 處理的情境下，可以特定安排某個 Node 單純的處理 ingest 的任務，而不讓 ETL 的處理佔用到其他角色的工作執行資源。

下一篇，我們將繼續這個主題，說明 EC 上的預設配置，以及我們如何做選擇。



## 參考資料

- [Elastic 官方文件 - Node 的說明](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)
- [Elastic 官方文件 - Ingest pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/current/pipeline.html)
- [Elastic 官方文件 - Master Node 的選舉 ](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/modules-discovery-quorums.html)

