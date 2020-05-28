# 第8章　スクレイピングのアラカルト

## はじめに

ここではWebスクレイピングに必要なWebの知識をまとます。Webスクレイピングを行うためには、Scrapyの使い方やXpathでの値の指定の仕方以外に、多くのことを知っていないと効率よく、かつ、迷惑をかけずにスクレイピングすることはできないように思います。

### HTTPについて

HTTPは、WebブラウザがWebサーバーと通信するためのプロトコルです。HTTPは、TCP/IPにおけるアプリケーションがやり取りを行う層のプロトコルで、トランスポート層の中に位置します。TCP/IPのことよりも、HTTPについて理解を深めることが、スクレイピングには大切かと思いますので、これ以降はHTTPについてまとめていきます。

いつも何のことなく使っているWebブラウザからです。Webブラウザといえば、Google chrome、Internet Explorer、Firefoxなどが有名なWebブラウザとしてあげられますが、これ何でしょうか。そもそもWebとは、World Wide Webを略して表現したもので、そのWebの中に、Webページがあります。WebページははHTML\(HyperText Markup Language\)という言語で構成されています。そのWebページを閲覧するために使うのものがブラウザです。

[WikipediaのHTMLサンプル](https://ja.wikipedia.org/wiki/HyperText_Markup_Language)をお借りします。`<xxx>hoge</xxx>`という方法でテキストをマークアップしていくことでHTMLは構成されます。そのため、HTMLというのは、マークアップ言語とも呼ばれます。マークアップ言語は、人間が見やすいものではありません。

```markup
<!DOCTYPE html>
<html lang="ja">
 <head>
  <meta charset="UTF-8">
  <link rel="author" href="mailto:mail@example.com">
  <title lang="en">HyperText Markup Language - Wikipedia</title>
 </head>
 <body>
  <article>
   <h1 lang="en">HyperText Markup Language</h1>
   <p>HTMLは、<a href="http://ja.wikipedia.org/wiki/SGML">SGML</a>
      アプリケーションの一つで、ハイパーテキストを利用してワールド
      ワイドウェブ上で情報を発信するために作られ、
      ワールドワイドウェブの<strong>基幹的役割</strong>をなしている。
      情報を発信するための文書構造を定義するために使われ、
      ある程度機械が理解可能な言語で、
      写真の埋め込みや、フォームの作成、
      ハイパーテキストによるHTML間の連携が可能である。</p>
  </article>
 </body>
</html>
```

そのため、マークアップ言語を解釈して、表示し直してくれるのが、Webブラウザの機能です。そこで、ブラウザがWebサーバーから、HTMLを含め様々な情報を取得するために使う手段がHTTPです。

WebブラウザがHTTPというプロトコルに従って、Webサーバーに「リクエスト」を送ります。Webサーバーは、そのリクエストを受け取って、Webブラウザに「レスポンス」を返します。これがWebブラウザとWebサーバーの1つのやり取りで、その通信方法がHTTPです。

### HTTPの内容

HTTPが実際にどのような内容をWebサーバーに送っているのか確認してみます。ターミナルから`curl`コマンドを利用します。`curl`コマンドは、ファイルを送信または受信するためのコマンド。バージョンはこちら。

```http
$ curl --version
curl 7.64.1 (x86_64-apple-darwin19.0) libcurl/7.64.1 (SecureTransport) LibreSSL/2.8.3 zlib/1.2.11 nghttp2/1.39.2
Release-Date: 2019-03-27
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS GSS-API HTTP2 HTTPS-proxy IPv6 Kerberos Largefile libz MultiSSL NTLM NTLM_WB SPNEGO SSL UnixSockets
```

[http://example.com](http://example.com/)にリクエストを送ってみます。`>`の部分がリクエストで、`<`の部分がレスポンスの部分です。なにやら沢山出力されたので、リクエストとレスポンスを分けて、内容を見ていきます。

```http
$ curl --verbose http://example.com
*   Trying 93.184.216.34...
* TCP_NODELAY set
* Connected to example.com (93.184.216.34) port 80 (#0)
> GET / HTTP/1.1
> Host: example.com
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Age: 532987
< Cache-Control: max-age=604800
< Content-Type: text/html; charset=UTF-8
< Date: Thu, 28 May 2020 08:36:36 GMT
< Etag: "3147526947+ident"
< Expires: Thu, 04 Jun 2020 08:36:36 GMT
< Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
< Server: ECS (oxr/832B)
< Vary: Accept-Encoding
< X-Cache: HIT
< Content-Length: 1256
< 
<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
* Connection #0 to host example.com left intact
* Closing connection 0
```

### HTTPリクエスト

まずはリクエストの部分です。4行目が「リクエスト行」で、5~7行目は「HTTPヘッダー」、8行目は「空白行」でヘッダーの終わりを意味し、POSTであれば、その下にwebサーバーにデータを送るための「メッセージボディ」が配置されます。

example.comに80番ポートから、GETというリクエストメソッドで接続を試みていることがわかります。HTTP1.1というのはHTTPのバージョンです。User-Agentというのは後で詳しく扱いますが、誰がしているのかを示します。ここでは、`curl`コマンドで行っていることがわかります。

```http
*   Trying 93.184.216.34...
* TCP_NODELAY set
* Connected to example.com (93.184.216.34) port 80 (#0)
> GET / HTTP/1.1
> Host: example.com
> User-Agent: curl/7.64.1
> Accept: */*
>
```

リクエストメソッドはGET以外にもよく使われるものとしてPOST、PUTなどがあります。コンテンツを取得する際に利用するものとして、GETとPOSTがありますが、GETはURLの後ろに情報を加える一方で、POSTはメッセージボディに情報を加えて通信します。基本的に見られて困る情報はPOSTで送ります。

| メソッド | 内容 |
| :--- | :--- |
| GET | 指定したURLのリソースをリクエスト。  |
| HEAD | GETリクエストと同じレスポンスを求めますが、レスポンス本文はなく、ヘッダのみ。 |
| POST | 指定したリソースに情報を送信するためのメソッド。 |
| PUT | 指定したURLにリソースを保存。 |
| DELETE | 指定したURLのリソースを削除。 |
| OPTIONS | 指定したURLの通信オプションを示すために使用。 |
| TRACE | サーバまでのネットワーク経路をチェック。 |
| PATCH | リソースを部分的に変更。 |
| CONNECT | TCPトンネルを接続する。暗号化したメッセージをプロキシサーバを経由して転送する際に用いる。 |

リクエストメソッドのPOST\(Create\)、GET\(Read\)、PUT\(Update\)、DELETE\(Delete\)の操作をまとめてCRUD操作として考えることをRESTアーキテクチャと呼びます。

### HTTPレスポンス

次はHTTPレスポンスの中身を見ていきます。1行目が「ステータス行」で、2〜12行目が「HTTPヘッダー」で、13行目が「空白行」でヘッダーの終わりを示す。14行目が「メッセージボディ」でHTMLが返されます。

ステータスコードが200で成功したことを意味し、コンテンツの内容はHTMLで、キャラセットはutf8ということがわかります。日付、キャッシュ、コンテンツの長さなどが書かれています。

```http
< HTTP/1.1 200 OK
< Age: 532987
< Cache-Control: max-age=604800
< Content-Type: text/html; charset=UTF-8
< Date: Thu, 28 May 2020 08:36:36 GMT
< Etag: "3147526947+ident"
< Expires: Thu, 04 Jun 2020 08:36:36 GMT
< Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
< Server: ECS (oxr/832B)
< Vary: Accept-Encoding
< X-Cache: HIT
< Content-Length: 1256
< 
<!doctype html>
<html>
<head>
【略】
</head>

<body>
【略】
</body>
</html>
* Connection #0 to host example.com left intact
* Closing connection 0
```

HTMLのデータを受け取り、Webブラウザは解析を行い、人間が見やすい形で表示します。その中に更に取得しなければ行けない情報\(画像など\)があると、再度、HTTPリクエストを送り、取得するように動きます。このHTTPリクエストとHTTPレスポンスを何度も行い、Webページを整えます。

### ステータスコード

HTTPレスポンスにはステータスコードが含まれます。先程の例では200が返されましたが、その他にも多くのステータスコードがあります。100番台から500番台までが使われます。



