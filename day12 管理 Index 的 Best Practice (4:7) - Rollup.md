# 喬叔教 Elastic - 12 - 管理 Index 的 Best Practice (4/7) - Rollup

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

Rollup 是 Elasticsearch 6.3 推出的新功能，目前還在 Experimental 階段，官方不建議直接使用在 Production 環境中，不過版本的演進已從 6.3 發展到了到 7.9，相信已經發展到了接近成熟的階段 ，因此這次還是選擇介紹這功能，不過實際在正式環境中，建議還是等官方正式移除 Experimental 的階段後再開始使用。



### 進入此章節的先備知識

- Elasticsearch Query DSL。
- Elasticsearch Aggregation 的基本認識。

### 此章節的重點學習

- Rollup 的使用方式、時機及限制。
- 透過 Kibana 來建立 Rollup Jobs。

---

## 什麼是 Rollup

Rollup 是一個把用來依時間分析的歷史資料的時間 **顆粒度( Granularity )** 變大，以節省空間的 **定期執行** 的機制。

例如我們收集的即時資料 1秒鐘 有 1000筆，每天就會有 86,400,000 筆資料，而一年後就會有 365 * 86,400,000 筆，這個佔用的儲存空間是很大的，而好處是我們可以隨時查到精確的某一筆原始資料。

但平常在分析上，若我們將時間顆拉度拉最小只能看到 **分鐘** 為單位，這時只需要保存以1分鐘產生一筆彙總的資料，這樣的筆數就只剩下 86,400 筆，只剩下原來的 1/1000。 (當然實際大小可能有所不同，因為彙總的資料的資料大小可能和原始資料有點落差。)



## 使用 Rollup 的好處

- 節省磁碟**儲存空間**
- 節省執行查詢的**記憶體**快取空間
- 節省 JVM heap 載入 Index 的**記憶體**空間
- 查詢的**速度**比較快



## 如何在 Kibana 中建立 Rollup Job

### 建立 Rollup Job

首先，進入 Kibana > Stack Management 後，點選左側 Data 區塊中的 Rollup Jobs。

![kibana rollup job](https://i.imgur.com/O1eIneU.png)

點 Create rollup job 之後，會進入設定頁面：

![create rollup job 1](https://i.imgur.com/f6MEft5.png)

這邊的設定基本上都蠻直覺的，依照旁邊的說明設定即可。

- **Name:** 幫 Rollup Job 取個名字。
- **Data flow:** 指定 Index pattern 以及 Rollup 產生的 Index 名字。
- **Schedule:** 這個 Rollup Job 執行的頻率。
- **Ho manay documents do you want to roll up at a time:** 執行時批次處理的大小，數字愈來處理速度愈快，但記憶體也耗的愈多。
- **How long should the rollup job wait before rolling up new data:** 在執行 Rollup 時，可以設定一個 Latency 執，這個 Latency 指的是資料在 ingest 進入 ES 時，有可能會有一些延遲 ，一但設了這個 Latency，Rollup Job 就會多等到 Latency 的時間過了之後，才會處理這部份的資料。

> 這邊有個要注意的 `Index pattern` 不應該包含到 `Rollup index name` ，上圖就是一個錯誤的例子，這樣會造成處理邏輯上的錯誤，如果你設定了這樣的配置，最終會看到以下這樣的錯誤畫面。
>
> ![create rollup job error](https://i.imgur.com/YXz6V3I.png)



接下來要分別設定 Date histogram 的時間顆粒度：

![create rollup job 2](https://i.imgur.com/peJKjTG.png)



設定有哪些欄位會使用到 Term bucketing：

![create rollup job 3](https://i.imgur.com/9wgDGss.png)



哪些欄位可能會進行 Histogram 的 aggregation：

![create rollup job 4](https://i.imgur.com/il4zcuV.png)



哪些欄位會使用到 Metrics，這邊請依照資料的特性來判斷，有些沒必要的就不用勾選了。

![create rollup job 5](https://i.imgur.com/NCAt07w.png)



最後 Review 完沒問題時，就可以直接建立。

![create rollup job 6](https://i.imgur.com/a0vPB1d.png)



### 查看 Rollup Jobs

當建立完成後，在 Rollup Jobs 的選單中可以看到我們建立的這個 Job。

![view rollup jobs](https://i.imgur.com/ouvdDCZ.png)



點開後也可以看到他的 Stats，包含目前已經處理了多少 Documents、Pages、以及執行過多少次。

![image-20200927200414569](https://i.imgur.com/GBRM6zu.png)



這時到 Index Management 來查看 Elasticsearch 中的 Index，就可以看到由這個 Rollup Job 所產生的 Index 了。

> 記得要把上面 `Include rollup indices` 打勾，才會看得到。

![rollup index](https://i.imgur.com/p5Nycc4.png)

這時 Rollup Job 產生的資料已經可以使用了。

而且我們可以看到， `rollup-logstash-daily` 佔用的是 **446mb** 的空間，比起整體 `logstash-1` ~ `logstash-10` 的總量，大約只佔了 **1/10** 。



## 使用 Rollup 的資料

### 先來剖析一下 Rollup 產生出來的資料

首先，這是我們測試用的一筆資料，這是一個 HTTP Request 的 Log：

```
{
        "_index" : "logstash-10",
        "_type" : "_doc",
        "_id" : "VGlRz3QBFWvcdj-NvfL6",
        "_score" : 1.0,
        "_source" : {
          "index" : "logstash-10",
          "@timestamp" : "2020-10-27T04:57:02.344Z",
          "ip" : "148.214.137.20",
          "extension" : "jpg",
          "response" : "200",
          "geo" : {
            "coordinates" : {
              "lat" : 34.48339944,
              "lon" : -104.2171967
            },
            "src" : "AU",
            "dest" : "ID",
            "srcdest" : "AU:ID"
          },
          "@tags" : [
            "warning",
            "info"
          ],
          "utc_time" : "2020-10-27T04:57:02.344Z",
          "referer" : "http://www.slate.com/success/lisa-nowak",
          "agent" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
          "clientip" : "148.214.137.20",
          "bytes" : 3586,
          "host" : "media-for-the-masses.theacademyofperformingartsandscience.org",
          "request" : "/uploads/konstantin-feoktistov.jpg",
          "url" : "https://media-for-the-masses.theacademyofperformingartsandscience.org/uploads/konstantin-feoktistov.jpg",
          "@message" : "148.214.137.20 - - [2020-10-27T04:57:02.344Z] \"GET /uploads/konstantin-feoktistov.jpg HTTP/1.1\" 200 3586 \"-\" \"Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)\"",
          "spaces" : "this   is   a   thing    with lots of     spaces       wwwwoooooo",
          "xss" : """<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8z/C/HgAGgwJ/lK3Q6wAAAABJRU5ErkJggg==" onload="alert('XSS found via img-onload!')"><script>alert("XSS found via script-tag!")</script>""",
          "headings" : [
            "<h3>chris-hadfield</h5>",
            "http://facebook.com/success/christopher-ferguson"
          ],
          "links" : [
            "alexander-poleshchuk@www.slate.com",
            "http://www.slate.com/login/pavel-belyayev",
            "www.twitter.com"
          ],
          "relatedContent" : [ ],
          "machine" : {
            "os" : "osx",
            "ram" : 6442450944
          },
          "longValues" : "efearpjmayxX"
        }
      }
```



而經過 Rollup Job 產生出來的資料，若我們直接用 `_search` API 來查詢：

```
GET rollup-logstash-daily/_search
```

會看到他回傳的結果長這樣子：

```
{
        "_index" : "rollup-logstash-daily",
        "_type" : "_doc",
        "_id" : "rollup-logstash-daily$8LKzd9hsjA2IhasDI3Z8kA",
        "_score" : 1.0,
        "_source" : {
          "@timestamp.date_histogram.time_zone" : "Asia/Taipei",
          "memory.histogram.interval" : 1000,
          "memory.histogram.value" : null,
          "bytes.value_count.value" : 1.0,
          "@timestamp.date_histogram._count" : 1,
          "memory.histogram._count" : 1,
          "geo.src.terms.value" : "SD",
          "geo.srcdest.terms.value" : "SD:AO",
          "machine.ram.max.value" : 2.147483648E9,
          "bytes.histogram.interval" : 1000,
          "bytes.histogram.value" : 0.0,
          "geo.dest.terms.value" : "AO",
          "phpmemory.histogram.value" : null,
          "bytes.sum.value" : 271.0,
          "bytes.min.value" : 271.0,
          "geo.src.terms._count" : 1,
          "geo.dest.terms._count" : 1,
          "_rollup.id" : "rollup-logstash-daily",
          "response.keyword.terms.value" : "200",
          "@timestamp.date_histogram.timestamp" : 1598544000000,
          "referer.terms._count" : 1,
          "referer.terms.value" : "http://www.slate.com/error/sigmund-j-hn",
          "bytes.max.value" : 271.0,
          "machine.os.keyword.terms.value" : "ios",
          "url.keyword.terms._count" : 1,
          "@timestamp.date_histogram.interval" : "24h",
          "bytes.avg.value" : 271.0,
          "host.keyword.terms._count" : 1,
          "machine.ram.min.value" : 2.147483648E9,
          "machine.ram.histogram.value" : 2.147483E9,
          "bytes.avg._count" : 1.0,
          "request.keyword.terms._count" : 1,
          "request.keyword.terms.value" : "/canhaz/yuri-artyukhin.gif",
          "phpmemory.histogram.interval" : 1000,
          "phpmemory.histogram._count" : 1,
          "bytes.histogram._count" : 1,
          "geo.srcdest.terms._count" : 1,
          "url.keyword.terms.value" : "https://motion-media.theacademyofperformingartsandscience.org/canhaz/yuri-artyukhin.gif",
          "_rollup.version" : 2,
          "machine.os.keyword.terms._count" : 1,
          "machine.ram.histogram._count" : 1,
          "host.keyword.terms.value" : "motion-media.theacademyofperformingartsandscience.org",
          "response.keyword.terms._count" : 1,
          "machine.ram.avg._count" : 1.0,
          "machine.ram.histogram.interval" : 1000,
          "machine.ram.avg.value" : 2.147483648E9
        }
      },
```

Rollup 後的資料，已經是使用另外的彙總的格式來儲存，所以他已經沒有原本的資料內容了。

### 查詢 Rollup 的資料

從上面的例子可以看到，使用 `_search` 的結果會是長得不一樣的文件，也因此 Rollup 後的資料，若是要進行 search 時，要使用 `_rollup_search` 的 API。

這邊先從原始的資料來查詢：

![rollup-demo-original-search](https://i.imgur.com/DIYONno.png)



再來使用 `_rollup_search` 來執行一樣的 query：

![rollup-demo-rollup-search](https://i.imgur.com/QhMryy0.png)

可以看到結果是一樣的，通常如果是有一些 metrics 的數值運算的話，有可能會有小誤差。

另外在使用上會有一些限制，以下會做相關的介紹。

> 這邊特別要說一下，使用 `_rollup_search` 若是跨到 Rollup Index 與 原始的資料 時，不會計算重覆的資料，這讓 Rollup 與 Live data 混搭使用非常的方便。

### 使用的限制

Rollup 的資料在使用上有一些限制

- `_rollup_search` 可同時包含多個非 rollup index，但一次只能包含一個 rollup index。
- 在使用 Aggregation 時，只有 rollup job 中有被指定的欄位可以被使用。
- 時間顆粒度只能以 rollup 的基本單位往上的乘數，例如設定的是 `3d` ，就只能用 `3d`, `6d`, `9d` ...以此類推。
- Query 的方式只能使用 `Term`, `Terms`, `Range`, `MatchAll`, 以及 `Boolean`, `ContantScore` 等 compound 查詢。
- 能使用的 Bucketing Aggregation 為：`Date Histogram`, `Histogram`, `Terms。`
- 能使用的 Metrics Aggregation 為：`Min`, `Max`, `Sum`, `Average`, `Value Count`。
- 若是在建立  Rollup Job 時有指定時區的話，在使用 Date Histogram 時也一定要指定同樣的時區。



### 在 Kibana 建立新的 Index Pattern

在 Kibana 中，若要使用 Rollup 的資料，要特別建立一個 `Rollup Index Pattern` ，如下圖：

![image-20200927201453586](https://i.imgur.com/y2iCCdo.png)

建立好這個 Index Pattern 後，就可以在 Discover, Virtual, Dashboard 來使用了。

![image-20200927213635040](https://i.imgur.com/94CK31v.png)



## 使用建議與注意事項

### Rollup 目前還不支援 Rollover

(2020.09.29 更新) Rollup 目前產生出來的 Index 還無法套用 Rollover 的機制，這部份的支援已在官方的討論中，有可能在未來會直接整併成 Index Lifecycle Management 中的其中一個 Action，可參考這篇 [Githib Issue](https://github.com/elastic/elasticsearch/issues/48003)。

### Rollup 的 Interval 支援度

ES 在 Rollup 的支援上，除了讓你整理成較大的時間**顆粒度**來儲存資料，例如以 `1天` 為單位。

如果你在進行 Query 與 Aggregate 時，當然最小的顆粒度就是 1天，但如果你要的單位是比1天還大的顆粒度，如：`1週`、`1個月`、`一季`、`一年`…等，你**不需要**為這每個顆粒度建立獨立的 Rollup job，這些都能在 `_rollup_search` 的計算時，直接以 `1天` 的資料來彙總計算出來。

### 盡量使用 Fixed time interval 而不要用 Calendar time interval

這部份的說明比較複雜，建議直接參考 [官方文件的說明](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/rollup-understanding-groups.html#rollup-understanding-group-intervals) ，主要的概念是，建議少用 Calendar 的"一天"，而是用"24小時"、少用 Calendar 的"一個月"，而是用"30天"，這樣的 Interval 單位，因為 Calendar 時間在查詢的時，會是比較難複雜且使用時的限制較多。

> 另外若使用上是有指定時區的話，記得設定好時區，這樣在時間的切分上才會如使用的預期。

### 參考騰訊的做法

騰訊針對 Metrics 類型的資料：

- 在原始的即時資料監控下，監控的時間顆粒度是 **10秒**。
- 在一個月以前的監控數據，時間顆粒度是 **1小時**。

這樣的顆粒度，是在設計 Rollup Job 時需先考量清楚的，確保使用的情境與需求來製訂規則。

> Rollup Job 一但決定下去，就無法改變，除非重建 Rollup Job。



## 參考資料

- [官方 Blog - Create, Manage, and Visualize Rollup Data in Kibana](https://www.elastic.co/blog/how-to-create-manage-and-visualize-elasticsearch-rollup-data-in-kibana)
- [騰訊萬億級 Elasticsearch 技術解密](https://cloud.tencent.com/developer/article/1598364)