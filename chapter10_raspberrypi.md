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

* [Raspberry Pi Kit](https://www.amazon.co.jp/gp/product/B082VVJCPT/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1)
* [SDメモリカードフォーマッター](https://www.sdcard.org/jp/downloads/formatter/eula_mac/index.html)
* [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)

SDカードをRaspberry Piに差し込み、電源を起動すると、Raspberry Piが起動します。設定ウィザードが表示されるので、順に従って設定します。

「Set Country」では、「日本」を設定しておきます。「Change Password」では、パスワードは初期設定のままだと「pi / raspberry」になっているので、よしなに修正します。「Set Up Screen」が表示されるので、スクリーンの端に黒い線が表示されているのであればチェックボックスにチェックをいれます。「Select WiFi NetWork」では、WiFiのネットワークを設定します。「Update Software」では、アップデートしておきます。そして、再起動します。

再起動後、「Raspberry Piマーク &gt; 設定 &gt; Raspberry Piの設定」と進み、

![](.gitbook/assets/sukurnshotto-2020-06-08-181918png.png)

「インターフェース」のSSH、VNCを有効にします。これでSSHとVNCでアクセスができます。

![Interface](.gitbook/assets/sukurnshotto-2020-06-08-182119png.png)

ここからはScrapyとMySQLをインストールしていきます。Raspberry PiにはPython3がすでにインストールされているので、Scrapyをインストールします。あわせてPyMySQLをインストールしておきます。

```text
pi@raspberrypi:~ $ sudo apt-get update
pi@raspberrypi:~ $ sudo apt-get upgrade
pi@raspberrypi:~ $ sudo apt-get install python-scrapy

~~~~~~
略
~~~~~~

pi@raspberrypi:~ $ pip install PyMySQL
Looking in indexes: https://pypi.org/simple, https://www.piwheels.org/simple
Collecting PyMySQL

  Downloading https://files.pythonhosted.org/packages/ed/39/15045ae46f2a123019aa968dfcba0396c161c20f855f11dea6796bcaae95/PyMySQL-0.9.3-py2.py3-none-any.whl (47kB)
    100% |████████████████████████████████| 51kB 787kB/s 
Installing collected packages: PyMySQL
Successfully installed PyMySQL-0.9.3

```

次はMySQLです。

```text
$ sudo apt-get install mariadb-server
$ sudo mysql_secure_installation

~~~~~
設定内容は省略
~~~~~
```

とりあえずテスト用のDB`test_db`とテスト用のユーザー`user01`を作成します。

```text
$ sudo mysql -u root -p
password : ******

MariaDB [(none)]> CREATE DATABASE test_db;
MariaDB [(none)]> CREATE USER 'user01'@'localhost' IDENTIFIED BY 'user01';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON test_db.* TO 'user01'@'localhost';
MariaDB [(none)]> FLUSH PRIVILEGES;

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test_db            |
+--------------------+
4 rows in set (0.001 sec)

MariaDB [(none)]> USE mysql;
MariaDB [mysql]> SELECT USER, HOST FROM mysql.user;
+--------+-----------+
| USER   | HOST      |
+--------+-----------+
| root   | localhost |
| user01 | localhost |
+--------+-----------+
2 rows in set (0.001 sec)
```

cronの設定を行います。下記コンフィグファイルを開き、cronがコメントアウトされているので、`#`を削除してcronを使えるようにします。

```text
pi@raspberrypi:~ $ sudo vi /etc/rsyslog.conf

###############
#### RULES ####
###############
#
# First some standard log files.  Log by facility.
#
auth,authpriv.*         /var/log/auth.log
*.*;auth,authpriv.none      -/var/log/syslog
#cron.*             /var/log/cron.log
daemon.*            -/var/log/daemon.log
kern.*              -/var/log/kern.log
lpr.*               -/var/log/lpr.log
mail.*              -/var/log/mail.log
user.*              -/var/log/user.log
```



テキストエディタとしてVS CODEを入れておきます。

```text
pi@raspberrypi:~ $ sudo -s
root@raspberrypi:/home/pi# . <( wget -O - https://code.headmelted.com/installers/apt.sh )
root@raspberrypi:~# su pi
```

下記のコマンドでVS CODEを起動できます。

```text
pi@raspberrypi:~ $ code-oss
```

![](.gitbook/assets/sukurnshotto-2020-06-08-185538png.png)

試しにデスクトップに`hello.py`を作成して実行してみます。

```text
pi@raspberrypi:~ $ python3 ~/Desktop/hello.py
Hello Python From VS CODE
```

補足ですが、GUIからCUIに変更するには、「Raspberry Piマーク &gt; 設定 &gt; Raspberry Piの設定」と進み、「システム」からブートをCLIにしてから再起動します。

![](.gitbook/assets/sukurnshotto-2020-06-08-184301png.png)

再起動後はCLIになります。戻すためには下記の通り設定します。

```text
$ sudo raspi-config

3 Boot Options
→ B1 Desktop / CLI
→→ B4 Desktop Autologin Desktop GUI, automatically logged in as 'pi' user

5 Interfacing Options
→ P3 VNC ⇒　Enable　

7 Advanced Options
→ A5 Resolution
→→ モニタの解像度の設置
```

