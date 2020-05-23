# 第6章　Scrapy Tutorial2

## はじめに

ここでは、架空のオンライン書店のWebページをスクレイピングし、Scrapyでクローラーを作成します。架空のオンライン書店とはいえ、HTMLの構造は一般的なECサイトなどとは変わらないので、非常によい練習になると思います。

* [Books to scrape](http://books.toscrape.com/)

### プロジェクトの作成

まずはプロジェクトを作成します。ここではプロジェクトの名前は「sample\_books」で、クローラーの名前を「books\_spider」としています。今回も前のチュートリアルと同じく「spiders/books\_spider.py」だけ使います。

```text
$ scrapy startproject sample_books
$ cd sample_books
$ scrapy genspider books_spider books.toscrape.com/

➜ tree
.
├── sample_books
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── settings.py
│   └── spiders
│       ├── __init__.py
│       └── books_spider.py
└── scrapy.cfg

```

### HTML構造の調査とクローラーの設計

スクレイピングする前に、スクレイピングするページのHTML構造を確認します。こんページであれば、黒枠のブロックごとに青い線の「名言」、赤い線の「名前」、黄色い線の「タグ」が同じようなレイアウトで表示されています。

