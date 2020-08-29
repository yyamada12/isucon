# mysql

## テーブルのカラム追加
### こんな時に使える
- JOINが遅い
- 自テーブルへの参照を持っている
→ カテゴリテーブルに親カテゴリと子カテゴリのレコードが混在している場合など

### 方法
#### 新規テーブルを作る場合
ADD PRIMARY KEY でエラー出るかも？

1. `CREATE TABLE new_table AS SELECT ...` 構文を使って新規テーブルを作成する
[MySQL 5.6 リファレンスマニュアル 13.1.17.1 CREATE TABLE ... SELECT 構文](https://dev.mysql.com/doc/refman/5.6/ja/create-table-select.html)
2. 適宜PRIMARY KEY, INDEXを設定する
`ALTER TABLE tbl_name ADD PRIMARY KEY (col_name, ...)` 
`ALTER TABLE tbl_name ADD INDEX index_name(col_name, ...)`
3. アプリケーションコードを新しいテーブルを用いる用に修正する
- テーブル名を全て修正
- `SELECT * FROM ...` を 全て修正する
- 追加したカラムを利用してJOIN句を無くす


#### 既存のテーブルを拡張する場合
1. カラムを追加する
`ALTER TABLE table_name ADD COLUMN col_name col_definition
[MySQLでカラムを追加する](https://uxmilk.jp/12612)
2. 追加したカラムにデータを入れる
`UPDATE  
4. アプリケーションコードを修正する
- `SELECT * FROM ...` を 全て修正する
- 追加したカラムを利用してJOIN句を無くす
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NjkxNTQ1NjUsLTI3Njk0ODMxOCwxMD
E1NDkxNTIwLDExMTczNjk4MCw3NDIxOTU2MDVdfQ==
-->