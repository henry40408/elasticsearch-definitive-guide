# 搜尋檔案

要從Elasticsearch中獲取檔案，我們需要使用同樣的`_index`，`_type`以及 `_id`但是不同的HTTP變數`GET`：
```js
GET /website/blog/123?pretty
```
返回結果包含了之前提到的內容，以及一個新的欄位`_source`，它包含我們在最初建立索引時的原始JSON檔案。

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
      "date":  "2014/01/01"
  }
}
```

****
>#### `pretty`

在任意的查詢字元串中新增`pretty`參數，類似上面的請求，Elasticsearch就可以得到_優美列印_的更加易於識別的JSON結果。`_source`欄位不會執行優美列印，它的樣子取決於我們錄入的樣子。

****

GET請求的返回結果中包含`{"found": true}`。這意味著這篇檔案確實被找到了。如果我們請求了一個不存在的檔案，我們依然會得到JSON反饋，只是`found`的值會變為`false`。

同樣，HTTP返回碼也會由`'200 OK'`變為`'404 Not Found'`。我們可以在`curl`後新增`-i`，這樣你就能得到反饋標頭檔案：

```js
curl -i -XGET /website/blog/124?pretty
```

反饋結果就會是這個樣子：

```js
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=UTF-8
Content-Length: 83

{
  "_index" : "website",
  "_type" :  "blog",
  "_id" :    "124",
  "found" :  false
}
```

### 檢索檔案中的一部分

通常，`GET`請求會將整個檔案放入`_source`欄位中一併返回。但是可能你只需要`title`欄位。你可以使用`_source`得到指定欄位。如果需要多個欄位你可以使用逗號分隔：

```js
GET /website/blog/123?_source=title,text
```
現在`_source`欄位中就只會顯示你指定的欄位：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "exists" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```
或者你只想得到`_source`欄位而不要其他的後設資料，你可以這樣請求：

```js
GET /website/blog/123/_source
```
這樣結果就只返回:

```js
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```
