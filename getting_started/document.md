# 面向文件

程式中的物件很少是單純的鍵值與數值的列表。更多的時候它擁有一個複雜的結構，比如包含了日期、地理位置、物件、陣列等。

遲早你會把這些物件儲存在資料庫中。你會試圖將這些豐富而又龐大的資料都放到一個由行與列組成的關聯式資料庫中，然後你不得不根據每個欄位的格式來調整資料，然後每次重建它你都要檢索一遍資料。

Elasticsearch 是 _面向文件型資料庫_，這意味著它儲存的是整個物件或者 _文件_，它不但會儲存它們，還會為他們建立**索引**，這樣你就可以搜尋他們了。你可以在 Elasticsearch 中索引、搜尋、排序和過濾這些文件。不需要成行成列的資料。這將會是完全不同的一種面對資料的思考方式，這也是為什麼 Elasticsearch 可以執行復雜的全文搜尋的原因。


# JSON

Elasticsearch使用 [_JSON_](http://baike.baidu.com/view/136475.htm?fr=aladdin) (或稱作JavaScript
Object Notation ) 作為文件序列化的格式。JSON 已經被大多數語言支援，也成為 NoSQL 領域的一個標準格式。它簡單、簡潔、易於閱讀。

把這個 JSON 想象成一個使用者物件:

```js
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "about": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01",
}
```

雖然 `user` 這個物件非常複雜，但是它的結構和含義都被保留到 JSON 中了。在 Elasticsearch 中，將物件轉換為 JSON 並作為索引要比在表結構中做相同的事情簡單多了。

***
>###將你的資料轉換為 JSON

幾乎所有的語言都有將任意資料轉換、機構化成 JSON，或者將物件轉換為JSON的模組。檢視 `serialization` 以及 `marshalling` 兩個 JSON 模組。[The official Elasticsearch clients](http://www.elasticsearch.org/guide) 也可以幫你自動結構化 JSON。

***
