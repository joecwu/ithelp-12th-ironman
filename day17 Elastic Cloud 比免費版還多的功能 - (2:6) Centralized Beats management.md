# 喬叔教 Elastic - 17 - Elastic Cloud 比免費版還多的功能 (2/6) - Centralized Beats Management

**Elastic Cloud 比免費版還多的功能** 系列文章索引

- [(1/6) Elastic Stack 的方案比較與銷售方式](https://ithelp.ithome.com.tw/articles/10247538)

- [(2/6) Centralized Beats Management](https://ithelp.ithome.com.tw/articles/10248028)

- [(3/6) Centralized Pipeline Management](https://ithelp.ithome.com.tw/articles/10248584)

- [(4/6) Watcher](https://ithelp.ithome.com.tw/articles/10248884)

- [(5/6) Elasticsearch Token Service](https://ithelp.ithome.com.tw/articles/10249181)

- [(6/6) Multi-stack monitoring & Automatic stack issue alerts](https://ithelp.ithome.com.tw/articles/10249630)

---

## 前言

在使用 Elastic Cloud 的 SaaS 服務時，Centralized Beats Management 這個功能是在 Standard 的版本上即可使用，不像是自己架設的版本，要買到 Gold License 才能使用，這篇文章主要介紹 Centralized Beats Management 的功能如何使用、以及介紹他在協助管理 Beats 上的便利性。

> 請特別注意， Centralized Beats Management 是在 Elastic Stack 6.5 時推出的 `Beta` 版本的功能，目前已在官方文章上明確註明已 `暫停開發` ，達來會有其他更全面的解決方案來取代這個功能，所以使用上請留意。

### 進入此章節的先備知識

- 知道什麼是 Elastic Stack 中的 Beats。
- 使用過 Filebeat 和 Metricbeat。

### 此章節的重點學習

- 如使在 Elastic Cloud 中的 Kibana 使用 Centralized Beats Management。

---

## Centralized Beats Management 的介紹

顧名思意，**Centralized Beats Management** 的目的就是能使用一個集中化的管理介面，讓我們能輕鬆的管理安裝在各機器上的 Beats，特別是改變一些組態設定時，能直接套用到各 Beats 身上。

這個功能是 Elastic Stack 6.5 時推出的 `Beta` 版本功能，並且是在 **Elastic Gold License** 的級別以上、或是 Elastic Cloud service 的 **Standard License** 版本才能使用。

目前有支援的 Beats 只有以下兩種：

- Filebeat
- Metricbeat

以下是 Centralized Beats Management 的架構圖：

![img](https://i.imgur.com/WddMPrD.png)



## 如何在 Kibana 上設定 Beats Central Management

### Enroll Beat

在 Kibana 上要設定 Beats Central Management時，依照下圖的路線進入這個功能，並點選 Enroll Beat。

![beats central management](https://i.imgur.com/VGPz3eI.png)

進入 Enroll Beat 畫面時，會讓你選擇要 Enroll 的是 Filebeat 還是 Metricbeat，並且選擇 Platform 的類型。

選擇完成後，會產生出 enroll 的 script，從這個 script 可以看到主要是將這個 beat 註冊到我們這個 Elastic Cloud 的路徑上，並且包含了 credential 的資訊，讓這個 client 可以透過這 credential 與 Central Management 互相溝通。

![enroll beats - 1](https://i.imgur.com/k00Owqg.png)



接下來到已安裝好 Filebeat 或 Metricbeat 的機器上，執行 `enroll` 的 command。

![enroll clients](https://i.imgur.com/fpgYDH2.png)

執行完成後，當 Filebeat 或 Metricbeat client 向 server 註冊完成後，畫面會自動帶出已註冊的這台 client。

![enroll beat result](https://i.imgur.com/RG7XU2K.png)



接下來可以進行 tag 的設定，這邊的 tag 可以綁定一種組態設定，而之後可以使用 tag 的方式來套用到各機器上，以便快速的套用不同的組態設定到各機器。

![create tag](https://i.imgur.com/NbZ193b.png)



我們可以幫這個 tag 去定義要給他的 configuration ，也就是針對 Filebeat 或是 Metricbeat 的 config 以及 module 的設定。

![create tag - add config](https://i.imgur.com/Y4lz9Iu.png)

這樣就是一個基本 Enroll 的執行過程。

![image-20201002185555879](https://i.imgur.com/jqGakLc.png)



### Enrolled Beats

回到 Enrolled Beats 的畫面後，可以看到每個已註冊進來的 Beats，並且他們各自的 tag 狀態，這時看到的 Config Status 還是 `Offline`

![image-20201002190053520](https://i.imgur.com/1bIjQEb.png)



### Run Enrolled Beats

接著我們到各 Filebeat 或是 Metricbeat 的機器上，讓他們執行起來

```
./filebeat run
```

或是

```
./metricbeat run
```

回到 Enrolled Beats 的 Config Status 來查看，會發現他們都執行起來了。

![running enrolled beats](https://i.imgur.com/NVpZGad.png)

我們這時可以動態調整每個 beat 他們的 Tags，來調整他們的組態設定。



### Configuration Tags

![image-20201002190006222](https://i.imgur.com/HEqwBO9.png)

每個 Tag 的配置方式，可依照佈署環境的狀態來定義想要的 **組態模組化** 的切割方式，例如：

- web-server: 要安裝 Filebeat apache module 來收集 apache log。
- db-server: 要安裝 Metricbeat mysql module 來收集 mysql 的 metrics、也要安裝 system module 來收集主機的 system metrics、也要安裝 Filebeat mysql module 來收集 mysql 的 logs。
- proxy-server: 要安裝 Filebeat 並且使用自定義的 Paths 到指定的路徑去收集 logs。
- elasticsearch-output: 要有一組 output 到 elasticsearch 的設定。

這時每次有安裝某個 beats 時，可以依照該機器的角色，來分配給他合適的 tags。

如此就能簡單的從 Kibana 做到 Beats 的組態集中化的管理。



## 參考資料

- [官方文件 - Beats Central Management](https://www.elastic.co/guide/en/kibana/7.9/managing-beats.html)
- [官方 Blog - Introducting Beats Central Management in Elastic Stack](https://www.elastic.co/blog/introducing-beats-central-management-in-the-elastic-stack)