# 檢索文件

現在，我們已經在 Elasticsearch 中儲存了一些資料，我們可以開始根據這個項目的需求進行工作了。第一個需求就是要能搜尋每一個員工的資料。

對於 Elasticsearch 來說，這是非常簡單的。我們只需要執行一次 HTTP GET 請求，然後指出文件的**地址**，也就是索引、類型以及 ID 即可。通過這三個部分，我們就可以得到原始的 JSON 文件：

```js
GET /megacorp/employee/1
```

返回的內容包含了這個文件的後設資料資訊，而 John Smith 的原始 JSON 文件也在 `_source` 欄位中出現了：

```js
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

****

我們通過將HTTP後的請求方式由 `PUT` 改變為 `GET` 來獲取文件，同理，我們也可以將其更換為 `DELETE` 來刪除這個文件，`HEAD` 是用來查詢這個文件是否存在的。如果你想替換一個已經存在的文件，你只需要使用 `PUT` 再次發出請求即可。

****

# 簡易搜尋

`GET` 命令真的相當簡單，你只需要告訴它你要什麼即可。接下來，我們來試一下稍微複雜一點的搜尋。

我們首先要完成一個最簡單的搜尋命令來搜尋全部員工：

```js
GET /megacorp/employee/_search
```

你可以發現我們正在使用 `megacorp` 索引，`employee` 類型，但是我們我們並沒有指定文件的ID，我們現在使用的是 `_search` 埠。你可以再返回的 `hits` 中發現我們錄入的三個文件。搜尋會預設返回最前的10個數值。


```js
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

注意：反饋值中不僅會告訴你匹配到哪些文件，同時也會把這個文件都會包含到其中：我們需要搜尋的使用者的所有資訊。

接下來，我們將要嘗試著實現搜尋一下哪些員工的姓氏中包含 `Smith`。為了實現這個，我們需要使用一種**輕量**的搜尋方法。這種方法經常被稱做 _查詢字元串(query string)_ 搜尋，因為我們通過URL來傳遞查詢的關鍵字：

```js
GET /megacorp/employee/_search?q=last_name:Smith
```

我們依舊使用 `_search` 埠，然後可以將參數傳入給 `q=`。這樣我們就可以得到姓Smith的結果：

```js
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

# 使用Query DSL搜尋

查詢字元串是通過命令語句完成 _點對點(ad hoc)_ 的搜尋，但是這也有它的侷限性（可參閱《搜尋侷限性》章節）。Elasticsearch 提供了更加豐富靈活的查詢語言，它被稱作 _Query DSL_，通過它你可以完成更加複雜、強大的搜尋任務。

DSL (_Domain Specific Language_ 領域特定語言) 需要使用 JSON 作為主體，我們還可以這樣查詢姓 Smith 的員工：

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

這個請求會返回同樣的結果。你會發現我們在這裡沒有使用 _查詢字元串_，而是使用了一個由 JSON 構成的請求體，其中使用了 `match` 查詢法，隨後我們還將會學習到其他的查詢類型。


# 更加複雜的搜尋

接下來，我們再提高一點兒搜尋的難度。我們依舊要尋找出姓 Smith 的員工，但是我們還將新增一個年齡大於30歲的限定條件。我們的查詢語句將會有一些細微的調整來以識別結構化搜尋的限定條件 _filter（過濾器）_:

```js
GET /megacorp/employee/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 } <1>
                }
            },
            "query" : {
                "match" : {
                    "last_name" : "Smith" <2>
                }
            }
        }
    }
}
```
1. 這一部分的語句是 `range` _filter_ ，它可以查詢所有超過30歲的資料 -- `gt` 代表 **greater than （大於）**。

2. 這一部分我們前一個操作的 `match` _query_ 是一樣的

先不要被這麼多的語句嚇到，我們將會在之後帶你逐漸瞭解他們的用法。你現在只需要知道我們添加了一個 _filter_，可以在 `match` 的搜尋基礎上再來實現區間搜尋。現在，我們的只會顯示32歲的名為**Jane Smith**的員工了：

```js
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

# 全文搜尋

上面的搜尋都很簡單：名字搜尋、通過年齡過濾。接下來我們來學習一下更加複雜的搜尋，全文搜尋——一項在傳統資料庫很難實現的功能。
我們將會搜尋所有喜歡 **rock climbing** 的員工:

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```
你會發現我們同樣使用了 `match` 查詢來搜尋 `about` 欄位中的 **rock climbing**。我們會得到兩個匹配的文件：

```js
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, <1>
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, <1>
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```
1. 相關評分

通常情況下，Elasticsearch 會通過相關性來排列順序，第一個結果中，John Smith 的 `about` 欄位中明確地寫到 **rock climbing**。而在 Jane Smith 的 `about` 欄位中，提及到了 **rock**，但是並沒有提及到 **climbing**，所以後者的 `_score` 就要比前者的低。

這個例子很好地解釋了 Elasticsearch 是如何執行全文搜尋的。對於 Elasticsearch 來說，相關性的概念是很重要的，而這也是它與傳統資料庫在返回匹配資料時最大的不同之處。


# 段落搜尋

能夠找出每個欄位中的獨立單詞固然很好，但是有的時候你可能還需要去匹配精確的短語或者 _段落_。例如，我們只需要查詢到 `about` 欄位只包含 **rock climbing** 的短語的員工。

為了實現這個效果，我們將對 `match` 查詢變為 `match_phrase` 查詢：

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

這樣，系統會沒有異議地返回 John Smith 的文件：

```js
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         }
      ]
   }
}
```

# 高亮我們的搜尋

很多程式希望能在搜尋結果中 _高亮_ 匹配到的關鍵字來告訴使用者這個文件是 _如何_ 匹配他們的搜尋的。在 Elasticsearch 中找到高亮片段是非常容易的。

讓我們回到之前的查詢，但是新增一個 `highlight` 參數：

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```
當我們執行這個查詢後，相同的命中結果會被返回，但是我們會得到一個新的名叫 `highlight` 的部分。在這裡包含了 `about` 欄位中的匹配單詞，並且會被 `<em></em>` HTML字元包裹住：

```js
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" <1>
               ]
            }
         }
      ]
   }
}
```

1. 在原有文字中高亮關鍵字。
