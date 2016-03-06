# 處理衝突

當你使用`索引`API來更新一個檔案時，我們先看到了原始檔案，然後修改它，最後一次性地將**整個新檔案**進行再次索引處理。Elasticsearch會根據請求發出的順序來選擇出最新的一個檔案進行儲存。但是，如果在你修改檔案的同時其他人也發出了指令，那麼他們的修改將會丟失。

很長時間以來，這其實都不是什麼大問題。或許我們的主要資料還是儲存在一個關聯式資料庫中，而我們只是將為了可以搜尋，才將這些資料拷貝到Elasticsearch中。或許發生多個人同時修改一個檔案的概率很小，又或者這些偶然的資料丟失並不會影響到我們的正常使用。

但是有些時候如果我們丟失了資料就會出**大問題**。想象一下，如果我們使用Elasticsearch來儲存一個網店的商品數量。每當我們賣出一件，我們就會將這個數量減少一個。

突然有一天，老闆決定來個大促銷。瞬間，每秒就產生了多筆交易。並行處理，多個程式來處理交易：

![無併發控制的後果](/images/03-01_concurrency.png "無併發控制的後果")

`web_1`中`庫存量`的變化丟失的原因是`web_2`並不知道它所得到的`庫存量`資料是是過期的。這樣就會導致我們誤認為還有很多貨存，最終顧客就會對我們的行為感到失望。

當我們對資料修改得越頻繁，或者在讀取和更新資料間有越長的空閒時間，我們就越容易丟失掉我們的資料。

以下是兩種能避免在併發更新時丟失資料的方法：

### 悲觀併發控制（PCC）

這一點在關聯式資料庫中被廣泛使用。假設這種情況很容易發生，我們就可以阻止對這一資源的訪問。典型的例子就是當我們在讀取一個資料前先鎖定這一行，然後確保只有讀取到資料的這個執行緒可以修改這一行資料。

### 樂觀併發控制（OCC）

Elasticsearch所使用的。假設這種情況並不會經常發生，也不會去阻止某一資料的訪問。然而，如果基礎資料在我們讀取和寫入的間隔中發生了變化，更新就會失敗。這時候就由程式來決定如何處理這個衝突。例如，它可以重新讀取新資料來進行更新，又或者它可以將這一情況直接反饋給使用者。

## 樂觀併發控制

Elasticsearch是分散式的。當檔案被建立、更新或者刪除時，新版本的檔案就會被複制到叢集中的其他節點上。Elasticsearch即是同步的又是非同步的，也就是說複製的請求被平行發送出去，然後可能會**混亂地**到達目的地。這就需要一種方法能夠保證新的資料不會被舊資料所覆蓋。

我們在上文提到每當有`索引`、`put`和`刪除`的操作時，無論檔案有沒有變化，它的`_version`都會增加。Elasticsearch使用`_version`來確保所有的改變操作都被正確排序。如果一箇舊的版本出現在新版本之後，它就會被忽略掉。

我們可以利用`_version`的優點來確保我們程式修改的資料衝突不會造成資料丟失。我們可以按照我們的想法來指定`_version`的數字。如果數字錯誤，請求就是失敗。

我們來建立一個新的博文:

```js
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```
反饋告訴我們這是一個新建的檔案，它的`_version`是`1`。假設我們要編輯它，把這個資料載入到網頁表單中，修改完畢然後儲存新版本。

首先我們先要得到檔案：

```js
GET /website/blog/1
```


返回結果顯示`_version`為`1`：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}
```
現在，我們試著重新索引檔案以儲存變化，我們這樣指定了`version`的數字：

```js
PUT /website/blog/1?version=1 <1>
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```
1. 我們只希望當索引中檔案的`_version`是`1`時，更新才生效。

請求成功相應，返回內容告訴我們`_version`已經變成了`2`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "1",
  "_version": 2
  "created":  false
}
```
然而，當我們再執行同樣的索引請求，並依舊指定`version=1`時，Elasticsearch就會返回一個`409 Conflict`的響應碼，返回內容如下：

```js
{
  "error" : "VersionConflictEngineException[[website][2] [blog][1]:
             version conflict, current [2], provided [1]]",
  "status" : 409
}
```
這裡面指出了檔案當前的`_version`數字是`2`，而我們要求的數字是`1`。

我們需要做什麼取決於我們程式的需求。比如我們可以告知使用者已經有其它人修改了這個檔案，你應該再儲存之前看一下變化。而對於上文提到的`庫存量`問題，我們可能需要重新讀取一下最新的檔案，然後顯示新的資料。

所有的有關於更新或者刪除檔案的API都支援`version`這個參數，有了它你就通過修改你的程式來使用樂觀併發控制。


### 使用外部系統的版本

還有一種常見的情況就是我們還是使用其他的資料庫來儲存資料，而Elasticsearch只是幫我們檢索資料。這也就意味著主資料庫只要發生的變更，就需要將其拷貝到Elasticsearch中。如果多個程式同時發生，就會產生上文提到的那些併發問題。

如果你的資料庫已經存在了版本號碼，或者也可以代表版本的`時間戳`。這是你就可以在Elasticsearch的查詢字元串後面新增`version_type=external`來使用這些號碼。版本號碼必須要是大於零小於`9.2e+18`（Java中long的最大正值）的整數。

Elasticsearch在處理外部版本號時會與對內部版本號的處理有些不同。它不再是檢查`_version`是否與請求中指定的數值_相同_,而是檢查當前的`_version`是否比指定的數值小。如果請求成功，那麼外部的版本號就會被儲存到檔案中的`_version`中。

外部版本號不僅可以在索引和刪除請求時使用，還可以在_建立_時使用。

例如，建立一篇使用外部版本號為`5`的博文，我們可以這樣操作：


```js
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```

在返回結果中，我們可以發現`_version`是`5`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 5,
  "created":  true
}
```

現在我們更新這個檔案，並指定`version`為`10`：

```js
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```

請求被成功執行並且`version`也變成了`10`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 10,
  "created":  false
}
```
如果你再次執行這個命令，你會得到之前的錯誤提示資訊，因為你所指定的版本號並沒有大於當前Elasticsearch中的版本號。
