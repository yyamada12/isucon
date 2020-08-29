# mysql

## テーブルのカラム追加
### こんな時に使える
- JOINが遅い

### 方法
1. `CREATE TABLE new_table AS SELECT ...` 構文を使って新規テーブルを作成する
[MySQL 5.6 リファレンスマニュアル 13.1.17.1 CREATE TABLE ... SELECT 構文](https://dev.mysql.com/doc/refman/5.6/ja/create-table-select.html)
2. 適宜INDEXを貼る
3. アプリケーションコードを新しいテーブルを用いる用に修正する


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ3MjY3NDQ4OF19
-->