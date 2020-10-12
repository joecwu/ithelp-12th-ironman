# 喬叔教 Elastic - 22 - 向 App Search 學習怎麼用 Elasticsearch (1/5) - 揭開 App Search 的面紗

**向 App Search 學習怎麼用 Elasticsearch** 系列文章索引

- [(1/5) - 揭開 App Search 的面紗](https://ithelp.ithome.com.tw/articles/10250168)

- [(2/5) - Engine 的 Index Settings 篇](https://ithelp.ithome.com.tw/articles/10250655)

- [(3/5) - Engine 的 Mapping 篇](https://ithelp.ithome.com.tw/articles/10251102)

- [(4/5) - Engine 的 Search 基礎剖析篇](https://ithelp.ithome.com.tw/articles/10251530)

- [(5/5) - Engine 的 Search 進階剖析篇](https://ithelp.ithome.com.tw/articles/10251945)

---

## 前言

這個系列我們針對 Elastic Stack 產品系列 Enterprise Search 中的 App Search 來進行剖析，看看 App Search 是如何使用 Elasticsearch ，從這個探索的過程中，讓我們期待從中學習一些 App Search 值得我們參考的使用方式。

### 進入此章節的先備知識

- Elasticsearch 的基礎使用知識
- App Search 的基礎使用知識

### 此章節的重點學習

- 初探 App Search 是如何把 Elasticsearch 當成 NoSQL Database 來使用。

---

## App Search 簡介

我這篇不會針對 App Search 有太多細節介紹，剛好這次鐵人賽我參加的團隊隊長 - **少女人妻** ，他這次的主題 [少女人妻的30天Elastic](https://ithelp.ithome.com.tw/users/20116811/ironman/3147) 就有蠻詳細的 App Search 介紹，所以對於 App Search 還不太熟悉的朋，推薦大家去拜讀她的文章。

![img](https://i.imgur.com/Z9gLqGq.png)

(圖片來源：[Solutions: Elastic App Search 入門](https://blog.csdn.net/UbuntuTouch/article/details/105392284))

簡單來說，App Search 是提供了一個 Out of Box Experience (OOBE) 使用 Elasticsearch 搜尋服務的產品，讓使用端以 API 的方式將文件透過 App Search 存入，並且提供 API 能將文件搜尋出來，你不需要知道怎麼使用 Elasticsearch、如何下 Query、如何優化 Query 的方式，這些 App Search 都提供了一套預設配置，讓你基本上可以達到一定好用的程度。

而 App Search 底層不僅是使用 Elasticsearch 當作 Document 的 Index engine，同時也把 Elasticsearch 當作 NoSQL database 來存放 App Search 這個應用程式所需要儲存的資料或是設定值。

## 揭開 App Search 的面紗

### App Search System Indices

當我們使用 Elastic Cloud 並啟用 Enterprise Search 、或是自己架設 App Search 並將他指向某一個 Elasticsearch Cluster 時，我們從 Elasticsearch 的 Index 中，可以發現有許多 `.ent-search` 開頭的 Indices，如下圖：

![app search indexes](https://i.imgur.com/7QK4kd2.png)

> 這圖是使用 ElasticSearch Head chrome plugin 的截圖，因為他的排版較適合列出大量的 index name

這些 Index 許多可以從名字猜出用途，另外這邊列幾個比較重要的：

- 包含 `*-app_search_*` 字眼的：這些用途蠻明的，包含 app search settings, accounts, api_tokens...等，就是存放 App Search 一些 application data 用的。
- `.ent-search-actastic-engine_document_backends_v2`: 這個是存放了 backend level 會用到的 engines 的 meta 資料，有幾個 engines 就有幾筆 document，只有簡單的 engine 建立時間、engine id、是否可讀寫的狀態。
- `.ent.search-actastic-engines_v9`: 這裡存放的是定義每個 engines 的詳細資料。
- `.ent-search-engine-*`: 這個會是每個 engines 產生一個 index，裡面存放的就會是這個 engine 裡面的所有 document。

### 建立 Engine

一開始我們 `.ent-search-actastic-engines_v9` index 會是空的，所以我們透過 App Search UI 來建立一個 Engine。

![image-20201007032436502](https://i.imgur.com/Wsl00pv.png)

建立完成後，我們查詢 `.ent-search-actastic-engines_v9` 

```
GET .ent-search-actastic-engines_v9/_search
```

看看裡面有什麼：

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : ".ent-search-actastic-engines_v9",
        "_type" : "_doc",
        "_id" : "5f7cc70189dec99f64b0dd35",
        "_score" : 1.0,
        "_source" : {
          "id" : "5f7cc70189dec99f64b0dd35",
          "created_at" : "2020-10-06T19:35:29Z",
          "updated_at" : "2020-10-06T19:42:15Z",
          "type_" : "Engine::IndexedEngine",
          "account_id" : null,
          "cluster_id" : "5f5c740c481a32491fcccccd",
          "key" : "JM-N2oBNNbieV2QXRrh-",
          "loco_moco_account_id" : "5f5c740d481a32491fccccd0",
          "managing_application_id" : null,
          "slug" : "joe-test",
          "name" : "joe-test",
          "api_based" : true,
          "last_touched_at" : null,
          "stub" : false,
          "demo" : false,
          "sample" : false,
          "meta_data" : { },
          "source" : null,
          "queued_for_deletion" : null,
          "frito_pie_content_source_id" : null,
          "page_limit" : null,
          "document_count" : 1,
          "moving" : false,
          "deaggregation_requested" : false,
          "index_settings_override" : { },
          "index_create_settings_override" : { },
          "query_boosted_documents_enabled" : false,
          "query_boosted_queries_enabled_override" : false,
          "language" : "zh",
          "source_engine_ids" : [ ]
        }
      }
    ]
  }
}
```

這個就是我們的 Engine 的資料，包含了 engine 名稱 `joe-test` 、我選擇的語系 `zh` …等資訊，另外 `5f7cc70189dec99f64b0dd35` 這個是我們的 Engine Id，這個要記起來，之後會用到。

### Import document

光是建立 Engine，文件沒有放進去的話，還不會產生對應的 Index ，所以我們再次透過 App Search UI 來新增一份文件。

以下是我從預設的範例精簡後的版本：

```
[
  {
    "id": "park_rocky-mountain",
    "title": "Rocky Mountain",
    "description": "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m).",
    "visitors": 4517585,
    "location": "40.4,-105.58",
    "date": "1915-01-26T06:00:00Z"
  }
]
```

透過 UI Import 進 App Search 後，會看到 5 個欄位被加入說 Engine's schema 了。

![image-20201007034222461](https://i.imgur.com/GQJo1XO.png)



這時我們去查看 Index

```
GET _cat/indices?v&index=*engine*
```

會發現多了一個 `.ent-search-engine-5f7cc70189dec99f64b0dd35` 的 Index ，也就是我們這個 Engine 的 Index ，並且 `docs.count` 顯示裡面有一份文件。

![image-20201007034849153](https://i.imgur.com/CLY8ZtD.png)



### Query Indexed Document

我們先簡單的查看這個裡面的 Document 長什麼樣子：

```
GET .ent-search-engine-5f7cc70189dec99f64b0dd35/_search
```

以下是回傳的結果：

```
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : ".ent-search-engine-5f7cc70189dec99f64b0dd35",
        "_type" : "_doc",
        "_id" : "5f7cc89489dec9fc34b0dd38",
        "_score" : 1.0,
        "_source" : {
          "title$string" : "Rocky Mountain",
          "description$string" : "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m).",
          "visitors$string" : "4517585",
          "location$string" : "40.4,-105.58",
          "date$string" : "1915-01-26T06:00:00Z",
          "id" : "5f7cc89489dec9fc34b0dd38",
          "external_id" : "park_rocky-mountain",
          "engine_id" : "5f7cc70189dec99f64b0dd35",
          "__st_text_summary" : "Rocky Mountain Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m). 4517585 40.4,-105.58 1915-01-26T06:00:00Z",
          "__st_expires_after" : null
        }
      }
    ]
  }
}

```

我們可以看到，這個 document 有幾部份：

- `id`, `engine_id`: 這邊都是記錄 engine id。
- `__st_text_summary`: 這個欄位把所有值都合併在一起並使用 text 型態來描述。
- 每個我們定義的欄位名稱，後面都被帶上了他的型態 `$string`。

這就是 App Search 使用 Elasticsearch 存放的 Document 外觀的長相。

### Update Engine Schema

![image-20201007040635761](https://i.imgur.com/hIxnlxc.png)

若是我們更新的欄位的型態，再來看看 Document 的變化：

```
{
  "took" : 256,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : ".ent-search-engine-5f7cc70189dec99f64b0dd35",
        "_type" : "_doc",
        "_id" : "5f7cc89489dec9fc34b0dd38",
        "_score" : 1.0,
        "_source" : {
          "title$string" : "Rocky Mountain",
          "description$string" : "Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m).",
          "visitors$float" : 4517585.0,
          "location$location" : "40.4,-105.58",
          "date$date" : "1915-01-26T06:00:00+00:00",
          "id" : "5f7cc89489dec9fc34b0dd38",
          "external_id" : "park_rocky-mountain",
          "engine_id" : "5f7cc70189dec99f64b0dd35",
          "__st_text_summary" : "Rocky Mountain Bisected north to south by the Continental Divide, this portion of the Rockies has ecosystems varying from over 150 riparian lakes to montane and subalpine forests to treeless alpine tundra. Wildlife including mule deer, bighorn sheep, black bears, and cougars inhabit its igneous mountains and glacial valleys. Longs Peak, a classic Colorado fourteener, and the scenic Bear Lake are popular destinations, as well as the historic Trail Ridge Road, which reaches an elevation of more than 12,000 feet (3,700 m). 4517585 40.4,-105.58 1915-01-26T06:00:00Z",
          "__st_expires_after" : null
        }
      }
    ]
  }
}

```

一但更新 Schema，這些 Document 都會被 re-indexing，並且變成新的 Document，而且改變型態的欄位名稱也都跟著改變了：

- `visitors$string` -> `visitors$float`
- `location$string` -> `location$location`
- `date$string` -> `date$date`

## 小結

到目前為止，我們了解了 App Search 在建立 Engine 時，是如何使用 Elasticsearch 存放相關的資料，以及 Import 一份文件進入 App Search 他是怎麼存放的，後續的文章將會進一步探索 Index Setting, Mapping...等深入的剖析。



## 參考資料

- [Solutions: Elastic App Search 入門](https://blog.csdn.net/UbuntuTouch/article/details/105392284)

