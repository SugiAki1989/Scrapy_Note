# Scrapy Note・・・製作中

## はじめに

この本は、Pythonで書かれたWebクロールフレームワークであるScrapyを中心に、Webスクレイピングの基礎的な部分からScrapyの使い方まで、個人的に学習した内容をまとめているものです。例えば、wget、正規表現、BeautifulSoup、Seleniumなどを組み合わせることで、クローラーの作成やWebスクレイピングはできますが、ここではScrapyに焦点をあてています。

備忘録みたいなものなので、この中で作成したscrapyのプロジェクトはパブリックリポジトリで管理してないです。なので、役に立つような情報はない・・・よ。

また、**Pythonについては、2020年5月からScrapyの学習に合わせて使い始めました。そのため、コードの記述において、杜撰な箇所が散見されると思いますし、トンチンカンな説明を書いている可能性は否めません。**基本的には参考書籍やサイトの内容をもとに、自分用にまとめています。

### 参考書籍とサイト

下記の書籍を参考に、Scrapyの利用方法をまとめています。

* [Dimitrios Kouzis-Loukas \(2016\) Learning Scrapy, Packt Publishing](https://www.packtpub.com/big-data-and-business-intelligence/learning-scrapy)
* [Scrapy Tutorial](https://docs.scrapy.org/en/latest/intro/tutorial.html#)

### 免責事項

Webスクレイピングは時と場合によっては法律の問題に発展する恐れがありますので、ここにまとめられているサンプルコードを実行したことによって、万一いかなる損害が発生したとしても、著者はいかなる責任も負いません。また、本書はスクレイピングを推奨するものではありません。サンプルスクリプトを参考に、スクレイピングを実行される場合は、すべて自己責任でお使いいただけますと幸いです。

### 実行環境

macOSの環境で実行しています。バージョンは下記の通りです。

```text
$ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.4
BuildVersion:	19E287
```

Pythonのバージョンは下記のとおりです。

```text
$ python3 -V
Python 3.8.2
```

Scrapyのバージョンは下記のとおりです。

```text
$ scrapy -V
Scrapy 2.0.1 
```

