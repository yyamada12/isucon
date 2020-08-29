# mysql

## テーブルのカラム追加
### こんな時に使える
- JOINが遅い

### 方法
1. `CREATE TABLE new_table AS SELECT ...` を使って新規テーブルを作成
[MySQL 5.6 リファレンスマニュアル 13.1.17.1 CREATE TABLE ... SELECT 構文](https://dev.mysql.com/doc/refman/5.6/ja/create-table-select.html)
2. てkぎい＼
2. アプリケーションコードを新しいテーブルを用いる用に修正


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk5NTI2Mjg5MF19
-->