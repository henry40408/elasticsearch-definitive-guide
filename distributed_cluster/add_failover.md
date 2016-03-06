# 增加故障轉移

在單一節點上執行意味著有單點故障的風險，沒有資料冗餘備份。幸運的是，我們可以啟用另一個節點來保護我們的資料。

****
> ### 啟動第二個節點

為了測試在增加第二個節點後發生了什麼，你可以使用與第一個節點相同的方式啟動第二個節點（你可以參考 入門-》安裝-》執行 Elasticsearch 一章），而且在同一個目錄——多個節點可以分享同一個目錄。

只要第二個節點與第一個節點的 `cluster.name` 相同（參見`./config/elasticsearch.yml`檔案中的配置），它就能自動發現並加入到第一個節點的叢集中。如果沒有，請結合日誌找出問題所在。這可能是多播（multicast）被禁用，或者防火牆阻止了節點間的通訊。
****

如果我們啟動了第二個節點，這個叢集應該叫做 **雙節點叢集(cluster-two-nodes)**

雙節點叢集——所有的主分片和從分片都被分配:
![雙節點叢集](../images/02-03_two_nodes.png)

當第二個節點加入後，就產生了三個 **從分片(replica shards)** ，它們分別於三個主分片一一對應。也就意味著即使有一個節點發生了損壞，我們可以保證資料的完整性。

所有被索引的新文件都會先被儲存在主分片中，之後才會被平行復制到關聯的從分片上。這樣可以確保我們的文件在主節點和從節點上都能被檢索。

當前，`cluster-health` 的狀態為 `green`，這意味著所有的6個分片（三個主分片和三個從分片）都已啟用：

```Js
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       2,
   "number_of_data_nodes":  2,
   "active_primary_shards": 3,
   "active_shards":         6,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```

1. 叢集的 `status` 是 `green`.

我們的叢集不僅功能齊全的，並且具有**高可用性**。
