# Elasticsearch 權威指南 繁體中文版

透過 [OpenCC](https://github.com/BYVoid/OpenCC) 自動翻譯。

#### [線上閱讀繁體中文版](https://www.gitbook.com/book/myceto/elasticsearch-definitive-guide)

#### [繁體中文版 GitHub](https://github.com/myceto/elasticsearch-definitive-guide)

## 項目資訊

#### [線上閱讀](http://learnes.net)
國外自動指向 [GITBOOK 項目](http://fuxiaopang.gitbooks.io/LearnElasticSearch) | 國內使用者自動指向 [阿里雲](http://learnes.net)

#### [GITHUB 倉庫](https://github.com/GavinFoo/elasticsearch-definitive-guide)

## 譯者前言
譯者現在的工作項目中需要用到 Elasticsearch，但是在網路中找了很多的相關的內容都很不完善，中文的檔案更是寥寥無幾，所以我決定邊研究邊翻譯一下官方推出的權威手冊。在這裡要先感謝原作者們！如果各位在這裡發現了錯誤之處，請大家在 Issue 中提出或者 PR [這個項目](https://github.com/GavinFoo/elasticsearch-definitive-guide/)。

> ##### 如果你喜歡這個翻譯項目可以[點選Star一下](https://github.com/GavinFoo/elasticsearch-definitive-guide/)，您的支援是我們最大的動力。

****
原作名字：elasticsearch - the definitive guide

原作作者：clinton gormley，zachary tong

譯者：Gavin Foo <fuxiaopang@gmail.com>

## 前言

這本書還在不斷地新增內容中，我們會陸陸續續地在這裡新增新的章節。這本書中的內容針對的是 Elasticsearch 的最新版本。

歡迎反饋 – 如果這裡出現了錯誤，或者你有什麼建議可以到我們 GitHub 項目中 [新建一個issue](https://github.com/GavinFoo/elasticsearch-definitive-guide/issues)。

****

這個世界已經被資料淹沒。我們創造的系統所產生的資料可以瞬間輕而易舉地將我們壓垮，現有的科技一直致力於如何儲存資料，並能將擁有大量資訊的資料倉儲結構化。而當你準備開始從大量的資料中得出結論做決策的時候，美好的一天就要被毀滅了……

Elasticsearch 是一個分散式可擴充套件的實時搜尋和分析引擎。它能幫助你搜索、分析和瀏覽資料，而往往大家並沒有在某個項目一開始就預料到需要這些功能。Elasticsearch 之所以出現就是為了重新賦予硬碟中看似無用的原始資料新的活力。

無論你是需要全文搜尋、結構化資料的實時統計，還是兩者的結合，這本指南都會幫助你瞭解其中最基本的概念，從最基本的操作開始學習 Elasticsearch。之後，我們還會逐漸開始探索更加複雜的搜尋技術，你可以根據自身的學習的步伐。

Elasticsearch 並不是單純的全文搜尋這麼簡單。我們將向你介紹講解結構化搜尋、統計、查詢過濾、地理定位、自動完成以及_你是不是要查詢_的提示。我們還將探討如何給資料建模能提升 Elasticsearch 的效能，以及在生產環境中如何配置、監視你的叢集。
 
