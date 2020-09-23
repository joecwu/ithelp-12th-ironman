# 喬叔教 Elastic - 03 - Elastic Cloud 如何建立 Deployment (2/2) - 配置的選擇

### 此章節的重點學習

- 了解 Elastic Cloud 的預設置

---

前一篇我們介紹了 Elasticsearch (ES) Node 的種類，接下來我們回到 Elastic Cloud (EC) 的設置介面。

## Elastic Cloud Deployment 的配置方案

![ECS Deployement Options](https://i.imgur.com/DElQFOg.png)

Elastic Cloud 已經提出了幾個情境的 deployment template 配置方案 ，細節也都有在 [官方文件](https://www.elastic.co/guide/en/cloud/current/ec-getting-started-templates.html) 中解釋，這邊每個 `Default specs` 裡面都有機器規格的列表。

![EC deployment specs](https://i.imgur.com/4VuRPwj.png)

中間有幾個關鍵字，例如 `coordinating`, `data`, `master`, `ml(machine learning)` 這些都是 Elastic 針對這種使用情境的特性，建議可以安排這些功能各自的 Nodes，而我這邊的例子是選擇 AWS 來當我的 Cloud Provider，所以後面的 `m5d`, `i3`, `r5d` 這些都可以對照下表，知道這些機器的安排規格。

### Deployment 的硬體規格參考(以 AWS 為例)

![EC elasticsearch AWS hardward](https://i.imgur.com/WoH45sD.png)



## 進入 Customize Deployment

我們再進一步選擇畫面底下的 `Customize Deployment`，可以針對下列各項主要的功能獨立設置硬體的配置：

- Data: 預設 Elasticsearch 存資料的 nodes
  - `highio`預設配置是: `master-eligiable`, `data`, `coordinating`, `ingest`
  - `highstorage`預設配置是: `data`, `coordinating`, `ingest`
- Machine Learning: 單純執行 Machine Learning 任務的 node。
- Coordinating: 沒有任何的角色，是之前提到的 `dedicated coordinating node`。
- Master: 是 `dedicated master node`。
- Kibana: 運行 Kibana Web Application 的機器。
- APM: 運行 APM 服務的機器。
- Enterprise Search: 是 App Search 和 Workspace Search 的應用程式安裝機。

如下圖紅框，可查看每個配置的角色為何，可針對此角色的 Node 來設定預期的硬體資源配置。

![EC customize deployment](https://i.imgur.com/PjSvD7h.png)



## 安裝 Plugins 及設置 Extensions

Elastic Cloud 預設將常用的一些 plugins 在設定畫面中可直接勾選，若你有要處理 CJK (Chinese, Japanese, Korean) 的語言，通常你至少會要選擇 `analysis-icu` 或甚至是其他的 analysis plugins 。

![EC plugins](https://i.imgur.com/Oxqej4D.png)

若有使用同義字 (Synonym) ，要掛載字典檔的話，要從目錄選單左側的 `Extensions` 進入。

![EC Extensions](https://i.imgur.com/1bOzQFl.png)



## 透過 Create Deployment API 建立 Deployment

當所有設置都決定好之後，我們按下 `Create deployment` 即可進入產生 Deployment 的處理。

![Create Deployment](https://i.imgur.com/E8A3xjG.png)

而其實這個 Elastic Cloud 的網頁設置畫面，只是協助產生 Create Deployment 的 API request，我們從底下的 `Equivalent API request` 可以直接看到透過 Elastic Cloud UI 所建立出來的配置，產生的 API request 細節。

![EC Create deployment API request](https://i.imgur.com/fB0Lf6M.png)

因此若是要特別客製某個 Node 的配置與角色，也可參考官方的 [Deployment CRUD API request](https://www.elastic.co/guide/en/cloud/current/Deployment_-_CRUD.html)，來客製自己的 deployment。



## 完成設置

按下 `Create Deployment` 後，會跳出 Elastic Cloud 在 xpack security 所設置的管理者帳號及密碼，這組帳號將是登入 `Kibana`, `APM`, `Enterprise Search`…等服務時需使用，最後需要約數分鐘的時間等待 deployment 的建置完畢。

![ECS create deploymenet done](https://i.imgur.com/dHBnUIB.png)

## 參考資料

- [官方文件 - EC deployement templates](https://www.elastic.co/guide/en/cloud/current/ec-getting-started-templates.html)
- [官方文件 - Elasticsearch service hardward](https://www.elastic.co/guide/en/cloud/current/ec-reference-hardware.html)
- [官方文件 - EC deployment CRUD](https://www.elastic.co/guide/en/cloud/current/Deployment_-_CRUD.html)

