# 第4章　Scrapy Tutorial

## はじめに

ここでは、Scrapyのドキュメントに載っている[チュートリアル](https://docs.scrapy.org/en/latest/intro/tutorial.html)を参考に、Scrapyでクローラーを作成する。チュートリアルを完全に再現するわけではなく、チュートリアルを題材にしながら寄り道しながら進めていきます。チュートリアルを実行されたい方はドキュメントを参照ください。スクレイピングの対象にするサイトは、チュートリアルと同じで下記の有名人の名言サイトです。

* [quotes.toscrape.com](http://quotes.toscrape.com/) 

### プロジェクトの作成

まずはプロジェクトを作成します。ここではプロジェクトの名前は「sample\_quotes」で、クローラーの名前を「quotes\_spider」としています。プロジェクトを作ったら、settings.pyの中身を礼儀が正しいように書き換えるのは忘れずに行います。\*\*今回は「spiders/quotes\_spider.py」と「settings.py」だけ使います。これだけでも動かせるので。

```text
$ scrapy startproject sample_quotes
$ cd sample_quotes
$ scrapy genspider quotes_spider quotes.toscrape.com

$ tree
.
├── sample_quotes
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── settings.py **これ**
│   └── spiders
│       ├── __init__.py
│       └── quotes_spider.py **これ**
└── scrapy.cfg
```

### HTML構造の調査と設計

スクレイピングする前に、スクレイピングするページのHTML構造を確認します。こんページであれば、黒枠のブロックごとに青い線の「名言」、赤い線の「名前」、黄色い線の「タグ」が同じようなレイアウトで表示されています。

![Image of HTML page](.gitbook/assets/img_792be4aea2a9-1.jpeg)

同じようなレイアウトで表示されているということは、HTMLの書き方も同じようになっているということです。実際にブラウザの検証機能で確認すると、同じようなHTML構造になっています。

![Structure of HTML](.gitbook/assets/sukurnshotto-2020-05-23-214926png.png)

これであれば、「ブロックを取得→詳細をスクレイピング→次のブロック→詳細をスクレイピング」ということを繰り返せば取得できそうです。実際に値がとれるか、Scrapy Shellを利用して確認します。

### Scrapy Shell

URLを渡して、Scrapy Shellでレスポンスを受け取ります。

```text
$ scrapy shell "http://quotes.toscrape.com/"

In [1]: response                                                                                                                        
Out[1]: <200 http://quotes.toscrape.com/>
```

まずはブロックの部分を取得します。ブロックの部分は、classがすべて「quote」なので、それをxpathに渡します。cssセレクタで指定することも可能ですが、ここでは、xpathにしぼります。xpathセレクターの使い方は、ドキュメントの[Selectors](https://docs.scrapy.org/en/latest/topics/selectors.html)を参照してください。

```text
In [2]: response.xpath('//*[@class="quote"]')                                                                                           
Out[2]: 
[<Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath='//*[@class="quote"]' data='<div class="quote" itemscope itemtype...'>]
```

数を数えてみると、10が返ってきます。ページの名言ブロックと一致しているのでOKですね。次は詳細部分をスクレイピングします。

```text
In [3]: len(response.xpath('//*[@class="quote"]')   )                                                                                   
Out[3]: 10
```

まずは名言を取得します。まずは、1番上のブロックをquoteに格納します。そして、quoteを使って、名言部分を先ほどと同じようにxpathで指定します。名言は、classがすべて「text」なので、それをxpathに渡します。

```text
In [4]: quotes = response.xpath('//*[@class="quote"]')                                                                                  
In [5]: quote = quotes[0]                                                                                                               

In [6]: quote.xpath('.//*[@class="text"]/text()').get()                                                                                 
Out[6]: '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
```

次は、同じ要領で「名前」と「タグ」をスクレイピングします。

```text
In [7]: quote.xpath('.//*[@class="author"]/text()').get()                                                                               
Out[7]: 'Albert Einstein'

In [8]: quote.xpath('.//*[@class="tag"]/text()').getall()                                                                               
Out[8]: ['change', 'deep-thoughts', 'thinking', 'world']
```

これでスクレイピングしたい情報のコードが書けました。これをブロックごとに繰り返したいので、下記のようにfor-loopで書き直し、取得した値はyieldで出力します。

```text
quotes = response.xpath('//*[@class="quote"]')

for quote in quotes:
    text = quote.xpath('.//*[@class="text"]/text()').get()
    author = quote.xpath('.//*[@class="author"]/text()').get()
    tags = quote.xpath('.//*[@class="tag"]/text()').getall()

    yield {
        "text": text,
        "author": author,
        "tags": tags
    }
```

このページ1枚であればこれで良いのですが、全部で10ページあるので、次のページにクローラーを動かせるように、コードを追加します。次のページへの判断は「Next」があるかどうかで判断させます。10ページを見ると「Next」がないので、これ以上はクローラーは進まなくなります。

「Next」があるかどうかを調べるためにScrapy Shellで確認します。このボタンはclassが「next」でa要素に次のページへのURLがついているので、それがあるかないかで判定します。

```text
In [9]: response.xpath('//*[@class="next"]/a/@href').get()                                                                              
Out[9]: '/page/2/'
```

URLがあるかどうかで判定するのはこれで良いのですが、クローラーがこの相対パスのURLでは移動できません。なので、完全パスにURLをurljoinで修正します。

```text

```

### クローラーの実行

### ログ情報







