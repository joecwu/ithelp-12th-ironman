# 喬叔教 Elastic - 01 - 前言

這次會參賽是被老婆推坑，明明平常工作已忙到不可開交，抱持者不確定是否能完賽的心情、要死也要拖個人一起下水的心態，硬是拉了個同事來組隊參賽，希望籍此機會，將我曾經花過不少時間累積的心法，及多年教學相長累積的經驗能分享給大家。

## 喬叔的 Elastic 經歷

在開始閱讀這系列的文章之前，和大家介紹喬叔在 Elastic 的相關背景

- 超過20年的程式開發經驗，7年以上 Elasticsearch 的使用經驗。
- 2013年 導入 Elasticsearch 於千萬級使用者的跨國產品中，提供多國語言的搜尋功能。
- 2013年 於舊金山參加原廠開設的 Core Elasticsearch Training。
- 2015年 創業時大量使用 Elastic Stack 於新創產品中的功能開發、大數據分析、運維監控。
- 2018年 成為台灣第一位 Elastic Certified Engineer
- 超過五年的 Elasticsearch 專業課程及企業內部培訓經驗。
- 曾擔任美國某新創公司 Distributed & Search Solution Consultant、並協助多間企業提供 Elastic 相關的技術支援及服務。

## 這系列文章的主題方向

由於網路上已經有許多基礎的入門文章，我也不想多花時間寫同樣的介紹，因此這次主題的文章不會專注在太基礎的操作介紹，若這部份還沒有經驗的朋友們，建議可以先閱讀這次鐵人賽 [Elastic Stack on Cloud 的系列文章](https://ithelp.ithome.com.tw/2020-12th-ironman/elastic)、[官方的說明文件(英文為主，少部份中文)](https://www.elastic.co/guide/index.html)、[官方的免費訓練課程(英文)](https://www.elastic.co/training/free)，裡面都有許多不錯的基礎介紹，或是也可以參加我在外開設的基礎實務培訓班(肯定是親切的中文)。

回想過往我在學習 Elastic Stack 的過程中，很容易找到入門的文章，但往往過程中某些設定背後的原理不清楚、一些需注意的事項沒注意到，而導致誤用或是繞了遠路，因此在這次的文章中，我會從 Elastic Cloud 的使用情境來出發，在過程中深入介紹基本的原理、重要的功能、某個設置的背後代表什麼意思、甚至有一些某些 **SaaS 版的 Elastic Cloud** 才有免費提供而 **自架設且免費授權的 Elastic Cloud Enterprise** 沒有的功能…等。

連續寫30篇文章是個鐵人的挑戰，閱讀完30篇文章並學習吸收也是件挺花時間的挑戰，就讓我們一起接受這挑戰、一起學習與成長，就從開始嘗試14天的 [Elastic Cloud 免費試用](https://cloud.elastic.co/) 吧！(為了這次的鐵人賽的參賽者，Elastic 官方有[另外的入口](https://ela.st/ithome-hackathon)，可以取得30天的免費試用，有興趣的朋友可以進去試試。)

![開始註冊Elastic Cloud](https://i.imgur.com/RyDzlYJ.png)