# 第11章 Scrapyを使わずクローラーを作る

## はじめに

ここでは、Scrapyを使わずにクローラーを作る方法をまとめておきます。Scrapyを使わない場合、基本的にはBeautiful SoupやSeleniumなどのライブラリを使うことになるかと思いますが、ここでは、lxmlライブラリを使用します。[lxmlライブラリ](https://lxml.de/)は、Pythonでxml、htmlを扱うためのライブラリで、Beautiful Soupなどに比べ、より速く、柔軟にhtmlを解析できる特長があるそうです。

今回、クローラーを走らせるサイトは、これまでも使ってきた架空のオンライン書店のサイトです。

* [Books to scrape](http://books.toscrape.com/)

### 単一ページから書籍URLを抽出

段階を追ってクローラーを作成していきます。まずはTOPページから書籍の詳細ページへのURLを抽出するコードを書いていきます。ファイル名は`books_crawler.py`とします。

`fetch()`はURLを引き受けて、HTMLを返します。そして、`book_link_extractor()`が各書籍のURLを返します。

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

このままでは、50ページあるサイトの各書籍のURLを取得できていないので、改良していきます。

### 複数ページから書籍URLを抽出

複数ページから書籍のURLを抽出できるようにループを回して、ページを進めていきます。

ここでは新たに`nextpage_link_extractor()`と`book_link_extractor()`を定義しています。`nextpage_link_extractor()`は、引き受けたHTMLにNextページへのリンクがあるかを判定します。この関数がページのURLを返す限り、各書籍のURLを抽出する`book_link_extractor()`は実行され続けます。

```python
import requests
import lxml.html


def main():
    start_url = 'http://books.toscrape.com/catalogue/page-49.html'
    books_urls = book_link_extractor(start_url)
    print(books_urls)


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





