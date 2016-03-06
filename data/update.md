# 更新整個檔案

在Documents中的檔案是不可改變的。所以如果我們需要改變已經存在的檔案，我們可以使用《索引》中提到的`index`API來_重新索引_或者替換掉它：

```js
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```
在反饋中，我們可以發現Elasticsearch已經將`_version`數值增加了：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 2,
  "created":   false <1>
}
```
1. `created`被標記為 `false`是因為在同索引、同類型下已經存在同ID的檔案。

在內部，Elasticsearch已經將舊檔案標記為刪除並且添加了新的檔案。舊的檔案並不會立即消失，但是你也無法訪問他。Elasticsearch會在你繼續新增更多資料的時候在後臺清理已經刪除的檔案。

在本章的後面，我們將會在《局部更新》中介紹最新更新的API。這個API允許你修改局部，但是原理和下方的完全一樣：

1. 從舊的檔案中檢索JSON
2. 修改它
3. 刪除修的檔案
4. 索引一個新的檔案

唯一不同的是，使用了`update`API你就不需要使用`get`然後再操作`index`請求了。
