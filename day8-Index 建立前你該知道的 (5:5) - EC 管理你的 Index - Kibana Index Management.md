# 喬叔教 Elastic - 08 - Index 建立前你該知道的 (5/5) - EC 管理你的 Index - Kibana Index Management

## 前言

前幾天我們介紹到 Index 被建立之前需知道的一些重要項目，包含 

- [ES Index 如何被建立](https://ithelp.ithome.com.tw/articles/10236906) 
- [ES 的超前佈署 - Dynamic Mapping](https://ithelp.ithome.com.tw/articles/10238283)
- [ES 的超前佈署 - Index Template](https://ithelp.ithome.com.tw/articles/10239736)
- [ES Index 的別名(Alias)](https://ithelp.ithome.com.tw/articles/10241035)

這些都是要建立 Index 需要知道的管理工具與運作原理，今天我們會回到 Elastic Cloud，如何透過 Kibana 來進行 Index 的管理。

### 此章節的重點學習

- 如何使用 Kibana 的 GUI 介面來進行 Index Management 的操作。

---

## Kibana 的入口

有些第一次使用 Kibana 的朋友，會被左邊落落長的功能選單搞得頭很痛，不知道從何進入我們 Index 的管理，請直接把左邊工作選單展開，拉到最底下的 **Management** 的區塊，選擇 **Stack Management**。

![Kibana Home](https://i.imgur.com/6QUx6TG.png)

再來就是進入到我們今天的主題 **Index Management**。

![Stack Management](https://i.imgur.com/Q7tnNrv.png)



## Index Management

![Index Management](https://i.imgur.com/tY5Tpu2.png)

進入到 Index Management 時，我們可以看到主要的四個 Tab：

- Indices：這就是我們管理 Index 基本的畫面，可以查列我們 Elasticsearch 中所有的 Index，以及進行基本的管理操作。
- Data Streams：這部份是搭配 Ingest Manager 來建立 time-series 類型的資料管理方式，不過因為目前還在 Beta 階段，這次暫時不討論他。
- Index Templates：管理 Index Templates 的畫面。
- Component Templates：管理 Component Template 的畫面。

### Indices

基本上 Index 的建立，還是必須透過 Create Index API，或是真的把一筆 Document indexing 進入 Elasticsearch 中，而由 ES 自動依照 Index Template 來建立 Index，這裡可以列出所有的 Index 及顯示他的基本資料。

![image-20200923024054396](https://i.imgur.com/cHQuboF.png)



這邊列出幾個重點功能：

- 可以同時對多個 Index 進行管理的操作。

![index management - manage indices](https://i.imgur.com/FhucpSA.png)

- 可以簡單明瞭的知道現在 Index 的各種 Settings, Mappings, Stats，也可直接在 UI 上直接操作進行 settings 修改。

![index management edit settings](https://i.imgur.com/HhCt8sq.png)

- 可以過濾掉系統預設、或是 Elastic 家族某些產品產生的 Index，能過濾當然也能取消嘍~

![include hidden indices](https://i.imgur.com/mGSqLaX.png)

> 所謂 hidden 的定義，就是 Index name 的開頭是否是 `.`，是的話就是屬於隱藏的 Index。



### Index Templates

再來進入的是 Index templates，這部份就是我們先前介紹的東西，透過 Kibana GUI 的畫面來進行管理。

![image-20200923025544596](https://i.imgur.com/5sBot0Q.png)

這裡要分享一個小技巧，在 Kibana 的 UI 上，會顯示這個 Index Template 是否是 Managed，而這邊會發現這個資訊其實是存在 `Metadata` 中，這個 Metadata 在之前的介紹有提到，是個可以自行定義的 JSON object ，不過我們既然能訂、能查出來，同時若前端有些應用，就能直接配合這欄位來設計及使用，例如這部份他們就把這個 `managed` 的屬行直接訂義在 Index Template 裡。

![Screen Shot 2020-09-23 at 2.59.24 AM](https://i.imgur.com/qVvDS0S.png)



在建立一個新的 Index Template 時，我們先前介紹的功能，也都能在畫面上直接操作、設定及產生，有一些部份還是需要告 JSON object 來宣告，例如：`Aliases`

![create template](https://i.imgur.com/0q89Jt2.png)



### Component Template

這部份和 Index Template 差別不大，建立、設定的畫面都差不多，主要是可以針對 `In use` 或是 `Not in use` 來進行過濾，也可以看到 `Usage count`，在管理方便於將根本沒在用的 Component Template 找出，並進行清理的處理。

![component template](https://i.imgur.com/vMygCLQ.png)



## 結論

透過 Kibana 的 Index Management，讓不少要手動操作的 Index 管理處理，方便了非常多，不過回歸到 Production 的管理上，最好將所有的設定都進入**版本控管**(例如：git)，所以如果是 index template, component template, 這些最好也都能用 Configuration Management (例如：Ansible, Puppet) 的工作來設置，並且都進入版控，以確保**管理 Index 的設置**有被良好的管理。



## 參考資料

- [官方文件 - Index Management](https://www.elastic.co/guide/en/kibana/7.9/managing-indices.html)



