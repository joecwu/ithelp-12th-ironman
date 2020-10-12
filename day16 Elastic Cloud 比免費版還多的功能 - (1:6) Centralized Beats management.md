# 喬叔教 Elastic - 16 - Elastic Cloud 比免費版還多的功能 (1/6) - Elastic Stack 的方案比較與銷售方式

**Elastic Cloud 比免費版還多的功能** 系列文章索引

- [(1/6) Elastic Stack 的方案比較與銷售方式](https://ithelp.ithome.com.tw/articles/10247538)

- [(2/6) Centralized Beats Management](https://ithelp.ithome.com.tw/articles/10248028)

- [(3/6) Centralized Pipeline Management](https://ithelp.ithome.com.tw/articles/10248584)

- [(4/6) Watcher](https://ithelp.ithome.com.tw/articles/10248884)

- [(5/6) Elasticsearch Token Service](https://ithelp.ithome.com.tw/articles/10249181)

- [(6/6) Multi-stack monitoring & Automatic stack issue alerts](https://ithelp.ithome.com.tw/articles/10249630)

---

## 前言

在使用 Elastic Stack 時，總共有哪些方案，而針對 Elastic Cloud 的使用，我們所選擇不同的方案能使用的功能又有哪些差異，這篇文章會先就基本面來做個比較與介紹。

### 進入此章節的先備知識

- Elastic Stack 整體的基本認識、能大約看懂 Elastic Stack 功能列表中專有名詞指的是什麼。

### 此章節的重點學習

- Elastic Stack 的 賣法。
- Elastic Pricing 與 Subscription 的比較。
- 使用 Elastic 官方所提供 SaaS Elastic Cloud 最便宜的 Standard 版本，比自己架設的免費 Basic 版本有多出哪些功能。

---

## Elastic Stack 的安裝方式

首先，我們先針對接下來會出現的名詞定義有個解釋：

- **Elastic Stack**：由 Elastic 官方提供使用者即時、迅速、可造的資料分析解決方案的統稱，包含了 Elasticsearch, Kibana, Beats, Logstash…等產品。
- **Elastic Cloud**：Elastic Stack 產品的佈署、配置、管理的一套管理機制，可以自行佈署在 public cloud 或是 private cloud，甚至是直接由 Elastic 官方佈署好，並以 SaaS 的形式來提供使用。



所以，當我們要安裝 Elastic Stack 時，有以下三種方式：

- 使用官方的 SaaS 服務 - **Managed Elastic Cloud**：直接打開 [Elastic Cloud](http://cloud.elastic.co/) 的網頁，信用卡填進去就可以在網頁上開始操作，並產生出佈建在 AWS, GCP 或 Azure 的 Elastic Stack Deployement 了。
- 自己架設 Elastic Cloud - **Self-managed Elastic Cloud**：想要使用像是 Elastic Cloud 這麼方便的管理工具來建置 Elastic Stack，但是又想要有更多的自主控制權、想要佈建在自己的機房、或是使用自己管理的 Cloud Provider、甚至想要有更進階的 Deployement 配置方式，就可以使用 Elastic Cloud Enterprise (ECE) 或是 Elastic Cloud on Kubernetes (ECK) 的方式來架設。
- 什麼都自己來 - **Self-managed Elastic Stack**：可能由手動架設、或是自己用任何容器化的佈署工具、Configuration Management…，總之不使用 Elastic Cloud 佈署機制的方式都算是這類。



## Elastic Stack 的賣法

再來，我們進入官方網站的 Pricing 頁面：

![elastic website - pricing](https://i.imgur.com/3PbRTMD.png)

這邊可以看到有三大類型，也就是對照到上面我們介紹到三種安裝方式。

我們把這三種方式的賣網的網址列出來如下：

- Managed Elastic Cloud: https://www.elastic.co/pricing/
- Self-managed Elastic Cloud: https://www.elastic.co/subscriptions/enterprise
- Self-managed Elastic Stack: https://www.elastic.co/subscriptions

聰明的大家可以看到兩個關鍵字 `Pricing` 和 `Subscriptions` ，這兩個的差別就是：

- **Pricing**： 使用 SaaS 服務時，照用量來收費的價格：
- **Subscription**：使用自己架設 (self-managed) 的方式，但又想使用到 Basic (免費版) 以上的版本時，要與 Elastic 官方購買訂閱制的 License，並將這個 License 登錄到 Elastic Stack 中，來啟用這些進階的功能。

接下來我們針對各種方案進行簡單介紹。

### Managed Elastic Cloud

![elastic cloud pricing](https://i.imgur.com/r1I4YOO.png)

若是使用 Elastic 官方提供的 SaaS 服務，使用的價格可以直接透過 [Elastic Cloud 價格計算機](https://cloud.elastic.co/pricing) 來評估計算。

總共有四個方案，從網頁的下方有完整的列出每個方案的功能列表與比較表。

![elastic cloud plan comparsion](https://i.imgur.com/3kKXSFk.png)

使用這方案時，主要就是依照你要的功能來決定你要買的是哪個版本，並且網頁上可以使用 **月結** 的方式來支付，若是要長時間使用建議與官方銷售人員聯繫，可以使用 **年結** 的方式來付款，並且可以有一些優惠折扣。

### Self-managed Elastic Cloud

再來的方案是，自己架設 Elastic Cloud，如前面提到的，這方案有分兩種方式：

#### Elastic Cloud Enterprise (ECE)

這部份基本上就是要錢的，而且授權是使用 `Platinum` 等級，費用的部份就要與官方的銷售人員聯繫。

> 之前詢價時得到的起跳價是 54,000 鎂/年，不同時間點與條件詢到的價格可能有不同，因此僅供參考。

![ece](https://i.imgur.com/SZmpNxF.png)

#### Elastic Cloud on Kubernetes (ECK)

另一個方式就是使用 Kubernetes 來架設，這裡有個 Basic 的免費版，大家可以直接來使用，若是要 Enterprise 的版本，同樣的也是要聯繫銷售窗口取得報價。

![eck](https://i.imgur.com/dgUTuCa.png)

這邊提到的 Basic 或是 Platinum 的版本，都是指 Subscription 的 License ，所以細節的功能比較，都會和下面的 Self-managed Elastic Stack 一樣。

### Self-managed Elastic Stack

這方案是自己架設了，所以要來看的只有他的各種 Subscription License 版本的差異。

![elastic stack subscriptions](https://i.imgur.com/wjTN2NC.png)

這邊可以看到 **免費版** 有包含 **Open Source** 和 **Basic** 兩種版本，若是要經過再開發、把這樣加上自己開發項目的進階方案再另外拿來賺錢銷售的話，可以使用 Open Source 版，但不能直接使用免費的 Basic 版哦！這邊要特別注意版權的細節，詳細可與官方銷售聯繫及確認授權的問題。

也因此，當我們使用到像是其他提供商提供的 Elasticsearch 服務，例如 AWS Elasticsearch Service，就必然是使用 Open Source 的版本來開發，裡面自然也少了許多 Elastic 官方開發的好用功能。

各版本的功能差異，也可以從網頁下方的比較表查看：

![elastic stack license comprison](https://i.imgur.com/y70HFwM.png)



## Elastic Cloud Standard 版本，比自己架設的 Basic 版本有多出哪些功能

這系列的文章，主要會針對官方提供的 SaaS - Elastic Cloud Standard 的版本 (也就是最便宜的版本)，和自己架設的 Basic 版本 (也就是不用錢的版本)，來比較使用官方的方案有什麼特別的好處。

> 這次比較的方式，都是以不購買到進階的 Gold, Platinum, Enterprise，而是兩種的最基本的方案來比較。

我這邊不會全部完整的比較，但會挑出幾個我覺得差異較大、也特別實用的、也會是這系列文章主要會介紹的功能來比較。

### Stack Management

下方是 **Self-managed 的 Subscription** 方案：

![stack mgt - subscription](https://i.imgur.com/KiZov5d.png)

而這是 **SaaS 的 Elastic Cloud** 的方案：

![stack mgt - saas](https://i.imgur.com/uNYkzyI.png)

可以看到兩個功能是特別有包含在 SaaS 的 Standard 的方案中

- Centralized Beats management
- Centralized Logstash pipeline management



### Alerting

下方是 **Self-managed 的 Subscription** 方案：

![image-20201001192839025](https://i.imgur.com/XvodLKI.png)

而這是 **SaaS 的 Elastic Cloud** 的方案：

![Screen Shot 2020-10-01 at 7.29.12 PM](https://i.imgur.com/Kgi62gv.png)

可以看到以下的功能是特別有包含在 SaaS 的 Standard 的方案中：

- Watcher



### Elastic Stack Security

下方是 **Self-managed 的 Subscription** 方案：

![image-20201001193130239](https://i.imgur.com/tVuQZpr.png)

而這是 **SaaS 的 Elastic Cloud** 的方案：

![Screen Shot 2020-10-01 at 7.31.52 PM](https://i.imgur.com/UJmLbQD.png)

可以看到以下的功能是特別有包含在 SaaS 的 Standard 的方案中：

- Elasticsearch Token Service



### Stack Monitoring

下方是 **Self-managed 的 Subscription** 方案：

![image-20201001193328221](https://i.imgur.com/Xf8tnbu.png)

而這是 **SaaS 的 Elastic Cloud** 的方案：

![Screen Shot 2020-10-01 at 7.33.50 PM](https://i.imgur.com/ywOCCt2.png)

可以看到以下的功能是特別有包含在 SaaS 的 Standard 的方案中：

- Multi-stack monitoring
- Automatic stack issue alerts

### 差異比較總結

這邊列出來的差異項目，將會是這系列文章中接下介紹的項目，總結如下：

- Centralized Beats management
- Centralized Logstash pipeline management
- Watcher
- Elasticsearch Token Service
- Multi-stack monitoring
- Automatic stack issue alerts



## 參考資料

- [官方網站 - Pricing](https://www.elastic.co/pricing/)