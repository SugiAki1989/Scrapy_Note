# 補章1　Pythonとクローラー

## はじめに

ここでは、Scrapyを使わずにクローラーを作る方法をまとめておきます。Scrapyを使わない場合、基本的にはBeautiful SoupやSeleniumなどのライブラリを使うことになるかと思いますが、ここでは、lxmlライブラリを使用します。[lxmlライブラリ](https://lxml.de/)は、Pythonでxml、htmlを扱うためのライブラリで、Beautiful Soupなどに比べ、より速く、柔軟にhtmlを解析できる特長があるそうです。

今回、クローラーを走らせるサイトは、これまでも使ってきた架空のオンライン書店のサイトです。

* [Books to scrape](http://books.toscrape.com/)

単一のページからURLを取得し、それを拡張して複数ページのURLを取得。そして、詳細ページの情報を抽出し、MySQLに保存するまで、段階を追ってクローラーを作成していきます。ここで紹介する方法以外に、もっと効率のよい方法があると思いますので、参考程度にしていただければと思います。例えばジェネレータとかも使えれば良いんだろうけど、まだ使いこなせてない。

### 単一ページから書籍URLを抽出

まずはTOPページから書籍の詳細ページへのURLを抽出するコードを書いていきます。ファイル名は`books_crawler.py`とします。基本的には、おのおの役割を持つ関数を作って、それを`main()`で実行します。

`fetch()`はURLを引き受けてHTTPリクエストを送り、HTTPレスポンスを受け取って、HTMLを返します。そして、`book_link_extractor()`にHTMLを渡して、各書籍のURLを取得します。

```python
import requests
import lxml.html
import pprint

def main():
    start_url = 'http://books.toscrape.com'
    html = fetch(start_url)
    urls = book_link_extractor(html)
    pprint.pprint(urls)


def fetch(url):
    req = requests.get(url)
    html = lxml.html.fromstring(req.text)
    return html


def book_link_extractor(html):
    urls = []
    for book in html.findall('.//*[@class="product_pod"]/h3/a'):
        url = book.get('href')
        urls.append(url)

    return urls


if __name__ == '__main__':
    main()
```

`books_crawler.py`を実行すると、トップページにある20個の書籍のURLが返されます。

```python
$ python3 books_crawler.py 
['catalogue/a-light-in-the-attic_1000/index.html',
 'catalogue/tipping-the-velvet_999/index.html',
 'catalogue/soumission_998/index.html',
【略】
 'catalogue/mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html',
 'catalogue/libertarianism-for-beginners_982/index.html',
 'catalogue/its-only-the-himalayas_981/index.html']
```

このままでは、50ページあるサイトの各書籍のURLを取得できていないので、改良していきましょう。

### 複数ページから書籍URLを抽出

複数ページから書籍のURLを抽出できるようにループを回して、ページを進めていきます。もちろんURLのページ番号を`for-loop`で回す方法でも良いのですが、ここでは各ページに「次ページ」へのボタンがあるかどうかを判断してクローラーを進めていきます。

そのため、ここでは新たに`nextpage_link_extractor()`と、少し改良した`book_link_extractor()`を定義しています。`nextpage_link_extractor()`は、引き受けたHTMLに「次ページ」へのリンクがあるかを判定します。この関数がページのURLを返す限り、各書籍のURLを抽出する`book_link_extractor()`は実行され続けるように`while-loop`を使用します。

```python
import requests
import lxml.html
import pprint

def main():
    start_url = 'http://books.toscrape.com/catalogue/page-49.html' # テストのためpage49から
    books_urls = book_link_extractor(start_url)
    pprint.pprint(books_urls)


def fetch(url):
    req = requests.get(url)
    html = lxml.html.fromstring(req.text)
    return html


def nextpage_link_extractor(html):
    res = html.find('.//*[@class="next"]/a')

    if res is None:
        return None
    else:
        next_page_url = 'http://books.toscrape.com/catalogue/' + res.get('href')
        return next_page_url


def book_link_extractor(start_url):
    urls = []

    while start_url is not None:
        html = fetch(start_url)

        for book in html.findall('.//*[@class="product_pod"]/h3/a'):
            url = 'http://books.toscrape.com/catalogue/' + book.get('href')
            urls.append(url)

        start_url = nextpage_link_extractor(html)

    return urls


if __name__ == '__main__':
    main()
```

クローラーが想定通りに、動くかどうかテストしてみます。ここでは、49ページと50ページの書籍URLを取得します。合計で40個のリンクが返ってきているので、問題ないですね。

```python
$ python3 books_crawler2.py 
['http://books.toscrape.com/catalogue/on-the-road-duluoz-legend_40/index.html',
 'http://books.toscrape.com/catalogue/old-records-never-die-one-mans-quest-for-his-vinyl-and-his-past_39/index.html',
 'http://books.toscrape.com/catalogue/off-sides-off-1_38/index.html',
 【略】
 'http://books.toscrape.com/catalogue/a-spys-devotion-the-regency-spies-of-london-1_3/index.html',
 'http://books.toscrape.com/catalogue/1st-to-die-womens-murder-club-1_2/index.html',
 'http://books.toscrape.com/catalogue/1000-places-to-see-before-you-die_1/index.html']

```

### 詳細ページから情報を抽出

各ページの書籍URLを抽出できる状態になったので、そのURLを使ってHTTPリクエストを送り、書籍の詳細ページから情報を抽出します。

詳細ページの情報は`scrape()`で行い、ここでは書籍のタイトルを抽出することにします。この関数の説明は不要かと思いますが、ざっくりと説明するとHTTPリクエストを送り、詳細ページからタイトルを抽出する関数です。

```python
import requests
import lxml.html
import pprint

def main():
    start_url = 'http://books.toscrape.com/catalogue/page-49.html'
    books_urls = book_link_extractor(start_url)
    scrape(books_urls)


def fetch(url):
    req = requests.get(url)
    html = lxml.html.fromstring(req.text)
    return html


def nextpage_link_extractor(html):
    res = html.find('.//*[@class="next"]/a')

    if res is None:
        return None
    else:
        next_page_url = 'http://books.toscrape.com/catalogue/' + res.get('href')
        return next_page_url

def book_link_extractor(start_url):
    urls = []

    while start_url is not None:
        html = fetch(start_url)

        for book in html.findall('.//*[@class="product_pod"]/h3/a'):
            url = 'http://books.toscrape.com/catalogue/' + book.get('href')
            urls.append(url)

        start_url = nextpage_link_extractor(html)

    return urls


def scrape(books_urls):
    books_info = []

    for book_url in books_urls:
        html = fetch(book_url)
        title = html.find('.//h1').text
        books_info.append(
            {"title": title}
        )
        pprint.pprint(books_info)

    return books_info


if __name__ == '__main__':
    main()
```

実行すると下記のように各書籍のタイトルが返されます。

```python
 $ python3 books_crawler3.py 
 【略】
 {'title': "A Spy's Devotion (The Regency Spies of London #1)"},
 {'title': "1st to Die (Women's Murder Club #1)"},
 {'title': '1,000 Places to See Before You Die'}]
```

### MySQLに情報を保存

最後はデータベースに保存する`save()`を新たに作って、クローラーを走らせます。まずは、データベース`scraping`にテーブル`books`を作成します。MySQLにインサートする方法は第7章で扱っているので、詳細はそちらを参照ください。

```python
mysql> create database scraping default character set utf8mb4;
Query OK, 1 row affected (0.01 sec)

mysql> CREATE TABLE books(id INT NOT NULL AUTO_INCREMENT, title TEXT, PRIMARY KEY(id));
Query OK, 0 rows affected (0.06 sec)

mysql> select * from books;
Empty set (0.00 sec)
```

`save()`はMySQLへのコネクションをつくり、`scrape()`から得られる書籍のタイトルをインサートします。

```python
import requests
import lxml.html
import pymysql
import datetime

def main():
    print('[START]:',datetime.datetime.now().strftime('%Y年%m月%d日 %H:%M:%S'))
    start_url = 'http://books.toscrape.com/catalogue/page-1.html'
    books_urls = book_link_extractor(start_url)
    books = scrape(books_urls)
    save(books)
    print('[END]:',datetime.datetime.now().strftime('%Y年%m月%d日 %H:%M:%S'))

def fetch(url):
    req = requests.get(url)
    html = lxml.html.fromstring(req.text)
    return html


def nextpage_link_extractor(html):
    res = html.find('.//*[@class="next"]/a')

    if res is None:
        return None
    else:
        next_page_url = 'http://books.toscrape.com/catalogue/' + res.get('href')
        return next_page_url

def book_link_extractor(start_url):
    urls = []

    while start_url is not None:
        html = fetch(start_url)

        for book in html.findall('.//*[@class="product_pod"]/h3/a'):
            url = 'http://books.toscrape.com/catalogue/' + book.get('href')
            urls.append(url)

        start_url = nextpage_link_extractor(html)

    return urls


def scrape(books_urls):
    books_info = []

    for book_url in books_urls:
        html = fetch(book_url)
        title = html.find('.//h1').text
        books_info.append(
            {"title": title}
        )

    return books_info

def save(books):
    con = pymysql.connect(host="localhost",
                          db="scraping",
                          user="****",
                          passwd="****",
                          charset="utf8")
    cur = con.cursor()

    for book in books:
        query = 'INSERT INTO books (title) VALUES (%s);'
        title = book.get("title")
        record = (title)
        print('[Insert]:{}'.format(record))

        cur.execute(query, record)

    con.commit()
    con.close()


if __name__ == '__main__':
    main()
```

スタートするページを1ページ目に戻し、最後の50ページまでクローラーを走らせます。これを実行すると、下記のようにMySQLのテーブルに1000冊分の書籍のタイトル情報がインサートされます。

```python
$ python3 books_crawler.py 
[START]: 2020年06月13日 17:46:49
[Insert]:A Light in the Attic
[Insert]:Tipping the Velvet
[Insert]:Soumission
【略】
[Insert]:A Spy's Devotion (The Regency Spies of London #1)
[Insert]:1st to Die (Women's Murder Club #1)
[Insert]:1,000 Places to See Before You Die
[END]: 2020年06月13日 17:58:14
```

MySQLのテーブルも確認しておきます。1000件全ての書籍タイトルが取得できています。

```python
mysql> select * from books limit 10;
+----+------------------------------------------------------------------------------------------------+
| id | title                                                                                          |
+----+------------------------------------------------------------------------------------------------+
|  1 | A Light in the Attic                                                                           |
|  2 | Tipping the Velvet                                                                             |
|  3 | Soumission                                                                                     |
|  4 | Sharp Objects                                                                                  |
|  5 | Sapiens: A Brief History of Humankind                                                          |
|  6 | The Requiem Red                                                                                |
|  7 | The Dirty Little Secrets of Getting Your Dream Job                                             |
|  8 | The Coming Woman: A Novel Based on the Life of the Infamous Feminist, Victoria Woodhull        |
|  9 | The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics |
| 10 | The Black Maria                                                                                |
+----+------------------------------------------------------------------------------------------------+
10 rows in set (0.00 sec)

mysql> select count(1) from books;
+----------+
| count(1) |
+----------+
|     1000 |
+----------+
1 row in set (0.00 sec)

```

ここまでScrapyを使わずクローラーを作ってみましたが、ここからさらに、ScrapyのSettingsの内容を実装するとなると、考えるだけでも大変ですし、Scrapyのありがたみがわかりますね。



