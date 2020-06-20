# 補章3　ScrapyとDocker

## はじめに

ここでは

### Splashのインストール

Dockerの話

```python
$ sudo docker pull scrapinghub/splash
$ sudo docker run -p 5023:5023 -p 8050:8050 -p 8051:8051 scrapinghub/splash
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-splash'
2020-06-20 03:05:50.051261 [-] Splash version: 3.4.1
2020-06-20 03:05:50.163295 [-] Qt 5.13.1, PyQt 5.13.1, WebKit 602.1, Chromium 73.0.3683.105, sip 4.19.19, Twisted 19.7.0, Lua 5.2
2020-06-20 03:05:50.165098 [-] Python 3.6.9 (default, Nov  7 2019, 10:44:02) [GCC 8.3.0]
2020-06-20 03:05:50.166229 [-] Open files limit: 1048576
2020-06-20 03:05:50.166437 [-] Can't bump open files limit
2020-06-20 03:05:50.188744 [-] proxy profiles support is enabled, proxy profiles path: /etc/splash/proxy-profiles
2020-06-20 03:05:50.189130 [-] memory cache: enabled, private mode: enabled, js cross-domain access: disabled
2020-06-20 03:05:50.401278 [-] verbosity=1, slots=20, argument_cache_max_entries=500, max-timeout=90.0
2020-06-20 03:05:50.402061 [-] Web UI: enabled, Lua: enabled (sandbox: enabled), Webkit: enabled, Chromium: enabled
2020-06-20 03:05:50.403264 [-] Site starting on 8050
2020-06-20 03:05:50.403699 [-] Starting factory <twisted.web.server.Site object at 0x7f675747b1d0>
2020-06-20 03:05:50.404881 [-] Server listening on http://0.0.0.0:8050

$ sudo pip install scrapy_splash
```

### Scrapy×Splash

偉人の名言サイトのJavaScript版ページを対象にします。このサイトの説明

### Splashサンプルコード

プロジェクト

```python
➜ scrapy startproject quotes_js
➜ cd quotes_js
➜ scrapy genspider quotes_spider_js quotes.toscrape.com
```

ここに

```text
# Enable or disable downloader middlewares
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810
}

SPLASH_URL = 'http://localhost:8050/'

DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
```

ここに

```python
# -*- coding: utf-8 -*-
import scrapy
from scrapy_splash import SplashRequest

class QuotesSpiderJsSpider(scrapy.Spider):
    name = 'quotes_spider_js'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/js']

    def start_requests(self):
        for url in self.start_urls:
            yield SplashRequest(url = url, callback = self.parse, endpoint = "render.html")

    def parse(self, response):
        quotes = response.xpath('//*[@class="quote"]')
        for quote in quotes:
            author = quote.xpath('.//*[@class="author"]/text()').get()
            quote = quote.xpath('.//*[@class="text"]/text()').get()
            yield {'author': author,
                   'quote': quote}

        # レンダリングされたHTMLなので、これでも問題ない
        # next_page_url = response.xpath('//*[@class="next"]/a/@href').get()
        # abs_next_page_url = response.urljoin(next_page_url)
        # if abs_next_page_url is not None:
        #     yield SplashRequest(abs_next_page_url, callback=self.parse)

        # Lua Scriptバージョン
        # URLに移動し、Nextボタンをクリックし、次のページのURLとHTMLを返す
        script = """function main(splash)
                assert(splash:go(splash.args.url))
                splash:wait(1)
                button = splash:select("li[class=next] a")
                splash:set_viewport_full()
                splash:wait(1)
                button:mouse_click()
                splash:wait(1)
                return {url = splash:url(),
                        html = splash:html()}
            end"""

        yield SplashRequest(url=response.url,
                            callback=self.parse,
                            endpoint='execute',
                            args={'lua_source': script})


```



