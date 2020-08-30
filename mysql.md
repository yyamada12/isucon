# mysql

## 外部接続
my.cnf の `bind-address` が以下の様になっていると外部からアクセスできない。

```
bind-address = 127.0.0.1
```

よって、外部接続する場合接続したいマシンのIPアドレスに変更する

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
eyJoaXN0b3J5IjpbMTk0MDUxMzEzOCwxNjI5NTU0Njc1LC05MD
k0NTY5OTcsLTExNDg1NDcyMjksLTI3Njk0ODMxOCwxMDE1NDkx
NTIwLDExMTczNjk4MCw3NDIxOTU2MDVdfQ==
-->