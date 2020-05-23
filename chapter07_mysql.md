# 第7章　ScrapyとMySQL

## はじめに

ここではScrapyでスクレイピングした値をMySQLのテーブルに格納する方法についてまとめていきます。MongoDBのようなNoSQLデータベースが使われることも多いようですが、ここではMySQLを扱います。

### アイテムの定義

あああ



```text

```

### クローラーの設計

ああああああああああ

```text

```

### パイプラインの設計

ああああ

```text

```

![Title duplication](.gitbook/assets/sukurnshotto-2020-05-24-45010png.png)

### 環境設定

ああああああ

```text

```

### テーブルの定義\(MySQL\)

ああああああ

```text
mysql> CREATE DATABASE book_online;
mysql> USE book_online;
mysql> CREATE TABLE books(id INT NOT NULL AUTO_INCREMENT,
                          title TEXT,
                          price VARCHAR(50),
                          detail_page_url TEXT,
                          PRIMARY KEY(id));
```

