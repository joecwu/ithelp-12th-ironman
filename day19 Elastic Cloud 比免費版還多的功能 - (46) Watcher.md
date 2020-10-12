# 喬叔教 Elastic - 19 - Elastic Cloud 比免費版還多的功能 (4/6) - Watcher

**Elastic Cloud 比免費版還多的功能** 系列文章索引

- [(1/6) Elastic Stack 的方案比較與銷售方式](https://ithelp.ithome.com.tw/articles/10247538)

- [(2/6) Centralized Beats Management](https://ithelp.ithome.com.tw/articles/10248028)

- [(3/6) Centralized Pipeline Management](https://ithelp.ithome.com.tw/articles/10248584)

- [(4/6) Watcher](https://ithelp.ithome.com.tw/articles/10248884)

- [(5/6) Elasticsearch Token Service](https://ithelp.ithome.com.tw/articles/10249181)

- [(6/6) Multi-stack monitoring & Automatic stack issue alerts](https://ithelp.ithome.com.tw/articles/10249630)

---

## 前言

Elastic Stack 主打的 Observability ，含蓋的範圍包括各種 Logs, Metrics, APM, Uptime 等資訊，這些資訊的收集對於掌握系統運作狀態非常有幫助，這些資訊非常的多，要靠人員主動觀察問題會有些不太實際，因此當有某些異常的條件發生時，能主動通知營運人員的功能也就很重要了，在 Elastic Stack 中，X-pack 的 **Watcher** 就是負責這項任務，而這個功能同樣也是要 **Gold License** 以上、或是 Elastic Cloud service **Standard** 以上版本才有授權使用。

### 進入此章節的先備知識

- Elasticsearch Query DSL。

### 此章節的重點學習

- Watcher 的基本介紹與使用觀念。
- 如何使用 Kibana 設定 Watcher。

---

## Watcher 的基本介紹

Watcher 主要是執行許多事先定義好的 Watch，並且在某個 Watch 滿足條件時，會執行預先所指定的 action 的一個機制。

### Watch 的組成

Watch 的設定當中主要包含下面幾個部份：

- **Trigger:** 決定 Watch 被檢查的時機點，例如：每 5 分鐘、或是使用 Cron schedule 來指定執行的時間。
- **Input:** 決定要將什麼東西放入到 Watch 的 Payload 中當成 Input，可包含四種：
  - `simple`: 固定的值。
  - `search`: 依照 Query DSL 從 Elasticsearch 查詢出來的結果。
  - `http`: 使用某個 http 請求的 response。
  - `chain`: 可將上述三種用法依照不同的順序來組合搭配使用。
- **Condition:** 決定 Watch 是否有達到要執行的條件，有以下幾種用法：
  - `always`: 總是執行。
  - `never`: 總是不執行。
  - `compare`: 能參照 Watch payload 的資料加上判斷的比較條件來決定是否執行。
  - `array_compare`: 針對 Watch payload 中 array 類型的資料來進行比較，並決定是否要執行。
  - `script`: 能使用 Painless script 來自己撰寫比較的邏輯。
- **Transform:** 當 Watch 決定要執行時，能透過另外的方式產生不同的 Watch Payload，讓之後的 Actions 能使用。(有時 Condition 的條件與 Action 要使用的資料會不同，這時就能透過 Transform 來準備 Action 要使用的資料)
- **Actions:** 最後要執行的動作是什麼，可以設定多個 actions，目前有支援的是：`email`, `webhook`, `index`, `logging`, `slack`, `pagerduty`, `Jira` ，目前在 actions 的設定時也能指定節流 (throttle) 的機制，也就是同樣的事件，在多久時間內不要重覆通知。
- **Metadata:** 這是開放自由存放 metadata 的欄位。

### Watch 的執行方式

Watch 在執行時，會先依照 Trigger 的時間定義被呼叫起來，並且依照 Input 的定義將 Watch Payload 準備好，接下來會檢查 Condition 的條件是否滿足，如果滿足的話，再確認是否需要 Throttle ，若需要正常執行的話，就會執行 Transform Payload 並且 執行定義的 Actions。

因為 Watcher 支援 [Acknowledgement Watch](https://www.elastic.co/guide/en/elasticsearch/reference/current/actions.html#actions-ack-throttle) 的機制，也就是另外透過 ack watch API 來標示這個 Watch 不要再執要 Action了，因在 `condition met? == no`  的時候，也因為 condition 改變了，所以會去檢查並清除 acked 的狀態，讓這個 Watch 的狀態重設。



![watch execution](https://i.imgur.com/wbMiTQ7.jpg)



其中 Throttle 的運作方式如下，會檢查是否有 `acked` 或是是否達到 `throttle_period` ，再決定是否要繼續接下來的 Actions。



![action throttling](https://i.imgur.com/FQwoOJU.jpg)



### 指定 Watcher Node

如果我們想讓 watcher 只能運作在 Cluster 中的某些 node 身上的話，我們可以用以下的方式來設定。

如同 hot-warm architecture 中的使用方式，我們可以在特定的 node 的 `elasticsearch.yml` 設定 node attribute，例如：

```
node.attr.role: watcher
```

這時我們要去 `.watches` 的設定進行宣告：

```
PUT .watches/_settings
{
  "index.routing.allocation.include.role": "watcher"
}
```

這樣透過限制 `.watches` 這個 index 的資料 routing 方式，限制只有我們安排的 node 會得到這份資料，也就因此才會去執行 Watcher 的任務。



## 使用 Kibana 設定 Watcher

### 建立新 Watcher

進入 **Kibana** > **Stack Management** > **Alerts and Insights** 中，找到 **Watcher**：

![kibana watcher](https://i.imgur.com/0O50tfU.png)

在 Create 時，可以有兩種方式，一個是直接用 UI 建立，或是直接使用預先建立好的 Json。

![watcher-create](https://i.imgur.com/cAk8F08.png)

在 UI 的設定介面中，有些基本的設定方式可以直接使用，不過一些進階的用法並沒有完全支援，進階的用法還是要透過 JSON 的方式來定義，並且請參考 [官方文件](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-alerting.html) 。

![create threshold alert](https://i.imgur.com/NHgba1z.png)



### Elastic Cloud 設定特定 Action 的配置

這邊要注意，如果要使用到 `Slack`, `Jira` 這類型的 Action 時，會需要先在 Elastic Cloud 中定義好這些的帳號設定。

1. 在 Keystore 要先設定好 slack 的 `secure_url` ，請參考 [官方文件](https://www.elastic.co/guide/en/elasticsearch/reference/current/actions-slack.html#configuring-slack)。

![elastic cloud keystore](https://i.imgur.com/1oCpYsM.png)

2. 到 Deployement > Elasticsearch 中，去修改 config。

![elastic cloud slack setting](https://i.imgur.com/BRletRD.png)

這些配置設定完成後，才能在配置中使用 Slack。

### Watcher 執行狀態

我們在 Kibana Watcher 的畫面，可以看到所有定義好的 Watch，也能看到他們最近的執行狀況。

![Screen Shot 2020-10-04 at 4.53.43 PM](https://i.imgur.com/6nuOhPP.png)

也可以進入看每次 Trigger 時間點的執行結果，若是有發生錯誤，也可以進去看到錯誤的原因。

![image-20201004165525240](https://i.imgur.com/OhVgTf5.png)

以上是使用 Kibana 操作 Watcher 的基本使用介紹，建議大家可以從下方的 **參考資料** 進入查閱官方文件的詳細介紹，官方也有一些 [Watcher 的 Examples](https://www.elastic.co/guide/en/elasticsearch/reference/current/example-watches.html) 可以參考，對於了解如何使用 Watcher 會有不少幫助。



## 參考資料

- [官方文件 - How Watcher works](https://www.elastic.co/guide/en/elasticsearch/reference/current/how-watcher-works.html)
- [官方文件 - Watcher](https://www.elastic.co/guide/en/kibana/current/watcher-ui.html)