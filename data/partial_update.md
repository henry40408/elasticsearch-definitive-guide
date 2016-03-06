# 更新檔案中的一部分

在《更新》一章中，我們講到了要是想更新一個檔案，那麼就需要去取回資料，更改資料然後將整個檔案進行重新索引。當然，你還可以通過使用`更新`API來做部分更新，比如增加一個計數器。

正如我們提到的，檔案不能被修改，它們只能被替換掉。`更新`API也**必須**遵循這一法則。從表面看來，貌似是檔案被替換了。對內而言，它必須按照_找回-修改-索引_的流程來進行操作與管理。不同之處在於這個流程是在一個片(shard) 中完成的，因此可以節省多個請求所帶來的網路開銷。除了節省了步驟，同時我們也能減少多個程式造成衝突的可能性。

使用`更新`請求最簡單的一種用途就是新增新資料。新的資料會被合併到現有資料中，而如果存在相同的欄位，就會被新的資料所替換。例如我們可以為我們的部落格新增`tags`和`views`欄位：

```js
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

如果請求成功，我們就會收到一個類似於`索引`時返回的內容:

```js
{
   "_index" :   "website",
   "_id" :      "1",
   "_type" :    "blog",
   "_version" : 3
}
```

再次取回資料，你可以在`_source`中看到更新的結果：

```js
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  3,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags": [ "testing" ], <1>
      "views":  0 <1>
   }
}
```

1. 新的資料已經新增到了欄位`_source`中。


#### 使用指令碼進行更新

****

我們將會在《指令碼》一章中學習更詳細的內容，我們現在只需要瞭解一些在Elasticsearch中使用API無法直接完成的自定義行為。預設的指令碼語言叫做MVEL，但是Elasticsearch也支援JavaScript, Groovy 以及 Python。

MVEL是一個簡單高效的JAVA基礎動態指令碼語言，它的語法類似於Javascript。你可以在[Elasticsearch scripting docs](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-scripting.html) 以及 [MVEL website](http://mvel.codehaus.org/Getting+Started+for+2.0)瞭解更多關於MVEL的資訊。

****

指令碼語言可以在`更新`API中被用來修改`_source`中的內容，而它在指令碼中被稱為`ctx._source`。例如，我們可以使用指令碼來增加博文中`views`的數字：
```js
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```
我們同樣可以使用指令碼在`tags`陣列中新增新的tag。在這個例子中，我們把新的tag聲明為一個變數，而不是將他寫死在指令碼中。這樣Elasticsearch就可以重新使用這個指令碼進行tag的新增，而不用再次重新編寫指令碼了：


```js
POST /website/blog/1/_update
{
   "script" : "ctx._source.tags+=new_tag",
   "params" : {
      "new_tag" : "search"
   }
}
```

獲取檔案，後兩項發生了變化：

```js
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  5,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags":  ["testing", "search"], <1>
      "views":  1 <2>
   }
}
```
1. `tags`陣列中出現了`search`。
2. `views`欄位增加了。

我們甚至可以使用`ctx.op`來根據內容選擇是否刪除一個檔案：

```js
POST /website/blog/1/_update
{
   "script" : "ctx.op = ctx._source.views == count ? 'delete' : 'none'",
    "params" : {
        "count": 1
    }
}
```

### 更新一篇可能不存在的檔案

想象一下，我們可能需要在Elasticsearch中儲存一個頁面計數器。每次使用者訪問這個頁面，我們就增加一下當前頁面的計數器。但是如果這是個新的頁面，我們不能確保這個計數器已經存在。如果我們試著去更新一個不存在的檔案，更新操作就會失敗。

為了防止上述情況的發生，我們可以使用`upsert`參數來設定檔案不存在時，它應該被建立：

```js
POST /website/pageviews/1/_update
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
```
首次執行這個請求時，`upsert`的內容會被索引成新的檔案，它將`views`欄位初始化為`1`。當之後再請求時，檔案已經存在，所以`指令碼`更新就會被執行，`views`計數器就會增加。


### 更新和衝突

在本節的開篇我們提到了當_取回_與_重新索引_兩個步驟間的時間越少，發生改變衝突的可能性就越小。但它並不能被完全消除，在`更新`的過程中還可能存在另一個程式進行重新索引的可能性。

為了避免丟失資料，`更新`API會在_獲取_步驟中獲取當前檔案中的`_version`，然後將其傳遞給_重新索引_步驟中的`索引`請求。如果其他的程式在這兩部之間修改了這個檔案，那麼`_version`就會不同，這樣更新就會失敗。

對於很多的局部更新來說，檔案有沒有發生變化實際上是不重要的。例如，兩個程式都要增加頁面瀏覽的計數器，誰先誰後其實並不重要 —— 發生衝突時只需要重新來過即可。

你可以通過設定`retry_on_conflict`參數來設定自動完成這項請求的次數，它的預設值是`0`。

```js
POST /website/pageviews/1/_update?retry_on_conflict=5 <1>
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 0
   }
}
```
1. 失敗前重新嘗試5次

這個參數非常適用於類似於增加計數器這種無關順序的請求，但是還有些情況的順序就是**很重要**的。例如上一節提到的情況，你可以參考樂觀併發控制以及悲觀併發控制來設定檔案的版本號。
