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

Scrapyのプロジェクトファイルは、`scrapy startproject`コマンドで生成できます。ここでは、「sample\_pj」という名前のプロジェクトフォルダを作成しています。 

```text
$ scrapy startproject sample_pj
New Scrapy project 'sample_pj', using template directory '/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/scrapy/templates/project', created in:
    /Users/aki/Desktop/sample_pj

You can start your first spider with:
    cd sample_pj
    scrapy genspider example example.com
```

`scrapy genspider`コマンドでクローラーの名前とスクレイピングしたいページのURLを指定します。指示にしたがって、`scrapy genspider`コマンドでクローラーの名前「sample」と対象のサイトのURLとして「example.com」を指定します。

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

```python
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

```python
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

```python
ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300
}
```

#### middlewares.py

後述するアーキテクチャの流れをみると、Middlewareというものににデータを通過させることがおおくありますが、このMiddlewareとは何でしょうか。

ScrapyにおけるMiddlewareとは、Scrapyを拡張させるために使用される機能とのことです。デフォルトでも多くの機能が提供されていますが、それでは足りない場合にはMIddlewareを使って拡張することになります。主に2つのMiddlewareがあります。

1. Downloader Middleware：Webページのダウンロード処理を拡張するものです。
2. Spider Middlware：コールバック関数の処理を拡張するものです。

Downloader Middlewareは下記の通りデフォルトで設定されています。詳細は[ドキュメント](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html)を参照ください。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x540D;&#x524D;</th>
      <th style="text-align:left">&#x5185;&#x5BB9;</th>
      <th style="text-align:left">&#x5B9F;&#x884C;&#x9806;&#x5E8F;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">RobotsTxtMiddleware</td>
      <td style="text-align:left">robots.txt&#x306B;&#x5F93;&#x3063;&#x3066;&#x30AF;&#x30ED;&#x30FC;&#x30EA;&#x30F3;&#x30B0;&#x3059;&#x308B;</td>
      <td
      style="text-align:left">100</td>
    </tr>
    <tr>
      <td style="text-align:left">HttpAuthMiddleware</td>
      <td style="text-align:left">&#x5225;&#x540D;BASIC&#x8A8D;&#x8A3C;&#x3092;&#x4F7F;&#x7528;&#x3057;&#x3066;&#x3001;&#x7279;&#x5B9A;&#x306E;&#x30B9;&#x30D1;&#x30A4;&#x30C0;&#x30FC;&#x304B;&#x3089;&#x751F;&#x6210;&#x3055;&#x308C;&#x305F;&#x3059;&#x3079;&#x3066;&#x306E;&#x30EA;&#x30AF;&#x30A8;&#x30B9;&#x30C8;&#x3092;&#x8A8D;&#x8A3C;&#x3057;&#x307E;&#x3059;&#x3002;</td>
      <td
      style="text-align:left">300</td>
    </tr>
    <tr>
      <td style="text-align:left">DownloadTimeoutMiddleware</td>
      <td style="text-align:left">&#x30C0;&#x30A6;&#x30F3;&#x30ED;&#x30FC;&#x30C9;&#x306E;&#x30BF;&#x30A4;&#x30E0;&#x30A2;&#x30A6;&#x30C8;&#x306E;&#x8A2D;&#x5B9A;</td>
      <td
      style="text-align:left">350</td>
    </tr>
    <tr>
      <td style="text-align:left">DefaultHeadersMiddleware</td>
      <td style="text-align:left">HTTP&#x30D8;&#x30C3;&#x30C0;&#x3092;&#x4ED8;&#x4E0E;</td>
      <td style="text-align:left">400</td>
    </tr>
    <tr>
      <td style="text-align:left">UserAgentMiddleware</td>
      <td style="text-align:left">
        <p>&#x30B9;&#x30D1;&#x30A4;&#x30C0;&#x30FC;&#x304C;&#x30C7;&#x30D5;&#x30A9;&#x30EB;&#x30C8;&#x306E;&#x30E6;&#x30FC;&#x30B6;&#x30FB;&#x30A8;&#x30FC;&#x30B8;&#x30A7;&#x30F3;&#x30C8;&#x3092;&#x30AA;&#x30FC;&#x30D0;&#x30FC;&#x30E9;&#x30A4;&#x30C9;&#x3067;&#x304D;&#x308B;&#x3088;&#x3046;&#x306B;&#x3059;&#x308B;&#x30DF;&#x30C9;&#x30EB;&#x30A6;&#x30A7;&#x30A2;&#x3002;</p>
        <p>&#x30B9;&#x30D1;&#x30A4;&#x30C0;&#x30FC;&#x304C;&#x30C7;&#x30D5;&#x30A9;&#x30EB;&#x30C8;&#x306E;&#x30E6;&#x30FC;&#x30B6;&#x30FC;&#x30FB;&#x30A8;&#x30FC;&#x30B8;&#x30A7;&#x30F3;&#x30C8;&#x3092;&#x4E0A;&#x66F8;&#x304D;&#x3059;&#x308B;&#x306B;&#x306F;&#x3001;&#x305D;&#x306E;
          user_agent &#x5C5E;&#x6027;&#x3092;&#x8A2D;&#x5B9A;&#x3059;&#x308B;&#x5FC5;&#x8981;&#x304C;&#x3042;&#x308A;&#x307E;&#x3059;&#x3002;</p>
      </td>
      <td style="text-align:left">500</td>
    </tr>
    <tr>
      <td style="text-align:left">RetryMiddleware</td>
      <td style="text-align:left">&#x63A5;&#x7D9A;&#x30BF;&#x30A4;&#x30E0;&#x30A2;&#x30A6;&#x30C8;&#x3084;HTTP
        500&#x30A8;&#x30E9;&#x30FC;&#x306A;&#x3069;&#x306E;&#x4E00;&#x6642;&#x7684;&#x306A;&#x554F;&#x984C;&#x304C;&#x539F;&#x56E0;&#x3067;&#x3042;&#x308B;&#x53EF;&#x80FD;&#x6027;&#x306E;&#x3042;&#x308B;&#x5931;&#x6557;&#x3057;&#x305F;&#x8981;&#x6C42;&#x3092;&#x518D;&#x8A66;&#x884C;&#x3059;&#x308B;&#x30DF;&#x30C9;&#x30EB;&#x30A6;&#x30A7;&#x30A2;&#x3002;</td>
      <td
      style="text-align:left">550</td>
    </tr>
    <tr>
      <td style="text-align:left">AjaxCrawlMiddleware</td>
      <td style="text-align:left">Ajax&#x30AF;&#x30ED;&#x30FC;&#x30EB;&#x53EF;&#x80FD;&#x306A;&#x30DA;&#x30FC;&#x30B8;&#x3092;&#x30AF;&#x30ED;&#x30FC;&#x30EB;&#x3057;&#x76F4;&#x3059;</td>
      <td
      style="text-align:left">560</td>
    </tr>
    <tr>
      <td style="text-align:left">MetaRefreshMiddleware</td>
      <td style="text-align:left">meta refresh&#x30BF;&#x30B0;&#x306B;&#x3088;&#x308B;&#x30EA;&#x30C0;&#x30A4;&#x30EC;&#x30AF;&#x30C8;&#x51E6;&#x7406;</td>
      <td
      style="text-align:left">580</td>
    </tr>
    <tr>
      <td style="text-align:left">HttpCompressionMiddleware</td>
      <td style="text-align:left">&#x5727;&#x7E2E;(gzip&#x3001;deflate)&#x3055;&#x308C;&#x3066;&#x3044;&#x308B;&#x3082;&#x306E;&#x3092;Web&#x30B5;&#x30A4;&#x30C8;&#x304B;&#x3089;&#x9001;&#x53D7;&#x4FE1;&#x3067;&#x304D;&#x307E;&#x3059;&#x3002;</td>
      <td
      style="text-align:left">590</td>
    </tr>
    <tr>
      <td style="text-align:left">RedirectMiddleware</td>
      <td style="text-align:left">&#x30EA;&#x30C0;&#x30A4;&#x30EC;&#x30AF;&#x30C8;&#x3092;&#x51E6;&#x7406;&#x3059;&#x308B;</td>
      <td
      style="text-align:left">600</td>
    </tr>
    <tr>
      <td style="text-align:left">CookiesMiddleware</td>
      <td style="text-align:left">&#x30BB;&#x30C3;&#x30B7;&#x30E7;&#x30F3;&#x3092;&#x4F7F;&#x7528;&#x3059;&#x308B;&#x30B5;&#x30A4;&#x30C8;&#x306A;&#x3069;&#x3001;&#x30AF;&#x30C3;&#x30AD;&#x30FC;&#x3092;&#x5FC5;&#x8981;&#x3068;&#x3059;&#x308B;&#x30B5;&#x30A4;&#x30C8;&#x3092;&#x64CD;&#x4F5C;&#x3067;&#x304D;&#x307E;&#x3059;&#x3002;Web&#x30B5;&#x30FC;&#x30D0;&#x30FC;&#x304C;&#x9001;&#x4FE1;&#x3057;&#x305F;&#x30AF;&#x30C3;&#x30AD;&#x30FC;&#x3092;&#x8FFD;&#x8DE1;&#x3057;&#x3001;Web&#x30D6;&#x30E9;&#x30A6;&#x30B6;&#x30FC;&#x304C;&#x884C;&#x3046;&#x3088;&#x3046;&#x306B;&#x3001;&#x305D;&#x306E;&#x5F8C;&#x306E;(&#x30B9;&#x30D1;&#x30A4;&#x30C0;&#x30FC;&#x304B;&#x3089;&#x306E;)&#x30EA;&#x30AF;&#x30A8;&#x30B9;&#x30C8;&#x3067;&#x305D;&#x308C;&#x3089;&#x3092;&#x9001;&#x308A;&#x8FD4;&#x3057;&#x307E;&#x3059;&#x3002;</td>
      <td
      style="text-align:left">700</td>
    </tr>
    <tr>
      <td style="text-align:left">HttpProxyMiddleware</td>
      <td style="text-align:left">Request&#x30AA;&#x30D6;&#x30B8;&#x30A7;&#x30AF;&#x30C8;&#x306B;proxy&#x30E1;&#x30BF;&#x5024;&#x3092;&#x8A2D;&#x5B9A;&#x3059;&#x308B;&#x3053;&#x3068;&#x306B;&#x3088;&#x308A;&#x3001;&#x30EA;&#x30AF;&#x30A8;&#x30B9;&#x30C8;&#x306B;&#x4F7F;&#x7528;&#x3059;&#x308B;HTTP&#x30D7;&#x30ED;&#x30AD;&#x30B7;&#x3092;&#x8A2D;&#x5B9A;&#x3057;&#x307E;&#x3059;&#x3002;</td>
      <td
      style="text-align:left">750</td>
    </tr>
    <tr>
      <td style="text-align:left">DownloaderStats</td>
      <td style="text-align:left">Downloader&#x306E;&#x7D71;&#x8A08;&#x60C5;&#x5831;&#x3092;&#x53CE;&#x96C6;&#x3059;&#x308B;</td>
      <td
      style="text-align:left">850</td>
    </tr>
    <tr>
      <td style="text-align:left">HttpCacheMiddleware</td>
      <td style="text-align:left">HTTP&#x30AD;&#x30E3;&#x30C3;&#x30B7;&#x30E5;&#x3092;&#x51E6;&#x7406;&#x3059;&#x308B;</td>
      <td
      style="text-align:left">900</td>
    </tr>
  </tbody>
</table>DownloaderMiddlewareの処理はHTTPリクエストを送る前に順番が小さいものから順番に実行され、それが完了するとダウンロードが始まります。

SpiderMiddlewareは、コールバック関数の処理を拡張できます。下記がデフォルトで設定されているSpiderMiddlewareです。

|  | 内容 | 実行順序 |
| :--- | :--- | :--- |
| HttpErrorMiddleware | 失敗したHTTPレスポンスをフィルター処理して、対処しないようにする。 | 50 |
| OffsiteMiddleware | スパイダーが対象とするドメインから外れているURLのリクエストを除外します。 | 500 |
| RefererMiddleware | クエストを生成したレスポンスのURLに基づいて、リクエスト Referer ヘッダーを生成します。 | 700 |
| UrlLengthMiddleware | URLLENGTH\_LIMITより長いURLを持つリクエストを除外します。 | 800 |
| DepthMiddleware | スクレイピングされるサイト内の各リクエストの深さを追跡するために使用されます。 | 900 |

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



