# Elasticsearch 權威指南

## 項目信息

#### [在線閱讀](http://learnes.net)
國外自動指向 [GITBOOK 項目](http://fuxiaopang.gitbooks.io/LearnElasticSearch) | 國內用戶自動指向 [阿里雲](http://learnes.net)

#### [GITHUB 倉庫](https://github.com/GavinFoo/elasticsearch-definitive-guide)


## 譯者前言
譯者現在的工作項目中需要用到 Elasticsearch，但是在網絡中找了很多的相關的內容都很不完善，中文的文檔更是寥寥無幾，所以我決定邊研究邊翻譯一下官方推出的權威手冊。在這裏要先感謝原作者們！如果各位在這裏發現了錯誤之處，請大家在 Issue 中提出或者 PR [這個項目](https://github.com/GavinFoo/elasticsearch-definitive-guide/)。

> ##### 如果你喜歡這個翻譯項目可以[點擊Star一下](https://github.com/GavinFoo/elasticsearch-definitive-guide/)，您的支持是我們最大的動力。

****
原作名字：elasticsearch - the definitive guide

原作作者：clinton gormley，zachary tong

譯者：Gavin Foo <fuxiaopang@gmail.com>


## 前言

這本書還在不斷地添加內容中，我們會陸陸續續地在這裏添加新的章節。這本書中的內容針對的是 Elasticsearch 的最新版本。

歡迎反饋 – 如果這裏出現了錯誤，或者你有什麼建議可以到我們 GitHub 項目中 [新建一個issue](https://github.com/GavinFoo/elasticsearch-definitive-guide/issues)。
****

這個世界已經被數據淹沒。我們創造的系統所產生的數據可以瞬間輕而易舉地將我們壓垮，現有的科技一直致力於如何存儲數據，並能將擁有大量信息的數據倉庫結構化。而當你準備開始從大量的數據中得出結論做決策的時候，美好的一天就要被毀滅了……

Elasticsearch 是一個分佈式可擴展的實時搜索和分析引擎。它能幫助你搜索、分析和瀏覽數據，而往往大家並沒有在某個項目一開始就預料到需要這些功能。Elasticsearch 之所以出現就是爲了重新賦予硬盤中看似無用的原始數據新的活力。

無論你是需要全文搜索、結構化數據的實時統計，還是兩者的結合，這本指南都會幫助你瞭解其中最基本的概念，從最基本的操作開始學習 Elasticsearch。之後，我們還會逐漸開始探索更加複雜的搜索技術，你可以根據自身的學習的步伐。

Elasticsearch 並不是單純的全文搜索這麼簡單。我們將向你介紹講解結構化搜索、統計、查詢過濾、地理定位、自動完成以及_你是不是要查找_的提示。我們還將探討如何給數據建模能提升 Elasticsearch 的性能，以及在生產環境中如何配置、監視你的集羣。
 
