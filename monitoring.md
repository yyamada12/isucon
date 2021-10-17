# alp
- [ ] インストール
```
wget https://github.com/tkuchiki/alp/releases/download/v1.0.3/alp_linux_amd64.zip
unzip alp_linux_amd64.zip
sudo install ./alp /usr/local/bin
rm alp_linux_amd64.zip alp

curl -L https://raw.githubusercontent.com/yyamada12/isucon-settings/master/alp.yml -o ~/alp.yml
```

- [ ] そもそもnginxをbenchが通っているかどうか確認する
通っていなければ、nginx をリバプロとして設定する
https://github.com/yyamada12/isucon10/commit/76ce0248358e683dc261a694e51472583984e395
- [ ] nginxのログフォーマットを変更する

* /etc/nginx/nginx.conf

```nginx
http {
  log_format ltsv "time:$time_local"
    "\thost:$remote_addr"
    "\tforwardedfor:$http_x_forwarded_for"
    "\treq:$request"
    "\tstatus:$status"
    "\tmethod:$request_method"
    "\turi:$request_uri"
    "\tsize:$body_bytes_sent"
    "\treferer:$http_referer"
    "\tua:$http_user_agent"
    "\treqtime:$request_time"
    "\tcache:$upstream_http_x_cache"
    "\truntime:$upstream_http_x_runtime"
    "\tapptime:$upstream_response_time"
    "\tvhost:$host";
  access_log /var/log/nginx/access.log ltsv;
}
```

- [ ] nginx再起動

デプロイスクリプトを回す

または `sudo systemctl restart nginx` 




### slow log
- [ ] インストール
`各種インストール` のスクリプトで無事インストールされていれば不要
```
wget percona.com/get/pt-query-digest
sudo install ./pt-query-digest /usr/local/bin
rm pt-query-digest 
```

- [ ] mysql でslow log を設定

```
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 0
```



- [ ] mysql再起動

デプロイスクリプト回す

または `sudo systemctl restart mysql; sudo sytemctl restart アプリのサービス` 

- [ ]  mysql で以下のコマンドを実行し、設定が変わっていることを確認
```
show variables like  'slow_query%';
show variables like  'long%';
```

- [ ] 計測

```
sudo pt-query-digest --limit 10 /var/log/mysql/slow.log | less
```


#### mariaDBの場合

logの場所を `/var/log/mariadb/slow.log` にする必要があるかも



#### my.cnfの場所を調べるには

```
mysql --help | grep my.cnf
```

一番左から順に読み込まれている

はず、なのだが、、

> In MySQL 5.7, the default cnf is at:

```
/etc/mysql/mysql.conf.d/mysqld.cnf
```
by [stackoverflow](https://stackoverflow.com/questions/38490785/where-is-mysql-5-7-my-cnf-file)

といいつつ、 /etc/my.cnfに書かないと反映されないこともある
設定書いてみて反映されるファイルを特定すべし


### netdata
- [ ] インストール
`各種インストール` のスクリプトで無事インストールされていれば不要

```
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

- [ ] mysql のメトリクスを追加する
1. 以下のsql を実行 
```
create user 'netdata'@'localhost';
grant usage on *.* to 'netdata'@'localhost';
flush privileges;
```
2. netdataを再起動
```
sudo service netdata restart
```

- [ ] go のメトリクスを追加する
https://learn.netdata.cloud/docs/agent/collectors/python.d.plugin/go_expvar
```
package main
import  (
  _  "expvar"
  "net/http"
)

func  main()  {
  http.ListenAndServe("127.0.0.1:8080",  nil)
}
```

- [ ] ポートが公開されていない場合はsshにポートフォワーディングの設定を入れる
```~/.ssh/config
LocalForward 19999 localhost:19999
```
or
```
ssh -L 19999:localhost:19999
```

# pprof

- [ ] コードに埋め込む

```
import (
        _ "net/http/pprof"
)

func main() {
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
```


- [ ] 計測

#### tl;dr

```bash
go tool pprof -png -output pprof.png http://localhost:6060/debug/pprof/profile
```



#### ちゃんとやるなら



1. ベンチ実行中にプロファイルを取得

```
curl -s http://localhost:6060/debug/pprof/profile > log/cpu.pprof
```



scripts/get_pprof.sh

```
#!/bin/sh

set -eux

SCRIPT_DIR=$(dirname "$0")

cd $SCRIPT_DIR

cd ..
mkdir -p log
curl -s http://localhost:6060/debug/pprof/profile > log/cpu.pprof
```



エンドポイントごとに色々ある

```
http://localhost:6060/debug/pprof/heap
http://localhost:6060/debug/pprof/block
http://localhost:6060/debug/pprof/goroutine
http://localhost:6060/debug/pprof/threadcreate
http://localhost:6060/debug/pprof/mutex
```



2. CLIで確認

```
go tool pprof バイナリ log/cpu.pprof
```

`top` コマンドや `list 関数名` コマンドを使う



3. web UIで確認

```
go tool pprof -http=":1234" バイナリ ~/pprof/xxx.cpu.00x.pb.gz
```

[見方](<https://medium.com/eureka-engineering/go%E8%A8%80%E8%AA%9E%E3%81%AE%E3%83%97%E3%83%AD%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AA%E3%83%B3%E3%82%B0%E3%83%84%E3%83%BC%E3%83%AB-pprof%E3%81%AEweb-ui%E3%81%8C%E3%82%81%E3%81%A1%E3%82%83%E3%81%8F%E3%81%A1%E3%82%83%E4%BE%BF%E5%88%A9%E3%81%AA%E3%81%AE%E3%81%A7%E7%B4%B9%E4%BB%8B%E3%81%99%E3%82%8B-6a34a489c9ee>)

- [ ] ポートが公開されていない場合はsshにポートフォワーディングの設定を入れる
```~/.ssh/config
LocalForward 1234 localhost:1234
```
or
```
ssh -L 1234:localhost:1234
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbNDA3MDM2MDkwXX0=
-->