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
SELECT * FROM table_name 
WHERE a = '1'
UNION ALL
 SELECT * FROM table_name 
WHERE a = '2'
```

[OR句の置き換えによるチューニング方法](https://oreno-it.info/archives/134)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2MjU0NjkyNzBdfQ==
-->