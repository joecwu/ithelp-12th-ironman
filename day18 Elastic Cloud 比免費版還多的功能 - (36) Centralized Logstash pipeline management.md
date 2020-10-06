# 喬叔教 Elastic - 18 - Elastic Cloud 比免費版還多的功能 - (3/6) Centralized Pipeline Management

**Elastic Cloud 比免費版還多的功能** 系列文章索引

- [(1/6) Elastic Stack 的方案比較與銷售方式](https://ithelp.ithome.com.tw/articles/10247538)

- [(2/6) Centralized Beats Management](https://ithelp.ithome.com.tw/articles/10248028)

- [(3/6) Centralized Pipeline Management](https://ithelp.ithome.com.tw/articles/10248584)

- [(4/6) Watcher](https://ithelp.ithome.com.tw/articles/10248884)

- [(5/6) Elasticsearch Token Service](https://ithelp.ithome.com.tw/articles/10249181)

- [(6/6) Multi-stack monitoring & Automatic stack issue alerts](https://ithelp.ithome.com.tw/articles/10249630)

---

## 前言

Logstash 是一個非常強大的資料 ETL (Extract-Transform-Load) 工具，在具有一定量級的處理環境時，常常會佈署多台的 Logstash，以及配合許多情境而有對應配置的 ETL 設定，這些對於管理及運維上都有一定的複雜度及成本，這篇文章將會介紹，如何透過 X-Pack 中 **Gold License** 以上的授權、或是 Elastic Cloud service **Standard** 以上版本才能使用的 Centralized Pipeline Management ，如個使用這個功能讓 Logstash Pipeline 的管理能集中化的在 Kibana 中被簡單的配置。

### 進入此章節的先備知識

- Logstash 的基本配置與使用方式。
- Logstash 的 Pipeline 配置方式。

### 此章節的重點學習

- 如何透過 Centralized Pipeline Management 來集中化管理 Logstash 的 Pipeline 配置。

---

## Centralized Pipeline Management 基本介紹

Centralized Pipeline Management 是 Elastic Stack 6.0 時以 Beta 版本首次推出的服務，主要的目的就是使用 Kibana 來當作集中化的管理工具、能管理多台 Logstash 身上的 Pipbline 配置，同時將 Elasticsearch 當作 Database ，存放這些 Pipeline 的配置。

### 功能定位

Centralized Pipeline Management 是為集中化的管理多台 Logstash，如果已經有使用其他更強大的 Configuration Management (CM) 工具像是 Puppet, Ansible, Chef 來解決這個問題，這功能可能就不見得需要使用，但如果還沒有使用到這些複雜的 CM 工具、但又有這個 Logstash 管理的需求時、又或者希望能由 Kibana 來提供需要修改 Pipeline 的操作者有統一的 GUI 管理工具，那這個 Centralized Pipeline Management 就會是你的選擇。

### Centralized Pipeline Management 基本運作架構圖

![centralized pipeline management](https://i.imgur.com/fQcvHcD.png)



## 將 Logstash 加入 Centralized Pipeline Management

首先我們要將 Logstash 註冊進入 Centralized Pipeline Management 的託管，我們會需要做幾件事：

#### 1. 先配置好 Logstash 所要使用的 Security User & Role

首先依照官方的建議配置，先建立兩個 Role - `logstash_writer` 和 `logstash_reader` 。

```
POST _xpack/security/role/logstash_writer
{
  "cluster": ["manage_index_templates", "monitor", "manage_ilm"], 
  "indices": [
    {
      "names": [ "logstash-*" ], 
      "privileges": ["write","create","delete","create_index","manage","manage_ilm"]  
    }
  ]
}
```

```
POST _xpack/security/role/logstash_reader
{
  "indices": [
    {
      "names": [ "logstash-*" ], 
      "privileges": ["read","view_index_metadata"]
    }
  ]
}
```

> `logstash_reader` 是給之後要能查閱我們透過 Logstash 傳進來的資料而先定義好的 Role。

建立一個 User ，並且給予 `logstash_admin`, `logstash_system` (這兩個是系統內建的 Role ) 和 `logstash_writer` 的 Role。

```
POST _xpack/security/user/logstash_user
{
  "password" : "t0p.s3cr3t",
  "roles" : ["logstash_admin","logstash_system","logstash_writer"],
  "full_name" : "Logstash User"
}
```

> 以上的操作都能直接在 Kibana > Stack Management > Security 中，使用 GUI 的畫面建立。



#### 2. 決定 Pipeline Id 及修改 Logstash 的設定

我們要在某一台已安裝好 Logstash 的機器上，將配置改成 `xpack.management.enable: true` 也就是將這台 Logstash 註冊透過 Centralized PIpeline Management 來託管。 

```
# config/logstash.yml

xpack.management.enabled: true
xpack.management.logstash.poll_interval: 5s
xpack.management.pipeline.id: ["main", "apache_logs", "cloudwatch_logs"]

# 若是自架 Elastic Stack 要設置以下 Elasticsearch 的配置
xpack.management.elasticsearch.hosts: "http://localhost:9200/"
xpack.management.elasticsearch.username: logstash_user
xpack.management.elasticsearch.password: t0p.s3cr3t

# 若是使用 Elastic Cloud 可以直接指定 cloud_id 和 cloud_auth
xpack.management.elasticsearch.cloud_id: uncle-joe:xxxxxxxxxxxx
xpack.management.elasticsearch.cloud_auth: logstash_user:t0p.s3cr3t
```

> 這邊要注意，在 Config 中指派的 pipeline id，也就會是這個 logstash 機器會從 Centralized Pipeline Management 抓取的 Pipeline 配置，所以這部份是要事先定義好的。

另外以下是 X-pack Monitoring 的設定，也建議打開，或是使用  Metricbeat 來另外收集 Logstash Monitoring 的資訊。

```
xpack.monitoring.enabled: true
xpack.monitoring.collection.interval: 10s
xpack.monitoring.elasticsearch.cloud_id: uncle-joe:xxxxxxxxxxxx
xpack.monitoring.elasticsearch.cloud_auth: logstash_system:t0p.s3cr3t
```



#### 3. 註冊 Logstash 進入 Centralized Pipeline Management 中

當配置完成後，不需要做其他特別的執行，只要啟動 (或重新啟動) Logstash

```
bin/logstash
```



![run logstash](https://i.imgur.com/oBvAvka.png)

這時會發現出現找不到 remote config 的錯誤 (因為 Kibana 上的 Pipeline config 我們這時還沒建立好)。



#### 4. 確認 Kibana 使用者的權限

這邊要特別注意，當我們配置好之後，要重新回到 Kibana 來設定 Logstash Pipelines 時，要確保 Kibana 的使用者有以下的權限：

- logstash_admin
- logstash_writer



一但以上的步驟完成後，我們接下來就要進入 Kibana 設置 Logstash Pipeline。

## 使用 Kibana 管理 Logstash Pipelines

進入 **Kibana** > **Stack Management** > **Ingest** 中，有一個 **Logstash Pipeline** 的管理介面。

![Screen Shot 2020-10-03 at 10.22.52 PM](https://i.imgur.com/G5o3agQ.png)



點選 **Create Pipeline** 之後，就可以依照一般使用 Pipeline 的配置方式來設定 Pipeline。

![create pipeline](https://i.imgur.com/avCu4u0.png)

最後按下 **Create and deploy** 就會立刻發佈到 Logstash 並生效。

## 透過 Kibana Stack Monitoring 確認 Logstash Pipelines 運作狀況

從 **Kibana** > **Stack Monitoring** > **Logstash** 中，可以看到目前 Logstash 的機器狀況，也可以即時監看 Pipeline 的運作狀態。

![stack monitoring](https://i.imgur.com/HnJj6iz.png)

![stack monitoring pipelines](https://i.imgur.com/WrLJojT.png)

若是有多台機器、多組 Pipeline 的配置，都能透過 Stack Monitoring 即時監看運作狀態，在更新 Pipeline 的配置後，請記得來這邊確認配置的結果是否運作正常。



## 使用建議與注意事項

- Centralized Pipeline Management 的目的是讓 Logstash Pipeline 可以讓多租戶的使用者們自行配置、自主管理，而這邊會配合的用方是 Logstash multiple pipeline 的配置方式，也就可以讓我們 **以 Pipieline 為單位** 來定義各種不同情境的需求，也就是每個租戶、或是每個 data-flow 可以配置屬於自己的 Pipeline。
- Centralized Pipeline Management 所建立的 Pipeline 資料是真接儲存在 Elasticsearch 中，所有的調整與變動都會直接套用到 Logstash，不需要重新開啟動 Logstash，而這些 Pipeline 的配置沒有任何的驗證錯誤的機制，也就是一但修改後就會直接生效，若是配置有錯誤的話，也就會直接反應到實際的環境中，這部份會需要特別小心。
- Logstash 在這樣集中化管理的情況，一但發生錯誤，也會需要看每台 Logstash 的 logs 才能知道狀況，因此也同樣會建議啟用 X-Pack monitoring 的功能，讓 Logstash 的 logs 被收集到 Elasticsearch 來一併監控與管理。
- 為了能控制發生錯誤時的影響，建議配置好 Logstash 的 [Dead Letter Queue](https://www.elastic.co/guide/en/logstash/current/dead-letter-queues.html) ，當有錯誤發生時，也能再次透過 Pipeline 的調整，以重新修復資料。



## 參考資料

- [官方文件 - Centralized Pipeline Management](https://www.elastic.co/guide/en/logstash/current/logstash-centralized-pipeline-management.html)
- [官方文件 - Configuring Centralized Pipeline Management](https://www.elastic.co/guide/en/logstash/current/configuring-centralized-pipelines.html)
- [官方文件 - Dead Letter Queue](https://www.elastic.co/guide/en/logstash/current/dead-letter-queues.html)
- [官方 Blog - Logstash Centralized Pipeline Management](https://www.elastic.co/blog/logstash-centralized-pipeline-management)

