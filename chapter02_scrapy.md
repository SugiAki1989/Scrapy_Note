# 第2章　Scrapyとは

## はじめに

ここではScrapyの基本的な使い方をまとめておきます。加えて、各ファイルの役割、アーキテクチャについても簡単におさらいしておきます。

### Scrapyとは

Scrapyとは、起点となるURLを決定し、特定の条件のもとでクローラーを移動させ、Webサイトをクロールさせることで、構造化されたデータを抽出するためのPythonのフレームワークです。wget、正規表現、BeautifulSoup、Seleniumなどを組み合わせることで、クローラーの作成やWebスクレイピングはできますが、Scrapyを使うことで、スクレイピングの作業に集中してクローラーを作ることが可能です。

例えば、行儀よくスクレイピングするためのオプション\(後述\)をはじめ、データを連携して処理するパイプライン機能など、スクレイピングで利用したい様々な機能が、デフォルトで利用可能なため、1からクローラーを作り、様々な機能を新しく実装する必要がない点が特徴かと思います。

もちろん、フレームワークなので、それで対応できないものについては、1から自前のクローラーを作る必要があるので、必ずしもScrapyが優れているわけでもないと思いますが、非常に便利なフレームワークなので、ここではScrapyに焦点を当てて説明していきます。

### Scrapyのプロジェクト

まずはScrapyをインストールしていきます。ご自身の環境に合わせてpipやcondaでインストールしてください。仮想環境が必要であれば、仮想環境を作ってからインストールしてください。ここでは、pipでインストールします。

```text
# conda install -c conda-forge scrapy
$ pip install Scrapy

$ scrapy -V
Scrapy 2.0.1
```

Scrapyのプロジェクトファイルは、下記のコマンドで生成できます。ここでは、"sample\_pj"という名前のプロジェクトフォルダを作成しています。 

```text
$ scrapy startproject sample_pj
New Scrapy project 'sample_pj', using template directory '/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/scrapy/templates/project', created in:
    /Users/aki/Desktop/sample_pj

You can start your first spider with:
    cd sample_pj
    scrapy genspider example example.com
```

scrapy genspiderコマンドでクローラーの名前とスクレイピングしたいページのURLを指定します。指示にしたがって、scrapy genspiderコマンドでクローラーの名前"sample"と対象のサイトのURLとして"example.com"を指定します。

```text
$ cd sample_pj
~/Desktop/sample_pj 

$ scrapy genspider sample example.com
Created spider 'sample' using template 'basic' in module:
  sample_pj.spiders.sample
```

これでプロジェクトフォルダの作成は完了です。ここからスクリプトを記述して、スクレイピングを行っていきます。

### Scrapyの.pyファイル

さきほどのコマンドを実行すると、下記のようなフォルダが生成されます。各.pyファイルの役割をまとめておきます。

```text
$ tree ~/Desktop/sample_pj/
/Users/aki/Desktop/sample_pj/
├── sample_pj
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── settings.py
│   └── spiders
│       ├── __init__.py
│       └── sample.py
└── scrapy.cfg
```

Scrapyは「収集するデータを定義する」→「抽出する場所と範囲を決定する」→「抽出するための方法を記述する」→「抽出したアイテムを処理する方法を決定する」→「クローラーを起動し、スクレイピングを実行する」という順番で役割を分けて、コードを記述できるように、複数のファイルが用意されています。

#### items.py

出力されるデータの形式を定義し、構造化されたデータを格納するために使用されます。矛盾したデータや、typoなどによる意図していないデータの混入を防ぐために利用できます。絶対に必要というわけではありませんが、使用することでクローラーを安定して運用できます。

出力データの形式を定義するために、Scrapyは Itemクラスを提供しています。Itemオブジェクトは、スクレイピングされたデータを収集するためのコンテナなようなもので、定義していない値などは弾かれてしまいます。

下記のように必要なアイテムを宣言します。

```text
class Product(scrapy.Item):
    X = scrapy.Field()
    Y = scrapy.Field()
    Z = scrapy.Field()
```

#### Spiders/spider.py

scrapy genspiderコマンドで生成したSpidersディレクトリの中のsample.pyに、クローラーの動きやスクレイピングのコードなど、スクレイピング方法を定義するクラスです。このような感じで記述します。

QuotesSpiderは、allowed\_domainsで許可されたドメイン内において、start\_urlsで指定されたURLを起点に、QuotesSpiderは動き始めます。 URLのRequestを生成するstart\_requestsメソッドと、リクエストのコールバック関数としてparseメソッドを呼び出します。コールバック関数は、レスポンスであるWebページをセレクターを使用してページ内容をパースし、データの抽出を行い、辞書形式でアイテムを返します。そして、スパイダーから返されたアイテムは、データベースやcsv、JSONで保存できます。

このように、クローラーのスタート地点を決め、どのような情報を取得し、どのようにクローラーを動かすのかをSpiderに記述します。下記はサンプルです。

```text
import scrapy


class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
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

        next_page_url = response.xpath('//*[@class="next"]/a/@href').get()
        abs_next_page_url = response.urljoin(next_page_url)
        if next_page_url is not None:
            yield scrapy.Request(abs_next_page_url, callback=self.parse)
```

* name：ターミナルでクローラーを起動させる場合に必要なクローラーの名前です。
* allowed\_domains：クロールできるドメインの文字列を渡します。
* start\_urls：クロールを開始するURLのリストです。複数指定できます。
* LinkExtractor：LinkExtractorを使えば、正規表現でクローラーを動かすディレクトリを指定できます。個人的には、意図していないページが正規表現のパターンに一致してしまい、サーバーに意図していないようなリクエストを大量に生みだすリスクを極力避けたいので、個人的には使っていません。他のページでも紹介していません。

このスクリプトがかければ、scrapyは動かすことができます。ターミナルで下記のように動かすと、大量のログとともにスクレイピングの情報が出力されます。上記のスクリプトであれば、スパイダーの名前はquotesなので、それを指定します。オプション-oでアウトプットする形式が選べます。XML、CSV、JSON、JSON LINES、XMLなどのさまざまな出力形式でエクスポートできます。

```text
$ scrapy crawl quotes -o output.json
```

#### Scrapy Shell

Scrapyは、基本的にはJupyter notebookでは動かすことができません。そのため、セレクターで情報を抽出する際にはインタラクティブに実行できず、自分が抽出したい情報なのかどうかを確かめるには、クローラーを起動するしかない、というようなことはありません。Scrapy Shellを使えばインタラクティブに実行しながらテストできます。

Scarpy Shellは下記のコマンドで実行できます。

```text
$ scrapy shell <url>
```

メインページから深く潜るためのURLの一覧を抽出できたら、次は詳細ページの情報を取得するコードを書く必要があります。その場合は、fetchを使うことで、ページを切り替えることができます。fetchは、指定されたURLからリクエストを取得し、オブジェクトを更新します。

```text
$ fetch(<detail url>)
```

Scrapyでクローラーを作る時は、Scarpy Shellで抽出するデータを確認しながら作業をすすめると、効率的に作業できるかと思います。

#### pipelines.py

スパイダーによってスクレイピングされた後、アイテムはアイテム・パイプラインに送られて、処理されます。 つまり、アイテムを受け取り、何らかのアクションを実行します。 また、そのアイテムがパイプラインで処理を継続されるか、そのアイテムを取り除くのをコントロールできます。

アイテム・パイプラインを利用することで、データのバリデーションチェックやデータクリーニング、画像であればサイズ変更やその他の画像処理、データベースへの保存などができます。

特定の条件い当てはまるものを削除するアイテム・パイプラインは下記のように記述します。

```text
from scrapy.exceptions import DropItem

class PricePipeline(object):

    vat_factor = 1.15

    def process_item(self, item, spider):
        if item.get('price'):
            if item.get('price_excludes_vat'):
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)
```

そして、アイテム・パイプライン・コンポーネントをアクティブにするために、settings.pyに書きを追加します。300というのは実行の順序を決める整数で0~1000まで自由に設定できますが、これ以外の処理もあるのであれば、それを考慮して順序関係を決定します。

```text
ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300
}
```

#### middlewares.py

後述するアーキテクチャの流れをみると、Middlewareというものににデータを通過させることがおおくありますが、このMiddlewareとは何でしょうか。

ScrapyにおけるMiddlewareとは、Scrapyを拡張させるために使用される機能とのこと。デフォルトでも多くの機能が提供されていますが、それでは足りない場合にはMIddlewareを使って拡張することになります。主に2つのMiddlewareがあります。

1. Downloader Middleware：Webページのダウンロード処理を拡張するものです。
2. Spider Middlware：コールバック関数の処理を拡張するものです。



### Scrapyのアーキテクチャ

クローラーを起動すると、どのようなことが内部で行われているのか、ドキュメントの画像をお借りしてまとめておく。

1. Scrapyエンジンは、Spidersからクロールする最初のRequestsを取得。 
2. Scrapyエンジンは、SchedulerでRequestsをスケジュールリングし、クロールする次のRequestsを求めます。 
3. Schedulerは、RequestsをScrapyエンジンに返します。 
4. Scrapyエンジン は、DownloaderにRequestsを送り、Middlewareを通過させます。 
5. ページのダウンロードが完了すると、 Downloaderは、そのページのレスポンスを生成し、Scrapyエンジンに送信。 Middlewareを通過させます。 
6. Scrapyエンジンは、 Downloaderからレスポンスを受け取り、それをSpidersに送って処理する。Middlewareを通過させます。 
7. Spidersはレスポンスを処理し、スクレイピングされたItemと新しいRequestsをScrapyエンジンに返し、Middlewareを通過させます。 
8. Scrapyエンジンは処理済みのItemを Item・pipelineに送信し、処理済みのRequestsをSchedulerに送信し、可能なら次のクロールを行います。 
9. Schedulerからの要求がなくなるまで、プロセスは\(ステップ1から\)繰り返されます。

![Scrapy Data Flow](.gitbook/assets/scrapy_architecture_02.png)



