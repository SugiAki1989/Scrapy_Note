# 第1章　スクレイピングの注意点

## はじめに

スクレイピングを行う際の注意事項をまとめておきます。著者は法律家や弁護士でもありませんので、これを頼りには行動しないでください。加え、本書の内容を法律家の助言と受け取らないようにご注意いただけますと幸いです。必要な場合は弁護士に相談してください。

ここでは、スクレイピングの際に注意すべき点についてまとめておきます。注意すべき点としては、「著作権」「動産不法侵入」「robots.txt」「利用規約」「リクエストの方法」などが考えられます。1つづ簡単にまとめておきます。

### 著作権

人によっては作られたものには「著作権」が発生します。芸術作品だけに著作権が発生するというわけではなく、Webページも著作物にあたりますし、私が書いた落書きも著作権が発生します。つまり、スクレイピングしてWebサイトから情報を取得することは、著作権が必然的に関わることにります。

ちなみに著作物について、著作権法第2条には、下記のようにあります。

> 思想又は感情を創作的に表現したものであつて、文芸、学術、美術又は音楽の範囲に属するものをいう。

著作権の利用許諾、利用禁止に関する著作者の権利として、Webスクレイピングが主に関わってくるのは、主に下記の3点かと思います。

* 複製権（第21条）：スクレイピングした情報を複製し保存する権利
* 翻案権（第27条）：スクレイピングした情報から新たな著作物を作る権利
* 公衆送信権（第23条）：スクレイピングした情報を公開する権利

「翻案」とは、新たな創作的表現を加えることを言うそうです。

これらは著作者の権利なので、実際に行う場合には、著作者の利用許諾が必要になりますが、著作権法は、いくつかの例外を認めています。それは「私的使用のための複製」「情報解析のための複製」と「検索エンジンの提供のための利用」です。

#### 私的使用のための複製 （第30条）

「私的使用のための複製」とは、文字の通りですが、私的に利用する範囲であれば、著作者の利用許諾なく、著作物が自由に使えるというものです。

[文化庁のWebページ](https://www.bunka.go.jp/seisaku/chosakuken/seidokaisetsu/gaiyo/chosakubutsu_jiyu.html)には下記のように記載されています。

> 家庭内で仕事以外の目的のために使用するために，著作物を複製することができる。同様の目的であれば，翻訳，編曲，変形，翻案もできる。 なお，デジタル方式の録音録画機器等を用いて著作物を複製する場合には，著作権者等に対し補償金の支払いが必要となる。 しかし，\[1\]公衆の使用に供することを目的として設置されている自動複製機器（注1）を用いて複製するときや，\[2\]技術的保護手段（注2）の回避により可能となった（又は，その結果に障害が生じないようになった）複製を，その事実を知りながら行うとき，\[3\]著作権等を侵害する自動公衆送信を受信して行うデジタル方式の録音又は録画を，その事実（＝著作権等を侵害する自動公衆送信であること）を知りながら行うときは，この例外規定は適用されない。 また，映画の盗撮の防止に関する法律により，映画館等で有料上映中の映画や無料試写会で上映中の映画の影像・音声を録画・録音することは，私的使用目的であっても，この例外規定は適用されない（注3）。

#### 情報解析のための複製等 （第47条の7）

「情報解析のための複製等」とは、スクレイピングしたデータをもとに、自分でデータ分析を行い、新たな価値を発見するためであれば、著作者の利用許諾なくデータを取得してもよいことになっています。だからといって、データ分析をするからと言って、何でもしてよいわけではありません。

[文化庁のWebページ](https://www.bunka.go.jp/seisaku/chosakuken/seidokaisetsu/gaiyo/chosakubutsu_jiyu.html)には下記のように記載されています。

> コンピュータ等を用いて情報解析（※）を行うことを目的とする場合には，必要と認められる限度において記録媒体に著作物を複製・翻案することができる。 ただし，情報解析用に広く提供されているデータベースの著作物については，この制限規定は適用されない。
>
>  ※情報解析とは，大量の情報から言語，音，映像等を抽出し，比較，分類等の統計的な解析を行うことをいう。

したがって、情報解析が目的であれば、著作物を複製したり、翻案することができるということになります。もちろん、それ以外の目的であればNGですし、情報解析を行わないにも関わらずスクレイピングした情報を他人に提供、公開するなどはNGとなりそうです。もちろん、この後に説明する「利用規約」という観点を忘れてはいけません。

情報解析のための複製等 （第47条の7）については、[ITmedia NEWS](https://www.itmedia.co.jp/news/)の記事で詳しく解説されています。

* [「日本は機械学習パラダイス」 その理由は著作権法にあり](https://www.itmedia.co.jp/news/articles/1710/10/news040.html)
* [改正著作権法が日本のAI開発を加速するワケ 弁護士が解説](https://www.itmedia.co.jp/news/articles/1809/06/news017.html)

#### 送信可能化された情報の送信元識別符号の検索等のための複製等（第47条の6）

「送信可能化された情報の送信元識別符号の検索等のための複製等」とは、GoogleやYahooのような検索エンジンを提供する業者は、著作物を公共に送信してもよいといものです。

[文化庁のWebページ](https://www.bunka.go.jp/seisaku/chosakuken/seidokaisetsu/gaiyo/chosakubutsu_jiyu.html)には下記のように記載されています。

> インターネット情報の検索サービスを業として行う者（一定の方法で情報検索サービス事業者による収集を禁止する措置がとられた情報の収集を行わないことなど、政令（施行令第7条の5）で定める基準を満たす者に限る。）は、違法に送信可能化されていた著作物であることを知ったときはそれを用いないこと等の条件の下で、サービスを提供するために必要と認められる限度で、著作物の複製・翻案・自動公衆送信を行うことができる。

これらのことを考慮すると、「著作者の観点」からは、私的利用で情報解析のためにスクレイピングすることは問題にならないと言えそうですが、もちろん、他にも注意すべき点はあります。

### 動産不法侵入

つぎは「動産不法侵入」についてまとめていきます。「動産」とは聞き慣れない言葉ですが、サーバーのようなものを指す言葉だそうです。当たり前な話ではありますが、サーバーは持ち主の資産であり所有物です。スクレイピングによって過度なリクエストを行うことで、サーバーがダウンしてしまい、本来のサービス提供の機能が制限される、または物理的に破損させてしまうなどがあると、これが関わってきます。動産不法侵入について問われかねない基準として、下記が考えられます。

#### 同意の欠如

サービスが提供されていれば、サーバーは誰でもアクセスできるようになっており、アクセスすることは可能です。つまり、同意を与えていることになりますが、特定のWebサイトでは、利用規約やサービス規約によって、スクレイピングの利用を禁止しています。

例えばAmazonでは、ログイン時に表示される[利用規約](https://www.amazon.co.jp/gp/help/customer/display.html/ref=jp_surl_conditions?nodeId=201909000)にスクレイピング、クローラーでの情報収集の禁止を明記しています。

> ### 利用許可およびサイトへのアクセス
>
> 本規約およびサービス規約の遵守を条件とし、アマゾンまたはコンテンツ提供者は、アマゾンサービスを限定的、非独占的、非商業的および個人的に利用する権利をお客様に許諾します（譲渡およびサブライセンス不可）。**この利用許可には、アマゾンサービスまたはそのコンテンツの転売および商業目的での利用、製品リスト、解説、価格などの収集と利用、アマゾンサービスまたはそのコンテンツの二次的利用、第三者のために行うアカウント情報のダウンロードとコピーやその他の利用、データマイニング、ロボットなどのデータ収集・抽出ツールの使用は、一切含まれません。**本規約またはその他の規約にて明示的に許諾されていない権利は全てアマゾンまたはそのライセンサー、供給者、出版者、権利保持者またはその他のコンテンツ権利者が留保します。アマゾンサービスまたはそのいかなる部分も、アマゾンからの書面による明示的な承諾を得ていない限り、商業目的のために、複製、複写、コピー、販売、再販、アクセス、その他の利用はできません。商標、ロゴ、およびアマゾンが有するその他の財産権的価値のある情報（画像、文字、ページレイアウト、フォームを含む）は、書面による明示的な承諾を得ていない限り、フレームにしたり、またはフレーム技術を使って取り込んだりすることはできません。また、アマゾンの明示的な書面による許諾なく、アマゾンの名称や商標をメタタグなど隠れたテキストとして使用することはできません。アマゾンサービスを不正に利用することは禁止されており、適用される法律に従ってのみ利用できます。本規約およびその他の利用規約に反する使用をした場合、アマゾンが使用許諾した権利は終了します。

ログイン画面では、ログインを「続行することで、 Amazonの[利用規約](https://www.amazon.co.jp/gp/help/customer/display.html/ref=ap_signin_notification_condition_of_use?ie=UTF8&nodeId=643006)および[プライバシー規約](https://www.amazon.co.jp/gp/help/customer/display.html/ref=ap_signin_notification_privacy_notice?ie=UTF8&nodeId=643000)に同意するものとみなされます」と明記されていますので、ログインするということは、利用規約に同意することになりますので、そこからスクレイピングした場合には、著作権法の例外があったとしても、それは利用規約違反にあたることになります。技術的にはスクレイピングできるからとはいえ、それを実行しても良いかは別問題です。

ほかにもクラウドファンディングのプラットフォームである[KickStarterの利用規約](https://www.kickstarter.com/terms-of-use?ref=global-footer)にもこのような記載があります。

![](.gitbook/assets/sukurnshotto-2020-06-27-114046png.png)

#### 実害の発生

サーバーは高価なものです。管理するためには、電力の供給にはじまり、冷却、監視、維持など様々なことを行う必要があります。これらを行う背景には、サーバー所有者のサービスを提供するためであって、間違ってもスクレイピングのためにサーバーを管理しているのではありません。

したがって、サービスの提供機能を過度なリクエストで制限した場合や、物理的に故障させるためにDDOS攻撃などを行えば実害が発生しますので、そのような場合は違法性を問われることになります。ちなみに、リクエストの間隔は1秒あければよい、という話も聞きますが、1秒には何の正当性も無いようです。

#### 意図的な行為

クローラーを走らせたり、スクレイピングするということは、その行為は意図的なものです。コードを書いたのであれば、そのコードが何をするかは理解しているはずです。問題が起こった際には、「知らない」では済まないということになります。

### robots.txt

robot.txtとは、クローラーの探索の動きに関して、指示を与えるものです。紳士協定のようなものらしいですが、用意されている以上は、サイトの管理者が行ってほしくないことが書かれているので、遵守するのが極めて無難かと思います。基本的には、https://www.website.com/robots.txtに配置されています。反対に解釈すると、スクレイピングしても良いページなど、許可している部分もrobot.txtから読み取ることができます。

| 名前 | 必須 | 内容 |
| :--- | :--- | :--- |
| User-agent | Yes | ユーザーエージェントの名前を指定します。Googlebotなどです。 |
| Disallow | No | クローラーにスクレイピングされたくない場所を指定します。 |
| Allow | No | クローラーにスクレイピングしてもよい場所を指定します。 |
| Sitemap | No | sitemap.xmlのURLを指定します。 |

例えば、[Yahoo!ニュースのrobots.txt](https://news.yahoo.co.jp/robots.txt)は、下記のようなフォーマットで書かれています。すべてのユーザーエージェントにアクセスすることを許可しており、3つのDisallowがありますので、この部分はスクレイピング、クローラーを走らせることが禁止されています。規則が互いに矛盾するような書き方がされることもありますが、この場合、最後の規則が優先されます。

```text
User-agent: *
Disallow: /comment/plugin/
Disallow: /comment/violation/
Disallow: /polls/widgets/
Sitemap: https://news.yahoo.co.jp/sitemaps.xml
Sitemap: https://news.yahoo.co.jp/sitemaps/article.xml
Sitemap: https://news.yahoo.co.jp/byline/sitemap.xml
Sitemap: https://news.yahoo.co.jp/polls/sitemap.xml
```

scrapyでは、オプションを設定することで、robot.txtがDisallowしている部分はスクレイピングしない、とう設定ができます。

### リクエスト関係について

最後にリクエストに関して、個人的にいくつか注意すべき点をまとめておきます。

#### リクエストの間隔

リクエストの間隔は1秒開ければ良いとも慣習的に言われますが、根拠は特にないようです。間隔は、開けれるだけ開ければ良いと思います。あくまでも、相手のサーバーに迷惑をかけずにスクレイピングすることが大切かと思います。

#### タイムアウト

リクエストを送っても、サーバーに負荷がかかっている状態だと、レスポンスが返ってこない場合があります。このような状況では、3~5秒待ってもレスポンスが返ってこないのであれば、リクエストをタイムアウトするのが良いかと思われます。また、リトライするときは2のn乗の間隔でリトライするのが良いと言われます。これはExponetial BackOffと言われる方法で、1,2,4,8,16秒…と間隔を開けます。

#### 並列化は避ける

スクレイピングする量や内容によっては、並列化して情報をスクレイピングしたいときもあると思いますが、本当に急ぐ必要がなければ並列化することは避けたほうが良いと思われます。

#### キャッシュを活用する

何度も同じページをリクエストしないようにキャッシュを利用するのもよいかもしれません。クライアントは1度キャッシュすれば、有効期限が切れるまではキャッシュの情報を利用するため、不要なリクエストが発生しません。CacheContorolライブラリでキャッシュを保存できるようになります。

**スクレイピングのタイミング**

スクレイピングのタイミングは、アクセスが少ない時間帯に行うように設定するのがよいかもしれません。サービスがフル稼働中の混雑時に、スクレイピングするような迷惑行為は避けるべきかと思います。

#### API

APIがあるのであれば、素直にAPIを使うのが望ましいかと思います。スクレイピングとかされないためにAPIを公開してくれているかもしれません。

### スクレイピングが事件に発展した例

最後に、スクレイピングによって事件に発展した例を下記の通り、まとめておきます。実際に自分がスクレイピングする前には一読することをおすすめします。

* [Librahack事件](https://ja.wikipedia.org/wiki/%E5%B2%A1%E5%B4%8E%E5%B8%82%E7%AB%8B%E4%B8%AD%E5%A4%AE%E5%9B%B3%E6%9B%B8%E9%A4%A8%E4%BA%8B%E4%BB%B6)
* [EBay v. Bidder's Edge](https://www.softic.or.jp/lib/cases/ebay_v_%20bidders_edge.htm)
* [Facebook, Inc. v. Power Ventures, Inc.](https://en.wikipedia.org/wiki/Facebook,_Inc._v._Power_Ventures,_Inc.)
* [EPIC - hiQ Labs, Inc. v. LinkedIn Corp.](https://epic.org/amicus/cfaa/linkedin/)

