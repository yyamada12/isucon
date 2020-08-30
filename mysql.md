# mysql

## 外部接続
[外部からMySQLの接続する際のチェックポイント](https://gist.github.com/koudaiii/10696132)

### mysqldのuserテーブルの設定

以下コマンドでユーザの権限状態を確認。
```
mysql> select user, host, password from mysql.user;
```
host が `localhost` だとアウト。
ip_address を `%` にして以下のクエリを実行する。
```
GRANT ALL PRIVILEGES 
ON [db_name].* TO [user_name]@"[ip_address]" 
IDENTIFIED BY '[password]' WITH GRANT OPTION;
```

[外部のホストから接続できるようにする方法](https://www.wakuwakubank.com/posts/322-mysql-access-host/)



### my.cnfのbind-address設定
my.cnf の `bind-address` が以下の様になっていると外部からアクセスできない。
```
bind-address = 127.0.0.1
```
よって、外部接続する場合この行をコメントアウトする

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




## テーブルのカラム追加
### こんな時に使える
- JOINが遅い
- 自テーブルへの参照を持っている
→ カテゴリテーブルに親カテゴリと子カテゴリのレコードが混在している場合など

### 方法
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MzMxMDczOTIsMTYyOTU1NDY3NSwtOT
A5NDU2OTk3LC0xMTQ4NTQ3MjI5LC0yNzY5NDgzMTgsMTAxNTQ5
MTUyMCwxMTE3MzY5ODAsNzQyMTk1NjA1XX0=
-->