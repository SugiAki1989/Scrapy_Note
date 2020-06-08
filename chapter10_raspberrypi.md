# 第10章 ScrapyとRaspberry Pi

## はじめに

ここではScrapyで作ったクローラーをRaspberry Piで定期的に実行する方法をまとめておきます。Raspberry Pi、Python3、Scrapyのバージョンは下記の通りです。

```text
pi@raspberrypi:~ $ cat /etc/issue
Raspbian GNU/Linux 10

pi@raspberrypi:~ $  python3 -V
Python 3.7.3

pi@raspberrypi:~ $ scrapy -V
Scrapy 1.5.1 - no active project
```

### Raspberry Piのセットアップ

ここで書く必要も無いかもしれませんが、まとめておきます。まずは購入したRaspberry Piにフォーマット済みのSDカードとRaspberry Pi Imagerを使って、Raspbianをインストールしていきます。

* [SDメモリカードフォーマッター](https://www.sdcard.org/jp/downloads/formatter/eula_mac/index.html)
* [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)

SDカードをRaspberry Piに差し込み、電源を起動すると、Raspberry Piが起動します。設定ウィザードが表示されます。順に従って設定します。

「Set Country」では、「日本」を設定しておきます。「Change Password」では、パスワードは初期設定のままだと「pi / raspberry」になっているので、よしなに修正します。「Set Up Screen」が表示されるので、スクリーンの端に黒い線が表示されているのであればチェックボックスにチェックをいれます。「Select WiFi NetWork」では、WiFiのネットワークを設定します。「Update Software」では、アップデートしておきます。そして、再起動します。

再起動後、「Raspberry Piマーク &gt; 設定 &gt; Raspberry Piの設定」と進み、

![](.gitbook/assets/sukurnshotto-2020-06-08-181918png.png)

「インターフェース」のSSH、VNCを有効にします。これでSSHとVNCでアクセスができます。

![Interface](.gitbook/assets/sukurnshotto-2020-06-08-182119png.png)

ここからはScrapyとMySQLをインストールしていきます。Raspberry Piに

