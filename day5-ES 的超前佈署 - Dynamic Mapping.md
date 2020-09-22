# 喬叔教 Elastic - 05 - ES 的超前佈署 - Dynamic Mapping

## 前言

一開始使用 Elasticsearch 時，簡單的透過一個 HTTP POST 或 PUT 就能將一個 JSON 文件 indexing 進入 Elasticsearch，然後就可以直接透過 `_search` API endpoint 來進行搜尋，如此的簡單，不像一般 Database 要先定義好 table schema，這背後到底是怎麼運作的？

### 進入此章節的先備知識

- 什麼是 Index 及建立 Index 的一些基本操作知識。
- 什麼是 Mapping 及 Mapping 基本操作的知識。

### 此章節的重點學習

- Indexing 文件進入 Elasticsearch 時，ES 如何針對未事前定義欄位產生其 Mapping 設定值。
- 我們如何自己定義自己的**自動判斷欄位型態**的規則。
- 有什麼實用的案例技巧與要注意的地方。
- 最終透過 `Dynamic Template`的設定，來強化我們設計 `Mapping` 時的技巧。

---

## Dynamic Mapping

這個功能是 Elasticsearch 之所以能宣稱他是 schema-less，也就是不用預先定義好資料的 schema 就能直接將文件 indexing 進入 Elasticsearch 的幕後機制，他的做法就是：

> 當一份被 indexing 進入 Elasticsearch 的文件，若是有一個欄位，沒有在 mapping 中被定義時，Elasticsearch 會自動的判斷他的資料型態，並且依照預設或指定的規則，來產生這個新欄位的 mapping 設定。

Dynamic Mapping 的運作主要依照以下兩種機制及其相關的設定：

- **Dynamic field mappings:** 從 JSON 文件，動態偵測欄位型態的規則。
- **Dynamic templates:** 自訂義的動態判斷欄位型態的規則。

以下我們分別就這兩個部份來說明。

### Dynamic Field Mappings

當一個 JSON 文件 indexing 進入 Elasticsearch 時，Dynamic field mapping 會依照 JSON 欄位原本的資料型態，來分別執行判定的規則：

| JSON 的資料型態   | 判定成為的 Elasticsearch 資料型態                            |
| :---------------- | :----------------------------------------------------------- |
| null              | 不會產生這個欄位                                             |
| `true` or `false` | `boolean`                                                    |
| 浮點數            | `float`                                                      |
| 整數              | `long`                                                       |
| 物件              | `object`                                                     |
| 陣列              | 依照陣列內的資料型態決定                                     |
| 字串              | 1. 判定這個字串是否為日期格式。<br />2. 判定這個字串是否為 `double` 或是 `long` 的格式。<br />3. 若都非以上的格式，會直接指派 `text` 型態，並搭配 `keyword` 的 sub-field。 |

其中日期格式預設的規則為：

```
[ "strict_date_optional_time","yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"]
```

> strict_date_optional_time 支援的是一般廣泛被使用在 JSON 中表示日期時間的 [ISO8601](https://www.w3.org/TR/NOTE-datetime) 格式。

也可以在宣告 mapping 時自行定義日期格式的規則：

```
PUT my-index-000001
{
  "mappings": {
    "dynamic_date_formats": ["MM/dd/yyyy"]
  }
}
```



### Dynamic Templates

我們可以在 mapping 宣告時，直接指定 `dynamic template` 的規則，從官方的文件可以看到宣告的格式如下：

![dynamic templates](https://i.imgur.com/wP7z2U8.png)

其中 match conditions 代表的定義為：

- `match_mapping_type`: Elasticsearch 根據 JSON 文件的欄位資料內容，所判斷出來的資料型態是什麼。 例如： `boolean`, `date`, `double`, `long`, `object`, `string`.
- `match` 和 `unmatch`: 欄位的名稱，可以支援萬用字元 `*`。例如： `*_text`, `long_*`.
- `match_pattern`: 一樣是針對欄位的名稱進行比對，不過是以 **full Java regular expression** 的比對方式。
- `path_match` 和 `path_unmatch`: 類似 `match` 與 `unmatch` 是針對欄位的名字，不過多包含整體的路徑，例如： `name.*`, `*.middle`。



## 實用的技巧與注意事項

#### 定義好合適的資料型態

- Elasticsearch 預設的 Dynamic Template 會將 `string` 的欄位指定成 `text` 加上包含 `keyword` sub-field 的型態，如果這個 index 的應用場景是 log，而 log 的格式有先定義好，大部份的字串欄位都不用被搜尋，只有特定的字串欄位會是 `text` 的話，可將預設的字串欄位指定成 `keyword`。

```
PUT my-index-000001
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_keywords": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```

- 如果預設會進入的字串資料很明確就是 `text`，不需要使用到 Aggregation, Sorting 或是 Script 的操作 ，因此不必保留 `keyword` 的 sub-field，即可明確指定為 `text`。

```
PUT my-index-000001
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_text": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "text"
          }
        }
      }
    ]
  }
}
```



### 團隊內部的命名規則

指定好團隊寫入特定 Index 的命名規則，可以簡化設定與錯誤發生的機會，例如：

- 若值是日期時間，則必需是 `_datetime` 結尾。
- 若值是整數的次數，則必需是 `_count` 結尾。
- 將特定型態定義在欄位的開頭，例如： `long_`, `double_`, `int_`。

這樣即可以較單純的設定來滿足日後欄位擴充的管理。

```
PUT my-index-000001
{
  "mappings": {
    "dynamic_templates": [
      {
        "long_field": {
          "match":   "long_*",
          "mapping": {
            "type": "long"
          }
        }
      },
      {
        "double_field": {
          "match":   "double_*",
          "mapping": {
            "type": "double"
          }
        }
      }
    ]
  }
}
```



### 必要的嚴謹，以避免意外發生

#### 1. 請小心設定 `Dynamic Template`，特別是修改原先的 `Dynamic Template`時，否則會發生 `Runtime Error`，也就是 indexing 才會發現有錯。

#### 2. 關閉 Dynamic fields mapping

```
PUT /my_index
{
    "mappings": {
        "dynamic": "strict"
    }
}
```

`dynamic`可以設定為：

- `true`: 執行 dynamic mapping。
- `false`: 不執行 dynamic mapping，並在 indexing 時忽略沒有被宣告的欄位。
- `strict`: 不執行 dynamic mapping，並在 indexing 時遇到沒有宣告的欄位會直接拋出 exception。

#### 3. 關閉 日期 或 數值 的自動判斷

```
PUT my-index-000001
{
  "mappings": {
    "date_detection": false,
    "numeric_detection": true
  }
}
```

> 其中數值的部份指的是在字串內容中是否要嘗試判斷是否為數值，預設是關閉的



## 參考資料

- [官方文件 - Dynamic Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-mapping.html)
- [Elasticsearch: 權威指南 - 動態映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html)

