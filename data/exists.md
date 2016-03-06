# 檢查檔案是否存在

如果確實想檢查一下檔案是否存在，你可以試用`HEAD`來替代`GET`方法，這樣就是會返回HTTP標頭檔案：

```js
curl -i -XHEAD /website/blog/123
```
如果檔案存在，Elasticsearch將會返回`200 OK`的狀態碼：

```js
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```
如果不存在將會返回`404 Not Found`狀態碼：

```js
curl -i -XHEAD /website/blog/124
```

```js
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

當然，這個反饋只代表了你查詢的那一刻檔案不存在，但是不代表幾毫秒後它不存在，很可能與此同時，另一個程式正在建立檔案。
