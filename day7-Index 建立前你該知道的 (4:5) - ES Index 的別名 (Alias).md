# 喬叔教 Elastic - 07 - Index 建立前你該知道的 (4/5) - ES Index 的別名 (Alias)

### **Index 建立前你該知道的** 系列文章索引

- [(1/5) ES Index 如何被建立](https://ithelp.ithome.com.tw/articles/10236906)
- [(2/5) ES 的超前佈署 - Dynamic Mapping](https://ithelp.ithome.com.tw/articles/10238283)
- [(3/5) ES 的超前佈署 - Index Template](https://ithelp.ithome.com.tw/articles/10239736)
- [(4/5) ES Index 的別名 (Alias)](https://ithelp.ithome.com.tw/articles/10241035)
- [(5/5) EC 管理你的 Index - Kibana Index](https://ithelp.ithome.com.tw/articles/10242398)

---

### 進入此章節的先備知識

- 知道什麼是 Index。
- 知道如何使用 Search 以及 Filter。

### 此章節的重點學習

- Index Alias 的基本使用方式。
- 學會 Index Alias 一些最佳實踐的做法。

---

## Index Alias 的使用方式

### 使用 `_cat` API，列出所有的 Aliases

```
GET _cat/aliases?v
```

使用到的參數說明：

- `v`: verbose，在回傳的結果顯示標題。
- `alias`：依 alias name 進行篩選。

![_cat list aliases](https://i.imgur.com/d9U4Kma.png)



### Add Alias alias API

建立與更新 index alias，這系列 API 都是針對某個 Index 來操作。

```
PUT /<index>/_alias/<alias>
POST /<index>/_alias/<alias>
PUT /<index>/_aliases/<alias>
POST /<index>/_aliases/<alias>
```

直接以官方的例子來看：

1. 建立一個 `2030` 的 alias，並直接指向 `logs_20302801` 這個 index。

```console
PUT /logs_20302801/_alias/2030
```

2. 建立時，也可以指定 `routing` 的值，或是 `filter` 的條件，來讓這個 alias 有限定的用途。

```
PUT /users/_alias/user_12
{
  "routing" : "12",
  "filter" : {
    "term" : {
      "user_id" : 12
    }
  }
}
```

> 使用 `alias` 搭配 `filter` 的用法，可以想像是一些 Relational Database 有提供的 `View` 這樣的功能，透過指定的條件，讓這個 `alias` 是具有某些限定的存取行為、或是具有明確使用意義的。例如 Movies 的資料有一個 Index，但可以建立 `Top Movies` 或是 `Action Movies` 這樣的 alias ，配合特定的 `filter` 條件，但是都指向 Movies 的 Index。



### Update index alias API

這個 API 不會綁定從某個 Index 出發，而且是可以同時進行多項 alias 相關的操作，整個請求所包含的動作會被封裝是一個 `atomic` 的操作，也就是成功就會全部都成功，其中有一個失敗就會全部都失敗，不會只有某部份成功而其他部份發生失敗。

```
POST /_aliases
```

API 的部份細節直接參考 [官方文件 - Update index alias API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html) ，這邊只快速列出幾個例子：

1. 新增一個 alias `alias2` 並指向 index `my-index-000001` 並且套用特定 `filter` 的規則：

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "my-index-000001",
        "alias": "alias2",
        "filter": { "term": { "user.id": "kimchy" } }
      }
    }
  ]
}
```

2. 透過 `atomic` 的特性，幫 alias 改名字，不會在改名的過程中，造成服務中斷。

```
POST /_aliases
{
  "actions" : [
    { "remove" : { "index" : "test1", "alias" : "alias1" } },
    { "add" : { "index" : "test1", "alias" : "alias2" } }
  ]
}
```

3. 建立 alias 並搭配 `filter` 與 `routing` 的設定：

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "test",
        "alias": "alias2",
        "search_routing": "1,2",
        "index_routing": "2"
      }
    }
  ]
}
```

>  指定 `routing` 可以減少讓 request 跑到其他 shard 運作的時間，能直接強制導到某些 shard 身上。

4. 使用 `alias` 時，若會需要將資料透過 `alias` 來寫入，必預要明確的標示哪個 index 是 `is_write_index` ，這部份的設置在建立 `alias` 與 `index` 的關係時，可以一併加上宣告：

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "test",
        "alias": "alias1",
        "is_write_index": true
      }
    },
    {
      "add": {
        "index": "test2",
        "alias": "alias1"
      }
    }
  ]
}
```

可透過 `atomic` 的特性，來把 `write_index` 的能力，從某個 index 移到另個 index 身上：

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "test",
        "alias": "alias1",
        "is_write_index": false
      }
    }, {
      "add": {
        "index": "test2",
        "alias": "alias1",
        "is_write_index": true
      }
    }
  ]
}
```



## Best Practices

- **盡量全面使用 Index Aliases 來存取 index**：因為要修改 mappings 中已存在的設定時，會需要重建 index、並重新 re-index 資料，因此使用 alias 的話，可以先建立新的 index (新的名字可再額外加上版號，例如 `v2` )，資料從 `v1` 搬到 `v2` 後，再把 alias 直向新的 `v2` index，驗證完成後再刪掉舊的 `v1` index，index 的使用者們因為指向 alias ，也就不用全面更改成新的 index 名字了。
- **善用 Index Aliases 搭配 Filter**：
  1. 減少使用端的查詢複雜化，先將適度的 filter 封裝在 alias 中，明確的定義這個 alias 提供的內容，讓使用端不用處理這部份的邏輯。
  2. 權限的管理，限定使用者只能存取特定的子集合的資料、或是限定時間內的資料，而配合 security 的權限處理，可避免使用者存取到不應取得的資料，或是一口氣查詢太多舊資料導致效能的影響。
- **配合 `routing` 來指定資料寫入到特定的 `shard`**：資料在 Indexing 進入 Elasticsearch 時，會依照 routing value 來進行 hash 的運算 (預設是使用 `_id` 當 routing value，並使用 `murmur3` 的 hash 演算法。)，並依照計算出來的值與 primary shard 的數量來進行 mod 運算，以決定資料要寫入到哪個 shard 身上，但如果有指定的 routing value，就可以決定同樣 routing value 的資料會被計算放到同樣的 shard 身上，這樣對於 performance 的優化或是資料的管理上都可以有許多應用的方式，而 alias 就能配合指定 `routing` 的值來達到這類型的運用。
- **配合 index Lifecycle Management (ILM)**：隨著時間增長的資料，使用 ILM 來管理這些資料時，其中就是搭配 Index Alias 來切換寫入時要指定到的實體 Index 在哪邊。



## 參考資料

- [官方文件 - cat aliases API](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/cat-alias.html)
- [官方文件 - Add index alias](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/indices-add-alias.html)
- [官方文件 - Update index alias API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html)
- [Index Alias - Elasticsearch Best Practice](https://spoon-elastic.com/all-elastic-search-post/elasticsearch-best-practices-index-alias/)

