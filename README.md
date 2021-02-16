# ithelp-12th-ironman
參加 iThome 2020年 IT邦幫忙 鐵人賽 Elastic Stack on Cloud 主題 - 喬叔帶你上手Elastic Stack

----
# 喬叔教 Elastic - 30天文章總整理 + 完賽心得

這次參加鐵人賽 30 天分享的文章，透過這篇做個總整理，並且建立一個索引目錄，讓大家能透過這系列文章的架構找到你可以參考的資料。

## 喬叔教 Elastic 文章總整理

在前言裡，有描述到這次參賽的原由、喬叔在 Elastic 的背景、這次文章撰寫的主要方向的概念介紹。

- [前言](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day1-%E5%89%8D%E8%A8%80.md)

### Elastic Cloud 如何建立 Deployment

這個系列文章主要介紹使用 Elastic Cloud 時，在選擇 Deployment 的時候，你應該要先知道的知識、以及如何進行選擇。

- [(1/2) - ES Node 的種類](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day2-Elastic%20Cloud%20%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%20Deployment%20(1:2)%20-%20ES%20Node%20%E7%9A%84%E7%A8%AE%E9%A1%9E.md)
- [(2/2) - 配置的選擇](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day3-Elastic%20Cloud%20%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%20Deployment%20(2:2)%20-%20%E9%85%8D%E7%BD%AE%E7%9A%84%E9%81%B8%E6%93%87.md)

### Index 建立前你該知道的

當你架起了 Elasticsearch Cluster 後，要把資料正式的放入 Elasticsearch 來使用之前，你應該要知道的一些進階知識。

- [(1/5) ES Index 如何被建立](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day4-Index%20%E5%BB%BA%E7%AB%8B%E5%89%8D%E4%BD%A0%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%20(1:5)%20-%20ES%20Index%20%E5%A6%82%E4%BD%95%E8%A2%AB%E5%BB%BA%E7%AB%8B.md)
- [(2/5) ES 的超前佈署 - Dynamic Mapping](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day5-Index%20%E5%BB%BA%E7%AB%8B%E5%89%8D%E4%BD%A0%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%20(2:5)%20-%20ES%20%E7%9A%84%E8%B6%85%E5%89%8D%E4%BD%88%E7%BD%B2%20-%20Dynamic%20Mapping.md)
- [(3/5) ES 的超前佈署 - Index Template](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day6-Index%20%E5%BB%BA%E7%AB%8B%E5%89%8D%E4%BD%A0%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%20(3:5)%20-%20ES%20%E7%9A%84%E8%B6%85%E5%89%8D%E4%BD%88%E7%BD%B2%20-%20Index%20Template.md)
- [(4/5) ES Index 的別名 (Alias)](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day7-Index%20%E5%BB%BA%E7%AB%8B%E5%89%8D%E4%BD%A0%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%20(4:5)%20-%20ES%20Index%20%E7%9A%84%E5%88%A5%E5%90%8D%20(Alias).md)
- [(5/5) EC 管理你的 Index - Kibana Index](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day8-Index%20%E5%BB%BA%E7%AB%8B%E5%89%8D%E4%BD%A0%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%20(5:5)%20-%20EC%20%E7%AE%A1%E7%90%86%E4%BD%A0%E7%9A%84%20Index%20-%20Kibana%20Index%20Management.md)

### 管理 Index 的 Best Practices

Index 建立起來之後，如何管理你的 Index、也就是如何管理你在 Elasticsearch 中的資料，這裡介紹了各種推薦的工具與實踐的技巧。

- [(1/7) - Shard 的數量與 Rollover & Shrink API](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day9%20%E7%AE%A1%E7%90%86%20Index%20%E7%9A%84%20Best%20Practices%20-%20(1:7)%20Shard%20%E7%9A%84%E6%95%B8%E9%87%8F%E8%88%87%20Rollover%20%26%20Shrink%20API.md)
- [(2/7) - 三溫暖架構 - Hot Warm Cold Architecture](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day10%20%E7%AE%A1%E7%90%86%20Index%20%E7%9A%84%20Best%20Practices%20-%20(2:7)%20%E4%B8%89%E6%BA%AB%E6%9A%96%E6%9E%B6%E6%A7%8B%20-%20Hot%20Warm%20Cold%20Architecture.md)
- [(3/7) - Index Lifecycle Management (ILM)](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day11%20%E7%AE%A1%E7%90%86%20Index%20%E7%9A%84%20Best%20Practices%20-%20(3:7)%20Index%20Lifecycle%20Management%20(ILM).md)
- [(4/7) - Rollup](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day12%20%E7%AE%A1%E7%90%86%20Index%20%E7%9A%84%20Best%20Practice%20(4:7)%20-%20Rollup.md)
- [(5/7) - Transform](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day13%20%E7%AE%A1%E7%90%86%20Index%20%E7%9A%84%20Best%20Practice%20(5:7)%20-%20Transform.md)
- [(6/7) - Snapshot Lifecycle Management (SLM)](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day14%20%E7%AE%A1%E7%90%86%20Index%20%E7%9A%84%20Best%20Practice%20(6:7)%20-%20Snapshot%20Lifecycle%20Management%20(SLM).md)
- [(7/7) - 總結](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day15%20%E7%AE%A1%E7%90%86%20Index%20%E7%9A%84%20Best%20Practice%20(7:7)%20-%20%E7%B8%BD%E7%B5%90.md)

### Elastic Cloud 比免費版還多的功能

Elastic Stack 包含了各種的功能，針對 SaaS 服務中 Standard 版本的功能，以及自己架設 (on-premise) 的 Basic 版本，有什麼差異? 如果你用 Elastic 官方代管的 SaaS 服務，最基本的版本就能得到自行架設要花大錢買進階 License 才能得到的功能有哪些？

- [(1/6) Elastic Stack 的方案比較與銷售方式](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day16%20Elastic%20Cloud%20%E6%AF%94%E5%85%8D%E8%B2%BB%E7%89%88%E9%82%84%E5%A4%9A%E7%9A%84%E5%8A%9F%E8%83%BD%20-%20(1:6)%20Centralized%20Beats%20management.md)
- [(2/6) Centralized Beats Management](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day17%20Elastic%20Cloud%20%E6%AF%94%E5%85%8D%E8%B2%BB%E7%89%88%E9%82%84%E5%A4%9A%E7%9A%84%E5%8A%9F%E8%83%BD%20-%20(2:6)%20Centralized%20Beats%20management.md)
- [(3/6) Centralized Pipeline Management](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day18%20Elastic%20Cloud%20%E6%AF%94%E5%85%8D%E8%B2%BB%E7%89%88%E9%82%84%E5%A4%9A%E7%9A%84%E5%8A%9F%E8%83%BD%20-%20(36)%20Centralized%20Logstash%20pipeline%20management.md)
- [(4/6) Watcher](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day19%20Elastic%20Cloud%20%E6%AF%94%E5%85%8D%E8%B2%BB%E7%89%88%E9%82%84%E5%A4%9A%E7%9A%84%E5%8A%9F%E8%83%BD%20-%20(46)%20Watcher.md)
- [(5/6) Elasticsearch Token Service](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day20%20Elastic%20Cloud%20%E6%AF%94%E5%85%8D%E8%B2%BB%E7%89%88%E9%82%84%E5%A4%9A%E7%9A%84%E5%8A%9F%E8%83%BD%20-%20(56)%20Elasticsearch%20Token%20Service.md)
- [(6/6) Multi-stack monitoring & Automatic stack issue alerts](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day21%20Elastic%20Cloud%20%E6%AF%94%E5%85%8D%E8%B2%BB%E7%89%88%E9%82%84%E5%A4%9A%E7%9A%84%E5%8A%9F%E8%83%BD%20-%20(66)%20Multi-stack%20monitoring%20%26%20Automatic%20stack%20issue%20alerts.md)

### 向 App Search 學習怎麼用 Elasticsearch

App Search 是使用 Elasticsearch 做成的產品，這個產品的目的是幫你配置好一般搜尋功能需求的基本最佳方案，讓 一般網站 或 App 能直接簡單的就拿來使用，想知道 Elasticsearch 可以怎麼被使用，當然就是從剖析 App Search 怎麼使用 Elasticsearch 來學習。

- [(1/5) - 揭開 App Search 的面紗](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day22%20-%20%E5%90%91%20App%20Search%20%E5%AD%B8%E7%BF%92%E6%80%8E%E9%BA%BC%E7%94%A8%20Elasticsearch%20(1:5)%20-%20%E6%8F%AD%E9%96%8B%20App%20Search%20%E7%9A%84%E9%9D%A2%E7%B4%97.md)
- [(2/5) - Engine 的 Index Settings 篇](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day23%20-%20%E5%90%91%20App%20Search%20%E5%AD%B8%E7%BF%92%E6%80%8E%E9%BA%BC%E7%94%A8%20Elasticsearch%20(2:5)%20-%20Index%20Settings%20%E7%AF%87.md)
- [(3/5) - Engine 的 Mapping 篇](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day24%20-%20%E5%90%91%20App%20Search%20%E5%AD%B8%E7%BF%92%E6%80%8E%E9%BA%BC%E7%94%A8%20Elasticsearch%20(3:5)%20-%20Mapping%20%E7%AF%87.md)
- [(4/5) - Engine 的 Search 基礎剖析篇](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day25%20-%20%E5%90%91%20App%20Search%20%E5%AD%B8%E7%BF%92%E6%80%8E%E9%BA%BC%E7%94%A8%20Elasticsearch%20(4:5)%20-%20Query%20%E7%AF%87.md)
- [(5/5) - Engine 的 Search 進階剖析篇](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day26%20-%20%E5%90%91%20App%20Search%20%E5%AD%B8%E7%BF%92%E6%80%8E%E9%BA%BC%E7%94%A8%20Elasticsearch%20(5:5)%20-%20Synonym%20%E7%AF%87.md)

### Elasticsearch 的優化技巧

使用 Elasticsearch 時，是否對於效能不滿意？對於硬體資源的成本想進一步優化？這個主題就帶大家來探討，最佳化 Elasticsearch 的各種技巧及注意事項。

- [(1/4) - Indexing 索引效能優化](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day27%20-%20Elasticsearch%20%E7%9A%84%E5%84%AA%E5%8C%96%E6%8A%80%E5%B7%A7%20(1:4)%20-%20Indexing%20%E7%B4%A2%E5%BC%95%E6%95%88%E8%83%BD.md)
- [(2/4) - Searching 搜尋效能優化](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day28%20-%20Elasticsearch%20%E7%9A%84%E5%84%AA%E5%8C%96%E6%8A%80%E5%B7%A7%20(2:4)%20-%20search%20%E6%90%9C%E5%B0%8B%E6%95%88%E8%83%BD.md)
- [(3/4) - Index 的儲存空間最佳化](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day29%20-%20Elasticsearch%20%E7%9A%84%E5%84%AA%E5%8C%96%E6%8A%80%E5%B7%A7%20-%20(3:4)%20index%20%E7%9A%84%E7%A3%81%E7%A2%9F%E7%A9%BA%E9%96%93.md)
- [(4/4) - Shard 的最佳化管理](https://github.com/joecwu/ithelp-12th-ironman/blob/master/day30%20-%20Elasticsearch%20%E7%9A%84%E5%84%AA%E5%8C%96%E6%8A%80%E5%B7%A7%20(4:4)%20-%20Shard%20%E7%9A%84%E6%9C%80%E4%BD%B3%E5%8C%96%E7%AE%A1%E7%90%86.md)

## 完賽感言

如同一位前輩所說的，平常好好的都沒事，偏偏參加鐵人賽的三十天就一定會有各種你意料不到的狀況不斷出現，生病破病超過一週、工作上的各種狀況不斷發生、家裡的狀況、連假滿滿三整天都去上超過11小時的課…，再怎麼苦總之最後熬過來了，就是一個"爽"字。

這次文章幾乎沒有預寫，只有先寫前三篇(含前言)，一開賽馬上就用光，所以幾乎所有文章都是前一晚寫的，一開頭也沒有完整的規劃架構，只有主要的概念，過程中不斷的修改、邊寫邊決定下一個塊主題要寫什麼，也曾經幾次 refactor 前面的文章標題，總之最後產生出來這系列的文章內容，還算滿意，有將我曾經一直想補充進入教材、或是還沒時間摸索的功能，趁這次鐵人賽花時間研究並整理成文章，我自己也在此過程中又學習了不少細節，另外 backlog 中還有不少 topics 最後是沒有寫出來的，這些未來應該會再一併規劃並且開設另外的新課程。

感謝團長的揪團、也感謝小孩沒滿三個月就被我拖下水，一邊餵奶一邊寫文章的 Edward，我們三個人是一個非常棒的互相傷害 (牽制) 組合，就這樣默默的成團、完賽、學習與成長，美中不足的是我們 **搭著ESTC飛上天** 的三個人都沒買 ESTC 啊!!!

希望不只是我們在這過程中自我成長與挑戰，也期待這些文章的知識產出能幫到需要的人，為軟體產業帶來一些貢獻。

(下圖藍框就是我們參加鐵人賽這30天的 ESTC 走勢，嘆~~)

![Screen Shot 2020-10-30 at 1.39.54 AM](https://i.imgur.com/kW2sgmx.png)

