# mysql

## 外部接続
[外部からMySQLの接続する際のチェックポイント](https://gist.github.com/koudaiii/10696132)

### mysqldのuserテーブルの設定

以下コマンドでユーザの権限状態を確認。
```
mysql> select user, host from mysql.user;
```
host が `localhost` だとアウト。
以下のSQLを実行する。
```
GRANT ALL PRIVILEGES 
ON [db_name].* TO [user_name]@"[ip_address]" 
IDENTIFIED BY '[password]' WITH GRANT OPTION;
```
USER/PASS が isucon/isuconであれば以下の通り。
```
GRANT ALL PRIVILEGES ON *.* TO isucon@"%" IDENTIFIED BY 'isucon' WITH GRANT OPTION;
```

[外部のホストから接続できるようにする方法](https://www.wakuwakubank.com/posts/322-mysql-access-host/)


#### mysql8 の場合
CREATE USER と GRANT を同時にできなくなったので、それぞれ実行する
```
CREATE USER isucon@'%' IDENTIFIED BY 'isucon';
GRANT ALL PRIVILEGES ON *.* TO isucon@'%' WITH GRANT OPTION;
```


### my.cnfのbind-address設定
my.cnf の `bind-address` が以下の様になっていると外部からアクセスできない。
```
bind-address = 127.0.0.1
```
よって、外部接続する場合この行をコメントアウトするか、接続元のIPアドレスを指定する

### ポートの確認

デフォルトではMySQLがTCPポート3306番でListenしているので、ポートが開いているか確認。

```
$ netstat -tlpn
```
`0.0.0.0:3306 ～ LISTEN xxxx/mysqld` の表示があればOK。

### ファイアーウォール設定を確認

Netfilter/iptables で、接続制限をかけていないか確認。
```
$ iptables -L
```
ここでファイアーウォールが有効になっている場合は、iptablesコマンドで制限を無効化するなどしてmysqlの通信を許可する。

## Generated Column

## 降順INDEX
ORDER BY の条件にDESCが入っているとINDEXを使ってくれない
数値のカラムの場合、マイナスかけて generated column すればOK


```
CREATE TABLE isuumo.estate
(
	...
	popularity  INTEGER             NOT NULL,
	minuspopularity INTEGER AS (-popularity) NOT NULL, 
	...
)
```

## テーブルのカラム追加
### こんな時に使える
- 降順INDEXのために、既存のカラムにマイナスをかけたカラムを追加する (5.7以前)
- JOINが遅い
- 自テーブルへの参照を持っている
→ カテゴリテーブルに親カテゴリと子カテゴリのレコードが混在している場合など

### 方法
自テーブルの情報を使う場合、Generated Columnを使うのが一番早くて楽。
レコード数が多いテーブルの場合updateに時間がかかるし、初期データ用のinsertSQLファイルを修正するのも大変。


#### Generated Column!!


#### 新規テーブルを作る場合
*\*ADD PRIMARY KEY でエラー出るかも？*

1. 新規テーブルを作成する
```
CREATE TABLE new_table AS SELECT ...
``` 
[MySQL 5.6 リファレンスマニュアル 13.1.17.1 CREATE TABLE ... SELECT 構文](https://dev.mysql.com/doc/refman/5.6/ja/create-table-select.html)

2. 適宜PRIMARY KEY, INDEXを設定する
```
ALTER TABLE tbl_name 
ADD PRIMARY KEY (col_name, ...)
``` 
```
ALTER TABLE tbl_name 
ADD INDEX index_name(col_name, ...)
```

3. アプリケーションコードを新しいテーブルを用いる用に修正する
- テーブル名を全て修正
- `SELECT * FROM ...` を 全て修正する
- INSERT時に追加したカラムへもSETする
- 追加したカラムを利用してJOIN句を無くす



#### 既存のテーブルを拡張する場合
*\*初期レコード数が多い場合はかなり時間がかかる*
1. カラムを追加する
```
ALTER TABLE table_name 
ADD COLUMN col_name col_definition
```
[MySQLでカラムを追加する](https://uxmilk.jp/12612)
2. 追加したカラムにデータを入れる
```
UPDATE
    tableA t1,
    tableB t2
SET 
    t1.column_01 = t2.column_01
WHERE
    t1.id = t2.id
```
[MySQLのupdateでselectした結果を使う方法 サブクエリで自己テーブル更新も可能](https://style.potepan.com/articles/19076.html)

3. 適宜INDEXを貼る
```
ALTER TABLE tbl_name 
ADD INDEX index_name(col_name, ...)
```

4. アプリケーションコードを修正する
- `SELECT * FROM ...` を 全て修正する
- INSERT時に追加したカラムへもSETする
- 追加したカラムを利用してJOIN句を無くす

### 例
降順INDEXのために、既存のカラムに-1をかけたgenerated columnを作成する


## MySQL パラメータチューニング
MySQLTunerをとりあえず動かしてみるのが良さそう
[MySQLTunerを使ってMySQLを診断しよう！] (https://note.dimage.co.jp/blog_009_mysqltuner.html)

```
wget http://mysqltuner.pl/ -O mysqltuner.pl
```


## Query Cache

### 設定方法
`query_cache_type=1` は必須。
sizeとlimitはメモリ容量に合わせて。
```
query_cache_type=1
query_cache_size=512M
query_cache_limit=8M
```

### クエリーキャッシュメモリが足りているか確認する

`show global status like "%QCache%";` の値を見て、 `Qcache_lowmem_prunes`  (メモリーが少ないためクエリーキャッシュから削除されたクエリーの数) が小さかったり、Qcache_free_memory (クエリーキャッシュの使用できるメモリーの残り) が大きければ、メモリの大きさは十分。

https://qiita.com/ryurock/items/9f561e486bfba4221747
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIyODQ5OTU3NCwyMDMxNjUyMDU0LC0xMT
U0MjUxNzg0LDY5MzU4Nzg2NywtNjc5Mzc4NjcxLDEwNTgwMjIx
ODUsMTE1NTQ3NTQ2MiwxMDE4MjMzNzUyLC0xMjkyMzQ1MjM5LC
0xNTMzMTA3MzkyLDE2Mjk1NTQ2NzUsLTkwOTQ1Njk5NywtMTE0
ODU0NzIyOSwtMjc2OTQ4MzE4LDEwMTU0OTE1MjAsMTExNzM2OT
gwLDc0MjE5NTYwNV19
-->