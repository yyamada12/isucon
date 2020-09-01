
# Unix Domain Socket

## nginx -> golang app
参考
[Golang で書いた Web アプリケーションを UNIX ドメインソケットで公開 - at kaneshin](http://blog.kaneshin.co/entry/2016/05/29/020302)
[ISUCON5 予選 part3](https://qiita.com/gky360/items/dccb88f4aecd50970915#nginx---go-%E3%81%AE%E6%8E%A5%E7%B6%9A%E3%82%92-tcp-socket-%E3%81%8B%E3%82%89-unix-domain-socket-%E3%81%AB%E5%A4%89%E6%9B%B4)

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
3. goのソースを修正する
4. nginx.confを修正する

[修正例](https://github.com/yyamada12/isucon5/commit/9a3a5cece2fac6df35c6dd4d6fe9cf77f395a8a0)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY5MjU4NDAzMywtNjI3NjU1MTM4XX0=
-->