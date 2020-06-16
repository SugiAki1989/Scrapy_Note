# 補章2　R言語とクローラー

## はじめに

ここでは、R言語を用いて、簡単なスクレイピングのコードを作成します。データ分析言語と言えば、R、Python…最近だとJuliaなどだと思うので、本章ではPythonのScrapyを中心に内容を扱ってきたので、補章では、Rでのクローラー作成方法も紹介しておきます。

### 対象サイトとライブラリの読み込み

対象のサイトは「食べログ」です。大阪エリアの店舗をスクレイピングしてみます。

![Image of tabelog](.gitbook/assets/sukurnshotto-2020-06-16-231401png.png)

まずは必要なライブラリを読み込みます。ここでは`base`ではなく`tidyverse`の関数を中心にコードを組み立てます。加えて、Beautiful Soupをベースにしている`rvest`と`xml2`パッケージを読み込みます。`後で使うl`れるため、直接呼び出すことができます。ただし、それは確実に存在し`xml2`、この関数を`::`名前空間構文で使用する場合は、存在する場所から呼び出す必要があります。`xml2::read_html`

```text
pacman::p_load(tidyverse, rvest, xml2)
```

まずは店名一覧のページから店舗の詳細ページへのURLとクローラーを次のページにすすめるかどうかの判定関数を作る。ページ番号とURLを結合してHTMLパーサーにわたす方法をここではやってない。理由は大阪エリアの店舗情報のページ数がわからないので、数字を渡せない・・・なので、次のページURLがあれば次のページに進むようにしている。

```text
list_url_parse <- function(html){
  tmp <- html %>%
    rvest::html_nodes(., xpath = "//*[@class = 'list-rst__rst-name-target cpy-rst-name']") %>% 
    rvest::html_attr("href")

  res <- tibble::tibble(urls = tmp)
  return(res)
}

is_next_url <- function(html){
  res <- html %>%
    rvest::html_nodes(., xpath = "//*[@class = 'c-pagination__arrow c-pagination__arrow--next']") %>% 
    rvest::html_attr("href")

  return(res)
}
```

先程の関数を使って、ページのURL一覧を取得する関数を作る。`Sys.sleep(10)`多めに10秒いれている。

```text
get_urls <- function(start_url) {
  df <- NULL
  url <- start_url

  cat("Log: Stsrt\n")
  cat("Log:", as.character(Sys.time()), url, "\n")

  while (length(url) != 0) {
    html <- url %>% xml2::read_html()
    tmp <- list_url_parse(html = html)
    url <- is_next_url(html = html)
    df <- rbind(df, tmp)
    Sys.sleep(10)
    cat("Log:", as.character(Sys.time()), url, "\n")
  }

  cat("Log: End\n")
  return(df)
}
```

食べログの1ページをスタートURLに指定し、ページがある分クローラーを走らせる。ログを見る限り60ページあったので、1枚20URLだとすると1200URLあれば問題ない。ログとかもglueパッケージを使ったほうがよさそう。

```text
start_url <- "https://tabelog.com/osaka/rstLst/1"
df <- get_urls(start_url = start_url)

Log: Start
Log: 2020-06-07 02:57:30 https://tabelog.com/osaka/rstLst/1 
Log: 2020-06-07 02:57:41 https://tabelog.com/osaka/rstLst/2/ 
・・・
Log: 2020-06-07 03:08:35 https://tabelog.com/osaka/rstLst/59/ 
Log: 2020-06-07 03:08:46 https://tabelog.com/osaka/rstLst/60/ 
Log: 2020-06-07 03:08:58  
Log: End

df %>% 
  dplyr::count()
# A tibble: 1 x 1
      n
  <int>
1  1200

df %>% 
  dplyr::n_distinct()
[1] 1200
```

これで大阪エリアのURLの一覧が手に入ったので、このURLを使って、詳細ページの情報を抽出する。ここでは、店名、住所、電話番号を取得する関数を作る。

```text
page_url_parse <- function(url){
  detail_html <- url %>% xml2::read_html()

  name <- detail_html %>%
    rvest::html_node(., xpath = "//*[@class = 'display-name']") %>% 
    rvest::html_text(trim = TRUE)

  address <- detail_html %>%
    rvest::html_node(., xpath = "//*[@class = 'rstinfo-table__address']") %>% 
    rvest::html_text(trim = TRUE)

  # 先に表示される店舗基本情の電話番号
  phone <- detail_html %>%
    rvest::html_node(., xpath = "//*[@class = 'rstinfo-table__tel-num']") %>% 
    rvest::html_text(trim = TRUE)

  latlong <- detail_html %>%
    rvest::html_nodes(., xpath = './/*[@class="rstinfo-table__map-wrap"]/div/a/img') %>% 
    rvest::html_attr("data-original") %>% 
    stringr::str_extract(., pattern = "([\\d.]+),([\\d.]+)") %>% 
    stringr::str_split(., pattern = ",")

  lat <- latlong[[1]][[1]]
  long <- latlong[[1]][[2]]

  res <- tibble::tibble(name, address, phone, lat, long)
  cat("Log:{'name':", name, ", 'address':", address, ", 'phone':", phone, ", 'lat':", lat, ", 'long':", long, "}\n")

  return(res)
}
```

別にfor-loopでも良いが、ここでは`map()`を使う。

```text
df_osaka <- df %>% 
  dplyr::mutate(tmp = purrr::map(.x = urls, 
                                 .f = function(x){
                                 Sys.sleep(10)
                                 page_url_parse(x)})) %>% 
  tidyr::unnest(cols = c(tmp), keep_empty = TRUE)

Log:{'name': 夜景 チーズとお肉のソラバル 梅田店 , 'address': 大阪府大阪市北区小松原町1-27 梅田エビスビル 9F , 'phone': 050-5595-3405 , 'lat': 34.70228264268257 , 'long': 135.50153635551814 }
Log:{'name': 活さば問屋 , 'address': 大阪府大阪市北区堂山町5-4 ABC観光ビル 1F , 'phone': 050-5596-4471 , 'lat': 34.70283704028684 , 'long': 135.50224960409074 }
Log:{'name': しゃかりき432" 堂山店 , 'address': 大阪府大阪市北区堂山町10-15 天神ビル 1F , 'phone': 050-5456-9194 , 'lat': 34.70337007924187 , 'long': 135.5045632762873 }…
```

こんな感じで1200件の店舗情報が取得できている。

```text
df_osaka
# A tibble: 1,200 x 6
   urls                                              name                                 address                                        phone         lat                long             
   <chr>                                             <chr>                                <chr>                                          <chr>         <chr>              <chr>            
 1 https://tabelog.com/osaka/A2701/A270101/27103352/ "夜景 チーズとお肉のソラバル 梅田店" 大阪府大阪市北区小松原町1-27 梅田エビスビル 9F 050-5595-3405 34.70228264268257  135.501536355518…
 2 https://tabelog.com/osaka/A2701/A270101/27090601/ "活さば問屋"                         大阪府大阪市北区堂山町5-4 ABC観光ビル 1F       050-5596-4471 34.70283704028684  135.502249604090…
 3 https://tabelog.com/osaka/A2701/A270101/27116951/ "しゃかりき432\" 堂山店"             大阪府大阪市北区堂山町10-15 天神ビル 1F        050-5456-9194 34.70337007924187  135.5045632762873
 4 https://tabelog.com/osaka/A2701/A270101/27116105/ "鉄板焼き 華粋"                      大阪府大阪市北区曽根崎新地1-5-31  JMビル B1F   080-2149-1779 34.69764613094936  135.497917425098…
 5 https://tabelog.com/osaka/A2701/A270101/27073634/ "北新地 湯木 新店"                   大阪府大阪市北区堂島1-5-39 マルタビル 1F       050-5869-6949 34.69616766207376  135.497007013224…
 6 https://tabelog.com/osaka/A2701/A270104/27115083/ "akala"                              大阪府大阪市中央区内淡路町2-3-17               050-5456-5607 34.68641842867681  135.513001133786…
 7 https://tabelog.com/osaka/A2701/A270103/27084786/ "輝楽家"                             大阪府大阪市北区天神橋1-11-13                  050-5571-8107 34.69405370101387  135.511735887323…
 8 https://tabelog.com/osaka/A2701/A270101/27073090/ "今井きしょう"                       大阪府大阪市北区梅田1-3-1 大阪駅前第1ビル  B1F 050-5596-5109 34.699063725204596 135.496091400370…
 9 https://tabelog.com/osaka/A2701/A270101/27007749/ "サントロペ"                         大阪府大阪市北区鶴野町4 コープ野村B棟 101      050-5868-0928 34.70930717432185  135.501834618436…
10 https://tabelog.com/osaka/A2701/A270202/27092954/ "THE WALL CABANA"                    大阪府大阪市中央区西心斎橋2-17-9 8F            06-6212-1730  34.670589851643655 135.4975398132911
# … with 1,190 more rows
```

MySQLに放り込んでおく。

```text
mysql> create database tabelog;
Query OK, 1 row affected (0.00 sec)

library(RMySQL)

con <- dbConnect(
  drv = RMySQL::MySQL(),
  dbname = "tabelog",
  user = "root",
  password = "pass",
  host = "localhost",
  port = 3306
)

dbWriteTable(con, "osaka_tbl", df_osaka)
[1] TRUE


mysql> select * from osaka_tbl limit 10;
+-----------+---------------------------------------------------+----------------------------------------------------+-------------------------------------------------------------------+---------------+--------------------+--------------------+
| row_names | urls                                              | name                                               | address                                                           | phone         | lat                | long               |
+-----------+---------------------------------------------------+----------------------------------------------------+-------------------------------------------------------------------+---------------+--------------------+--------------------+
| 1         | https://tabelog.com/osaka/A2701/A270101/27103352/ | 夜景 チーズとお肉のソラバル 梅田店                 | 大阪府大阪市北区小松原町1-27 梅田エビスビル 9F                    | 050-5595-3405 | 34.70228264268257  | 135.50153635551814 |
| 2         | https://tabelog.com/osaka/A2701/A270101/27090601/ | 活さば問屋                                         | 大阪府大阪市北区堂山町5-4 ABC観光ビル 1F                          | 050-5596-4471 | 34.70283704028684  | 135.50224960409074 |
| 3         | https://tabelog.com/osaka/A2701/A270101/27116951/ | しゃかりき432" 堂山店                              | 大阪府大阪市北区堂山町10-15 天神ビル 1F                           | 050-5456-9194 | 34.70337007924187  | 135.5045632762873  |
| 4         | https://tabelog.com/osaka/A2701/A270101/27116105/ | 鉄板焼き 華粋                                      | 大阪府大阪市北区曽根崎新地1-5-31  JMビル B1F                      | 080-2149-1779 | 34.69764613094936  | 135.49791742509896 |
| 5         | https://tabelog.com/osaka/A2701/A270101/27073634/ | 北新地 湯木 新店                                   | 大阪府大阪市北区堂島1-5-39 マルタビル 1F                          | 050-5869-6949 | 34.69616766207376  | 135.49700701322436 |
| 6         | https://tabelog.com/osaka/A2701/A270104/27115083/ | akala                                              | 大阪府大阪市中央区内淡路町2-3-17                                  | 050-5456-5607 | 34.68641842867681  | 135.51300113378682 |
| 7         | https://tabelog.com/osaka/A2701/A270103/27084786/ | 輝楽家                                             | 大阪府大阪市北区天神橋1-11-13                                     | 050-5571-8107 | 34.69405370101387  | 135.51173588732354 |
| 8         | https://tabelog.com/osaka/A2701/A270101/27073090/ | 今井きしょう                                       | 大阪府大阪市北区梅田1-3-1 大阪駅前第1ビル  B1F                    | 050-5596-5109 | 34.699063725204596 | 135.49609140037037 |
| 9         | https://tabelog.com/osaka/A2701/A270101/27007749/ | サントロペ                                         | 大阪府大阪市北区鶴野町4 コープ野村B棟 101                         | 050-5868-0928 | 34.70930717432185  | 135.50183461843628 |
| 10        | https://tabelog.com/osaka/A2701/A270202/27092954/ | THE WALL CABANA                                    | 大阪府大阪市中央区西心斎橋2-17-9 8F                               | 06-6212-1730  | 34.670589851643655 | 135.4975398132911  |
+-----------+---------------------------------------------------+----------------------------------------------------+-------------------------------------------------------------------+---------------+--------------------+--------------------+
10 rows in set (0.00 sec)

mysql> select count(1)  from osaka_tbl;
+----------+
| count(1) |
+----------+
|     1200 |
+----------+
1 row in set (0.00 sec)
```

データベースに入れた値を読み出す際には注意が必要。そのままクエリを飛ばすと、文字化けが起こる。

```text
library(DBI)
library(RMySQL)
con <- dbConnect(
   drv = RMySQL::MySQL(),
  dbname = "tabelog",
  user = "root",
  password = "pass",
  host = "localhost",
  port = 3306
)

data1 <- dbGetQuery(con, "select * from osaka_tbl limit 5;")
data1
  row_names                                              urls               name                       address
1         1 https://tabelog.com/osaka/A2701/A270101/27103352/ ?? ??????????? ???   ????????????1-27 ??????? 9F
2         2 https://tabelog.com/osaka/A2701/A270101/27090601/              ?????     ???????????5-4 ABC???? 1F
3         3 https://tabelog.com/osaka/A2701/A270101/27116951/      ?????432" ???      ???????????10-15 ???? 1F
4         4 https://tabelog.com/osaka/A2701/A270101/27116105/            ???? ?? ?????????????1-5-31  JM?? B1F
5         5 https://tabelog.com/osaka/A2701/A270101/27073634/          ??? ?? ??     ??????????1-5-39 ????? 1F
          phone               lat               long
1 050-5595-3405 34.70228264268257 135.50153635551814
2 050-5596-4471 34.70283704028684 135.50224960409074
3 050-5456-9194 34.70337007924187  135.5045632762873
4 080-2149-1779 34.69764613094936 135.49791742509896
5 050-5869-6949 34.69616766207376 135.49700701322436
```

なので、`dbGetQuery()`で文字コードをセットしてから読み込む。

```text
data <- dbGetQuery(con, "set names utf8") 
data <- dbGetQuery(con, "select * from osaka_tbl limit 5;")
data
  row_names                                              urls                               name
1         1 https://tabelog.com/osaka/A2701/A270101/27103352/ 夜景 チーズとお肉のソラバル 梅田店
2         2 https://tabelog.com/osaka/A2701/A270101/27090601/                         活さば問屋
3         3 https://tabelog.com/osaka/A2701/A270101/27116951/              しゃかりき432" 堂山店
4         4 https://tabelog.com/osaka/A2701/A270101/27116105/                      鉄板焼き 華粋
5         5 https://tabelog.com/osaka/A2701/A270101/27073634/                   北新地 湯木 新店
                                         address         phone               lat               long
1 大阪府大阪市北区小松原町1-27 梅田エビスビル 9F 050-5595-3405 34.70228264268257 135.50153635551814
2       大阪府大阪市北区堂山町5-4 ABC観光ビル 1F 050-5596-4471 34.70283704028684 135.50224960409074
3        大阪府大阪市北区堂山町10-15 天神ビル 1F 050-5456-9194 34.70337007924187  135.5045632762873
4   大阪府大阪市北区曽根崎新地1-5-31  JMビル B1F 080-2149-1779 34.69764613094936 135.49791742509896
5       大阪府大阪市北区堂島1-5-39 マルタビル 1F 050-5869-6949 34.69616766207376 135.49700701322436
```

以上メモおわり。

