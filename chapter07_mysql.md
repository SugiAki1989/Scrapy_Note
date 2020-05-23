# 第7章　ScrapyとMySQL

## はじめに

ここではScrapyでスクレイピングした値をMySQLのテーブルに格納する方法についてまとめていきます。MongoDBのようなNoSQLデータベースが使われることも多いようですが、ここではMySQLを扱います。

### プロジェクトの作成

対象にするサイトは、チュートリアル2で使った架空のオンライン書店のサイトです。チュートリアル2のコードをベースにMySQLに連携できるように書き換えます。ではプロジェクトを作成します。

* [Books to scrape](http://books.toscrape.com/)

```text
$ scrapy startproject sample_books_mysql
$ cd sample_books_mysql
$ scrapy genspider books_spider_mysql books.toscrape.com
```

### アイテムの定義MySQL

Itemを定義していきます。これまで使ってきませんでしたが、Itemを定義する必要があります。Itemはスクレイピングしたデータを格納しておくためのオブジェクトで、ここに格納して、MySQLに保存するためのパイプラインにデータを流します。

また、Itemを定義することで、タイポによるカラム名の間違いがあればエラーを返すようになります。指定の仕方は「名前=scrapy.Field\(\)」で、タイトル、価格、URLを保存するようにします。

```text
import scrapy

class BooksMysqlItem(scrapy.Item):
        title = scrapy.Field()
        price = scrapy.Field()
        detail_page_url = scrapy.Field()
```

### クローラーの設計

Itemにあわせて、不要な項目を削除しています。スクレイピングした情報はItemクラスのコンストラクタ`BooksMysqlItem`で初期化した後に、キーを指定して渡します。その他はこれまでと変わりません。

```text
from scrapy import Spider
from scrapy.http import Request
from sample_books_mysql.items import BooksMysqlItem　#忘れずにインポートする

class BooksSpiderMysqlSpider(Spider):
    name = 'books_spider_mysql'
    allowed_domains = ['books.toscrape.com']
    start_urls = ['http://books.toscrape.com']

    def parse(self, response):
        books = response.xpath('.//*[@class="product_pod"]')
        for book in books:
            item = BooksMysqlItem()

            item["title"] = book.xpath('.//h3/a/@title').get()
            item["price"] = book.xpath('.//*[@class="price_color"]/text()').get()
            img_url = book.xpath('.//*[@class="image_container"]/a/@href').get()
            item["detail_page_url"] = "http://books.toscrape.com/" + img_url

            yield item


        # If there is a next button on this page, move the crawler
        next_page_url = response.xpath('//a[text()="next"]/@href').get()
        abs_next_page_url = response.urljoin(next_page_url)
        if abs_next_page_url is not None:
            yield Request(abs_next_page_url, callback=self.parse)
```

### パイプラインの設計

Itemに保存されているデータをMySQLにインサートするためのパイプラインを記述していきます。

```text
import pymysql


class BooksMysqlPipeline:
    def open_spider(self, spider):
        self.connection = pymysql.connect(
            host="localhost",
            user="****", # DBにあわせて変更
            passwd="****", # DBにあわせて変更
            database="book_online",
            charset="utf8mb4"
        )
        self.cursor = self.connection.cursor()

    def process_item(self, item, spider):
        # duplication check
        check_title_id = item["title"]
        print(check_title_id)
        find_qry = "SELECT `title` FROM `books` WHERE `title` = %s"
        is_done = self.cursor.execute(find_qry, check_title_id)

        # if already a record exists in database, return 1
        if is_done == 0:
            insert_qry = "INSERT INTO `books` (`title`, `price`, `detail_page_url`) VALUES (%s, %s, %s)"
            self.cursor.execute(insert_qry, (item["title"], item["price"], item["detail_page_url"]))
            self.connection.commit()
        else:
            pass

        return item

    def close_spider(self, spider):
        self.connection.close()
```

![Title duplication](.gitbook/assets/sukurnshotto-2020-05-24-45010png.png)

### 環境設定

ああああああ

```text
# Configure item pipelines
# See https://docs.scrapy.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
    'sample_books_mysql.pipelines.BooksMysqlPipeline': 800,
}
```

### テーブルの定義\(MySQL\)

ああああああ

```text
mysql> CREATE DATABASE book_online;
mysql> USE book_online;
mysql> CREATE TABLE books(id INT NOT NULL AUTO_INCREMENT,
                          title TEXT,
                          price VARCHAR(50),
                          detail_page_url TEXT,
                          PRIMARY KEY(id));
```

### クローラーを実行する

ああああ

```text
$ scrapy crawl books_spider_mysql
```

MySQLの中身を確認しておきます。想定通り999件のデータが格納されていることがわかります。

```text
mysql> select count(1) from books;
+----------+
| count(1) |
+----------+
|      999 |
+----------+
```

