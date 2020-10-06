# 喬叔教 Elastic - 14 - 管理 Index 的 Best Practice (6/7) - Snapshot Lifecycle Management (SLM)

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

這個系列的文章前面主要介紹的都是如何優化 Index 的儲存空間、執行效率…等各種優化，最後這一部份是資料的備份，雖然 Elasticsearch Cluster 可以有許多的 Nodes, 也能設置多份的 Replica 來確保資料的可靠性，但是定期的資料備份還是不能少的，能有效在發生災難時能救回資料，例如： [微軟 6.5T 的 Elasticsearch 料被駭客刪了](https://www.ithome.com.tw/news/140115) 這樣的事件。

### 進入此章節的先備知識

- Elasticsearch Index 的相關基本知識。
- 若備份在雲端儲存空間的話 (例如 AWS S3)，會要知道這些雲端儲存空間的基本知識。

### 此章節的重點學習

- Snapshot / Restore 的使用方式。
- 如何在 Kibana 建立 Snapshot Policy，以及使用 AWS S3 Repository。

---

## Snapshot

**Snapshot** 是 Elasticsearch 用來備份的方式，這邊要注意一件事，如果你打算備份 Elasticsearch 的資料，千萬不要自己從磁碟區去備份 Elasticsearch 的 data 資料夾內的資料，因為有很大的機率當你要復原時，Elasticsearch 在啟動的檢查中會告訴你資料是毀損的，因此在 Elasticsearch 要備份資料，請使用 **Snapshot**。

### Repository

使用 Snapshot 的時候，第一個要先決定你備份的資料要存哪邊，所以要先產生 Repository。

Repository 主要支援的類型有下面幾種：

- `fs`: shared file system，要使用 file system 來建立 Repository 的話，要先在 `elasticsearch.yml` 設定檔中指定好 `path.repo` 的路徑。

- `repository-s3`: 以 AWS S3 來當 Repository, 要另外安裝 [官方的 Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/7.9/repository-s3.html)。

- `repository-hdfs`: 以 Hadoop HDFS 來當 Repository, 要另外安裝 [官方的 Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/7.9/repository-hdfs.html)。

- `repository-gcs`: 以 Google Cloud Storage 來當 Repository, 要另外安裝 [官方的 Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/7.9/repository-gcs.html)。

- `repository-azure`: 以 Azure 來當 Repository, 要另外安裝 [官方的 Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/7.9/repository-azure.html)。

- `repository-swift`: 這是 [OpenStack Swift 的 Repository 擴充套件](https://github.com/BigDataBoutique/elasticsearch-repository-swift)，是社群開發、非官方的，也是要另外安裝。

  

#### 在 Elastic Cloud 中增加其他 Azure 和 GCP 的 Repository 支援

若是在 Elastic Cloud 中要使用其他的 Repository 時，要先到 Deployement 中去安裝 Plugins。

![ec edit deployement](https://i.imgur.com/v6d2xTN.png)

在 **Elasticsearch plugins, extensions, and settings** 的區塊展開後，就可以看到 repository 的 plugins 可以選擇。

![ec install plugins](https://i.imgur.com/CerADUz.png)

#### 建立 Repository

安裝好之後，在 **Kibana** > **Stack Management** 裡 **Data** 區塊的 **Snapshot and Restore** 就可以 **Register a repository**。

![register a repository](https://i.imgur.com/MvuWnTH.png)

這就就可以看到 Azure, GCS, AWS S3 的支援了。

![image-20200929013923134](https://i.imgur.com/CEkT3o5.png)

這邊以 AWS S3 為例，先在自己的 AWS S3 上建立一個 bucket，然後設定好 IAM 權限：

```
{
  "Statement": [
    {
      "Action": [
        "s3:*"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::bucket-name",
        "arn:aws:s3:::bucket-name/*"
      ]
    }
  ]
}
```

然後再透過 Elastic Cloud 的 Console (不是 Kibana 哦!)，進入 Security 將 IAM 的 Access Key & Secret Key 設定在 Keystore 中。 

> 注意，格式上是 `s3.client.{client}.access_key` 和  `s3.client.{client}.secret_key` 。
>
> 這個 `client` 是回到 Kibana Register repository 時要指定的自訂的 client 名字。

![Screen Shot 2020-09-29 at 2.04.10 AM](https://i.imgur.com/EO3WUNM.png)

接下來就繼續將 Register Repository 的步驟走完。

![image-20200929020623597](https://i.imgur.com/Qhoj17v.png)

建立完成後，也可以點擊 Repository ，並選擇右方的 `Verify repository` 確認是否能正常存取。

![repositories](https://i.imgur.com/w4YREEb.png)



### 建立 Snapshot Policy

決定好備份要儲存的 Repository 後，接下來就可以開始建立 Snapshot Policy 了，這是一個可以定時自動備份，並且決定 Snapshot 要保留多少份、保留多久的機制。

![snapshot policy overview](https://i.imgur.com/UVIgOeT.png)

進入 Create Policy 後，設定 policy 的名字、 Snapshot 的名字，以及選擇要用哪一個 Repository，最後是決定定期的週期規則。

> Repository 一但被某一個 Policy 使用後，就不能重覆被另一個 Policy 使用。

![create policy](https://i.imgur.com/MkDhG0t.png)

再來是選擇要備份的 index 或是 Data stream，以及相關的設定。

![image-20200929021107559](https://i.imgur.com/iEfat0g.png)

**Shapshot retention** 是設定備份要保留多久，以及最少保留的份數及最多保留的份數。

![image-20200929021135240](https://i.imgur.com/NO9QZjM.png)

最後確認一切設置正確後，即可建立。

![create snapshot policy review](https://i.imgur.com/D81nPIa.png)



### 查看 Snapshot Policy 的執行狀況

建立完成後，可以直接 **Run Policy**，並且在 Snapshots 分頁中去看執行的狀態。

![taking snapshot](https://i.imgur.com/G7G4qMc.png)

進入 AWS S3 也可以看到 snapshot 的資料被寫入。

![aws s3 repository](https://i.imgur.com/Ob2lblI.png)



## Restore 復原某個版本的資料

在 **Snapshots** 的畫面中，可以找到你想要回復的那份 Snapshot，並點選後面的 **Restore** 按紐。

![Screen Shot 2020-09-29 at 2.36.15 AM](https://i.imgur.com/Jjgc0Rf.png)



**Restore** 時，可以指定要針對哪些特定的 Index，甚至可以改變 restore 之後的名字 (支援 Regular expression group 的方式來取代名字)。

![restore 1](https://i.imgur.com/Po6XjEA.png)



再來 Index Settings 的部份，可以修改 index settings 甚至是清除原先 Snapshot index 中的某些 index settings。

![image-20200929023902608](https://i.imgur.com/0ONInyb.png)



確認沒問題後直接進行 **Restore snapshot**。

![image-20200929024657214](https://i.imgur.com/MBG4BIm.png)



### 檢視 Restore 的結果

在 **Restore Status** 的頁面中，可以看到執行的 Restore 進度與結果。

![restore result](https://i.imgur.com/dxlBkRf.png)

Restore 完成後，我們確認一下這個 Index `restored_logstash-10` 的確已經被復原回來了。

![image-20200929024828659](https://i.imgur.com/C5dD0kp.png)



### Restore 的版本的支援度

這邊要先注意，Restore 是不允許到舊版的 Elasticsearch Cluster 中，也就是 7.6 版的 snapshot 不能 restore 到 7.5 版的環境。

再來要 Restore 到新版本的 Elasticsearch 的話，請參考下面的表格：

![snapshot restore version matrix](https://i.imgur.com/DGclaM1.png)

只有在這表格支援的版本才能進行 restore。

如果真的要 restore 到版本差異較大的環境時，能做的方法是先 restore 到最新支援 restore 的版本，再來透過遠端 reindex 的方式來進行資料的搬移。



## 參考資料

- [官方文件 - Snapshot and restore](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/snapshot-restore.html)
- [官方文件 - Snapshot/Restore Repository Plugins](https://www.elastic.co/guide/en/elasticsearch/plugins/7.9/repository.html)

