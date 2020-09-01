
# Unix Domain Socket

## nginx -> golang app
1. ソケットファイルの置き場を作成する。  `/etc/tmpfiles.d/isucon.conf`  を作成して設定を書く。
```
$ cat /etc/tmpfiles.d/isuxi.conf
d /var/run/isuxi 0755 root root -
```
2. その後、設定ファイルを登録 & 反映
```
$ sudo systemd-tmpfiles --create /etc/tmpfiles.d/isucon.conf
$ sudo systemctl daemon-reload
```
3. 
[例](https://github.com/yyamada12/isucon5/commit/9a3a5cece2fac6df35c6dd4d6fe9cf77f395a8a0)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5OTQ0OTE2MzksLTYyNzY1NTEzOF19
-->