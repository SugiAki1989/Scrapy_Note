# 第7章　ScrapyとMySQL

## はじめに

### アイテムの定義

### クローラーの設計

### パイプラインの設計

```text

```

![Title duplication](.gitbook/assets/sukurnshotto-2020-05-24-45010png.png)

### 環境設定

### テーブルの定義\(MySQL\)

```text
mysql> CREATE DATABASE book_online;
mysql> USE book_online;
mysql> CREATE TABLE books(id INT NOT NULL AUTO_INCREMENT,
                          title TEXT,
                          price VARCHAR(50),
                          detail_page_url TEXT,
                          PRIMARY KEY(id));
```

