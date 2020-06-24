# 第3章　Scrapyの環境設定

## はじめに

ここではScrapyの設定で利用するsettimgs.pyについて、各項目の内容をまとめておきます。デフォルトの設定では、サーバーに強い負荷をかける恐れがありますので、Scrapyのプロジェクトを作ったら、まずはじめにsettimgs.pyを調整するのがよいかもしれません。

### USER\_AGENT

Webページにアクセスする際のユーザーエージェントを設定できます。例えば、ChormeでWebページを確認しているのであれば、ユーザーエージェントを下記のようにChormeに設定しておくことで、Chormeと同じHTMLを返してくれます。クローラーの所有者を明らかにするために、メールアドレスを付ける方もいるとか。

```text
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36
```

デフォルトの設定は下記の通りです。

```text
# Crawl responsibly by identifying yourself (and your website) on the user-agent
#USER_AGENT = 'sample_pj (+http://www.yourdomain.com)'
```

Webブラウザによるすべてのリクエストには、ユーザーエージェントが含まれます。そのため、1つのユーザーエージェントから多数のリクエストを送信すると、ブロックされる場合があります。そのため、ユーザーエージェントを切り替えるなどの必要があります。これは1つのIPアドレスから多数のリクエストを送信する場合も同じで、ブロックされる可能性があります。

Scrapyには、scrapy-fake-useragentというライブラリを併用することで、毎回のリクエストで異なるユーザーエージェントからのリクエストを実現できます。また、scrapy-proxies-toolでは、IPアドレスを変更することで、特定のIPアドレスを禁止されることを防止できます。

### ROBOTSTXT\_OBEY

Trueにしておけば、robots.txtのDisallowページのクローリングを防止できるので、Trueが推奨されます。

```text
# Obey robots.txt rules
ROBOTSTXT_OBEY = True
```

### CONCURRENT\_REQUESTS

並列化してスクレイピングする\(同時リクエスト数\)かどうかの設定です。デフォルトは16にっているので、1~4とかくらいに設定している方が安全かと思われます。16だと個人のサーバーとかであればダウンする恐れもあります。ドメイン、IPごとに設定できます。

```text
# Configure maximum concurrent requests performed by Scrapy (default: 16)
#CONCURRENT_REQUESTS = 32

# The download delay setting will honor only one of:
#CONCURRENT_REQUESTS_PER_DOMAIN = 16
#CONCURRENT_REQUESTS_PER_IP = 16
```

### DOWNLOAD\_DELAY

これはリクエスト間隔の設定を行うもので、リクエスト間隔はデフォルトでは0秒です。これも3~5秒とかにしておけばよいと思います。慣習的に1秒スリープさせるとかもありますが。

```text
# Configure a delay for requests for the same website (default: 0)
# See https://docs.scrapy.org/en/latest/topics/settings.html#download-delay
# See also autothrottle settings and docs
#DOWNLOAD_DELAY = 3
```

### **COOKIES\_ENABLED**

これはクッキー・ミドルウェアを有効にするかどうかです。 無効にすると、クッキーはWebサーバーに送信されません。

```text
# Disable cookies (enabled by default)
#COOKIES_ENABLED = False
```

### Middleware

Middlewareについては、有効にする場合はコメントアウトを外します。また、独自でMiddlewareを拡張した場合は、行を追加して名前を記入します。

```text
# Enable or disable spider middlewares
# See https://docs.scrapy.org/en/latest/topics/spider-middleware.html
#SPIDER_MIDDLEWARES = {
#    'sample_pj.middlewares.SamplePjSpiderMiddleware': 543,
#}

# Enable or disable downloader middlewares
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
#DOWNLOADER_MIDDLEWARES = {
#    'sample_pj.middlewares.SamplePjDownloaderMiddleware': 543,
#}
```

### Itempipeline

Itempipelineの設定もsettings.pyで行います。

```text
# Configure item pipelines
# See https://docs.scrapy.org/en/latest/topics/item-pipeline.html
#ITEM_PIPELINES = {
#    'sample_pj.pipelines.SamplePjPipeline': 300,
#}
```

### DEFAULT\_REQUEST\_HEADER

基本的にはデフォルトの設定で問題ないと思われます。

```text
# Override the default request headers:
#DEFAULT_REQUEST_HEADERS = {
#   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
#   'Accept-Language': 'en',
#}
```

### HTTPCACHE\_ENABLED

スクレイピングする場所を調べるためにリクエストを何度も送ることがありますが、そのような場合にキャッシュを有効にしておけば、2回目以降はキャッシュを見に行くので、高速に機能します。

```text
# Enable and configure HTTP caching (disabled by default)
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-middleware-settings

// キャッシュを有効にするかどうか
#HTTPCACHE_ENABLED = True

// キャッシュの有効期限。0は無限を意味する。
#HTTPCACHE_EXPIRATION_SECS = 0

// キャッシュの保存先
#HTTPCACHE_DIR = 'httpcache'

// キャッシュしないステータスコード
#HTTPCACHE_IGNORE_HTTP_CODES = []
#HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'
```

キャッシュの保存先は`.scrapy`ディレクトリに作られる。下記は、有効にした場合にログに出力されるキャッシュの保存先。

```text
2020-06-22 23:48:40 [scrapy.extensions.httpcache] DEBUG: Using filesystem cache storage in
/Users/name/Documents/scrapy/<scrapy project>/.scrapy/httpcache
```

