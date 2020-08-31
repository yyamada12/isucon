# SQL

SQL のチューニング

## OR句の置き換え

```
SELECT * 
FROM table_name 
WHERE a = '1' OR a = '2'
```
↓
```
SELECT * 
FROM table_name 
WHERE a = '1'
UNION ALL
SELECT * FROM table_name 
WHERE a = '2'
```

[OR句の置き換えによるチューニング方法](https://oreno-it.info/archives/134)

## 日付型
DATE型は、文字列に変換してWHERE 条件が指定されていると、INDEXが適用されない

```
```
SELECT ...
  FROM sales
 WHERE DATE_FORMAT(sale_date, "%Y-%M")
     = DATE_FORMAT(now()    , "%Y-%M")
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbOTAwMTYzMjM3LC0xMzQzNjE1NTAyXX0=
-->