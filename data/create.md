# 建立一個檔案

當我們索引一個檔案時，如何確定我們是建立了一個新的檔案還是覆蓋了一個已經存在的檔案呢？


請牢記`_index`,`_type`以及`_id`組成了唯一的檔案標記，所以為了確定我們建立的是全新的內容，最簡單的方法就是使用`POST`方法，讓Elasticsearch自動建立不同的`_id`：

```js
POST /website/blog/
{ ... }
```

然而，我們可能已經決定好了`_id`，所以需要告訴Elasticsearch只有當`_index`，`_type`以及`_id`這3個屬性全部相同的檔案不存在時才接受我們的請求。實現這個目的有兩種方法，他們實質上是一樣的，你可以選擇你認為方便的那種：

第一種是在查詢中新增`op_type`參數：

```js
PUT /website/blog/123?op_type=create
{ ... }
```

或者在請求最後新增 `/_create`:

```js
PUT /website/blog/123/_create
{ ... }
```

如果成功建立了新的檔案，Elasticsearch將會返回常見的後設資料以及`201 Created`的HTTP反饋碼。

而如果存在同名檔案，Elasticsearch將會返回一個`409 Conflict`的HTTP反饋碼，以及如下方的錯誤資訊：

```js
{
  "error" : "DocumentAlreadyExistsException[[website][4] [blog][123]:
             document already exists]",
  "status" : 409
}
```

