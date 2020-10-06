# 喬叔教 Elastic - 21 - Elastic Cloud 比免費版還多的功能 - (6/6) Multi-stack monitoring & Automatic stack issue alerts

**Elastic Cloud 比免費版還多的功能** 系列文章索引

- [(1/6) Elastic Stack 的方案比較與銷售方式](https://ithelp.ithome.com.tw/articles/10247538)

- [(2/6) Centralized Beats Management](https://ithelp.ithome.com.tw/articles/10248028)

- [(3/6) Centralized Pipeline Management](https://ithelp.ithome.com.tw/articles/10248584)

- [(4/6) Watcher](https://ithelp.ithome.com.tw/articles/10248884)

- [(5/6) Elasticsearch Token Service](https://ithelp.ithome.com.tw/articles/10249181)

- [(6/6) Multi-stack monitoring & Automatic stack issue alerts](https://ithelp.ithome.com.tw/articles/10249630)

---

## 前言

這篇是 **Elastic Cloud 比免費版還多的功能** 系列的最後一篇，主要是針對 **Stack Monitoring** 中 **Gold License** 才能使用、但已經在 Elastic Cloud **Standard** 版本中開放使用的兩個功能 `Multi-stack monitoring` 和 `Automatic stack issue alerts` 。

### 進入此章節的先備知識

- Elasticsearch 的基本認識
- Kibana 的基本操作

### 此章節的重點學習

- Multi-stack monitoring 的功能介紹。
- Automatic stack issue alerts 的功能介紹。

---

## Multi-stack monitoring

這個功能主要的目的是讓你透過一個 **集中化的監控叢集 (centrlized monitoring cluster)** ，來記錄、追縱、比對來自多個 Elastic Stack deployment 的健康或是效能的狀態，白話就是：把多個 cluster + elastic stack 產品的 log 送到專們 monitoring 用的 cluster 來監管。

> 這樣 Centralized monitoring cluster 也是 Elastic 官方建議的做法，做好角色分工，也避免讓處理其他工作的 Elasticsearch Cluster 若發生異常時，不會影響到 Monitoring 的監控狀態。

以下是在 Elastic Cloud 的 Elasticsearch 設定時，會看到 Monitoring deployment 的設定選項，其中也明確的建議 Production 環境要記得將 Montoring 的任務安排給一組特定的 Deployment 來管理。

![elastic cloud monitoring deployment](https://i.imgur.com/Q7xuQ6n.png)

若是 Monitoring Culster 同時管理多個 Elasticsearch Culster 時，進入 **Kibana** > **Stack Monitoring** 會出現 Clusters 的選單，此時也有所有 Clusters 的 Overview。

![stack monitoring - multi culster](https://i.imgur.com/NY5SXpz.png)

進入某個 Cluster 後，可以查看這組 Deployement 的 Elastic Stack monitor。

![stack monitoring - specific cluster](https://i.imgur.com/hO3c9xO.png)

> Elastic Stack 中，Metricbeat 是被 Elastic 官方推薦用來收集 Monitoring data 的工具，詳細如何使用，請參考 [官方文件 - Collecting monitoring data with metricbeat](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-metricbeat.html) 。



## Automatic stack issue alerts

Elastic Stack 中除了先前介紹的 Watcher 之外，Elastic Stack 7.7 時推出全新的 Alerting Framework，將 6.x 版時陸續為了先舖路而推出的各個功能整合起來，完成 Alerting 的大業。詳細的介紹可以從 [官方 Blog - Introducing the new alerting framework for Elastic Observability, Elastic Security, and the Elastic Stack](https://www.elastic.co/blog/introducing-the-new-alerting-framework-for-observability-security-and-the-elastic-stack) 來查閱。

> Watcher 主要是以 Elasticsearch 來當成執行的 Instance，而 Alerting 是以 Kibana 來當執行的 Instance，兩者層級不太一樣，詳細差別官方文件有特別一個章節在描述他們的差異，可以參考 [這邊](https://www.elastic.co/guide/en/kibana/current/alerting-getting-started.html#alerting-concepts-differences)。

在 Elastic Stack 中，這個 Automatic stack issue alerts 有預先針對 Elastic Stack 定義好一些通知的 alerts 機制，我們進入 setup mode 去修改相關的設定。

![stack monitoring edit mode](https://i.imgur.com/QNtlstv.png)

在不同的 Elastic Stack 中，點選 `alerts` 之後，可看到會有不同的 Alerts 的定義。

![Screen Shot 2020-10-05 at 11.55.59 PM](https://i.imgur.com/dMfOTNz.png)

每個 Alerts 都可以進入編輯，並修改一些觸發條件、或是執行的動作。

![image-20201005235701713](https://i.imgur.com/WMwTTBg.png)

除此之外，Alerting Framework 目前強調的是可以在 Kibana 的任何地方都能輕鬆的建立合適的 Alert，也包含在 Machine Learning 時也能用同樣的機制來設定 alert 的 actions，這個 Alerting Framework 還在持續的發展中，期待接下來的版本有更多的功能推出，這次 Alerting Framework 不在此章節的介紹範圍內，有興趣的朋友可以參考下方的參考資料查閱更多的細節。



## 參考資料

- [官方文件 - Monitoring in a production environment](https://www.elastic.co/guide/en/elasticsearch/reference/current/monitoring-production.html)
- [官方 Blog - Alerting in the Elastic Stack](https://www.elastic.co/blog/alerting-in-the-elastic-stack)
- [官方 Blog - Introducing the new alerting framework for Elastic Observability, Elastic Security, and the Elastic Stack](https://www.elastic.co/blog/introducing-the-new-alerting-framework-for-observability-security-and-the-elastic-stack)

