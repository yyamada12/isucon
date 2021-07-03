# ISUCON チェックリスト

## 開始直後

### 環境構築

- [ ] ssh/config

```
Host isucon1
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
Host isucon2
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
Host isucon3
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
```

- [ ] 必要なツールと設定をインストール 
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/yyamada12/isucon-settings/master/install_essentials.sh)"
```

- [ ] 計測ツールインストール

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/yyamada12/isucon-settings/master/installer.sh)"
```

- [ ] webサーバが何か確認する

```bash
sudo systemctl list-unit-files
 
sudo systemctl list-units --type=service --state=running
```

- [ ] 言語をgoに変更する



- [ ] gitにsshできるようにする
- 既に存在する鍵を利用する場合
```
mkdir -p ~/.ssh
echo '''秘密鍵''' >> ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa
```

- 鍵を1から作る場合
```
ssh-keygen -t rsa -b 4096 -C "12yacropolisy@gmail.com"
```
で生成し、[https://github.com/settings/keys](https://github.com/settings/keys) から登録

- ssh できることを確認
```
ssh -T git@github.com
```


- [ ] レポジトリ作る
    - **public など、 go より上のフォルダもレポジトリに含めるようにする**
    -  `du -sh *` で大きなファイルが有る場合はgitignoreする
    - `GOPATH` 配下もgitignoreすべし

```
/webapp/nodejs/
/webapp/python/
/webapp/perl/
/webapp/php/
/webapp/ruby/
/webapp/rust/
.*
!.gitignore
!.bashrc
!.alias.bash
/pprof
```



- [ ] etcをレポジトリに追加する
  - /etcからファイルを移動し、シンボリックリンクをはる
  -  mysqlはシンボリックリンクだと設定が反映されないため、毎回手動で ~/etc から /etc にコピーする運用とする

```bash
mkdir ~/etc
# nginx
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.org
sudo mv /etc/nginx/nginx.conf ~/etc/
sudo chmod 666 ~/etc/nginx.conf
sudo ln -s ~/etc/nginx.conf /etc/nginx/nginx.conf

# mysql
sudo cp /etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf.org
sudo cp /etc/mysql/mysql.conf.d/mysqld.cnf ~/etc/
sudo chmod 666 ~/etc/mysqld.cnf

# sysctl.conf
sudo cp /etc/sysctl.conf /etc/sysctl.conf.org
sudo mv /etc/sysctl.conf ~/etc/sysctl.conf
sudo chmod 666 ~/etc/sysctl.conf
sudo ln -s ~/etc/sysctl.conf /etc/sysctl.conf
```

- mysql は設定ファイルの場所がまちまちなので気をつける
以下のファイルの場合もあり
```
sudo cp /etc/my.cnf /etc/my.cnf.org
sudo cp /etc/my.cnf ~/etc/
sudo chmod 666 ~/etc/my.cnf
```

- benchが動くことを一応確認しておく
```
sudo systemctl restart mysql
sudo systemctl restart nginx
```
ベンチ実行して問題なければOK

- [ ] デプロイスクリプトの準備

```
vim deploy.sh
:set paste
```

```bash
#!/bin/bash

set -eux

date -R
echo "Started deploying."

# rotate logs
function rotate_log () {
  if sudo [ -e $1 ]; then
    sudo mv $1 ${1%.*}_$(date +"%Y%m%d%H%M%S").${1##*.}
  fi
}
rotate_log /var/log/nginx/access.log
rotate_log /var/log/mysql/slow.log
rotate_log ~/pprof/pprof.png


# build go app
cd Makefileのパス
make

# update mysqld.cnf
if [ -e ~/etc/mysqld.cnf ]; then
  sudo cp ~/etc/mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf
fi

# restart services
sudo systemctl restart mysql
sudo systemctl restart アプリのサービス
sudo systemctl restart nginx

date -R
echo "Finished deploying."
```

```
chmod +x deploy.sh
```


- [ ] レポジトリを他のサーバーにpullする
2台目、3台目のサーバーで以下の手順を実施
```
git init
git remote add origin レポジトリURL
git pull origin main
```
-> 競合するファイルが出力されるはずなので、それらのファイルを削除してもう一度 `git pull origin main` 

デプロイしてベンチが通ればOK


- [ ] systemdにおまじないを追加する

起動に失敗した時（再起動試験時mysqlサーバーへの接続失敗など）のsystemdによる再起動試行回数を999とする。
```
[Service]
StartLimitBurst=999
```
```
sudo systemctl daemon-reload
```

## VSCode Remote 開発環境構築

- [ ] remote ssh につないだ状態で拡張機能 Goを追加する
- [ ] 不要なファイルの追跡を止める
```
{
    "files.watcherExclude": {
        "**/.*/**": true,
        "**/webapp/nodejs/**": true,
        "**/webapp/perl/**": true,
        "**/webapp/php/**": true,
        "**/webapp/python/**": true,
        "**/webapp/ruby/**": true,
        "**/webapp/rust/**": true,
        "**/bin/**": true,
        "**/pkg/**": true,
        "**/src/**": true,
    }
}
```

- [ ] remote用のsettings.jsonにGOROOT, GOPATHを必要であれば設定する

```json
{
    "go.goroot": "/home/isucon/local/go",
    "go.gopath": "/home/isucon/go:/home/isucon/webapp/go/"
}
```



※ import文で指定したパッケージはGOPATHが通っている箇所であればコンパイラが見つけることができ、go get により外部のパッケージをダウンロード・インストールする場合はGOPATHの１つ目のPATHのみ有効となる





## 計測

### alp
- [ ] インストール
`各種インストール` のスクリプトで無事インストールされていれば不要
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



#### h2oの場合

- ログフォーマットを変更する
- alp.ymlを修正する

/etc/h2o/h2o.conf

```
access-log:
  path: /var/log/h2o/access.log
  format: "time:%t\thost:%h\tua:\"%{User-agent}i\"\tstatus:%s\treq:%r\turi:%U\treqtime:%{duration}x\tsize:%b\tmethod:%m\t"
```



#### **sudo: alp: command not found** になる場合

<https://cha-shu00.hatenablog.com/entry/2017/03/02/123659>

`sudo visudo` で↓を

```
Defaults       secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

↓にする

```
#Defaults       secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Defaults        env_keep +="PATH"
```





### slow log
- [ ] インストール
`各種インストール` のスクリプトで無事インストールされていれば不要
```
wget percona.com/get/pt-query-digest
sudo install ./pt-query-digest /usr/local/bin
rm pt-query-digest 
```

- [ ] my.cnf でslow log を設定

/etc/my.cnf

```
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 0
```



- [ ] mysql再起動

デプロイスクリプト回す

または `sudo systemctl restart mysql; sudo sytemctl restart アプリのサービス` 

確認方法
```
show variables like  'slow_query%';
show variables like  'long%';
```

- [ ] 計測

```
sudo pt-query-digest --limit 10 /var/log/mysql/slow.log | less
```





#### mariaDBの場合

logの場所を `/var/log/mariadb/slow.log` にする



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


### pprof

- [ ] インストール

```
go get -u github.com/google/pprof
```

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

- [ ] ビルド

デプロイスクリプトを回す

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
pprof バイナリ log/cpu.pprof
```

`top` コマンドや `list 関数名` コマンドを使う



3. web UIで確認

```
pprof -http=":1234" バイナリ log/cpu.pprof
```

[見方](<https://medium.com/eureka-engineering/go%E8%A8%80%E8%AA%9E%E3%81%AE%E3%83%97%E3%83%AD%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AA%E3%83%B3%E3%82%B0%E3%83%84%E3%83%BC%E3%83%AB-pprof%E3%81%AEweb-ui%E3%81%8C%E3%82%81%E3%81%A1%E3%82%83%E3%81%8F%E3%81%A1%E3%82%83%E4%BE%BF%E5%88%A9%E3%81%AA%E3%81%AE%E3%81%A7%E7%B4%B9%E4%BB%8B%E3%81%99%E3%82%8B-6a34a489c9ee>)







## インフラチューニング

### サーバースペック確認
[Linuxでコマンドラインからマシンスペックを確認する方法](https://qiita.com/DaisukeMiyamoto/items/98ef077ddf44b5727c29)

CPU 
```
cat /proc/cpuinfo
```

メモリ
```
cat /proc/meminfo
```


### nginx

参考:

- [ISUCON5 予選 part3](https://qiita.com/gky360/items/dccb88f4aecd50970915)
- [fix nginx.conf](https://github.com/gky360/isucon5-qual-etc/commit/b20b5fc5f445600db213c374b025bcf901f71118)
- [ISUCON 5でalpを使ってNginxのログを解析した話](https://papix.hatenablog.com/entry/2015/09/28/094310)


- [ ] エラーログの出力先を指定、ログレベルをwarnにする (main > `error_log` )

- [ ] nginx -> go の接続を unix ドメインソケット化

```
upstream app {
  server unix:/var/run/isuxi/go.sock;
}
```

- [ ] 静的ファイルを nginx から配信 (http > server > location)

```
server {
  location ~ ^/(img|css|js|favicon.ico) {
    root /home/isucon/webapp/static;
  }
  ...
```

- [ ] (main > `worker_process auto;` )
- [ ] MIMEタイプ読み込み (http > `types_hash_max_size 2048;` ), (http > `include       /etc/nginx/mime.types;` )
- [ ] keepalive 設定 (http > `keepalive_timeout 65;` ), (http > `keepalive_requests 500;` )
- [ ] (http > `sendfile on;` )
- [ ] (http > `tcp_nopush  on;` )
- [ ] (http > `tcp_nodelay on;` )
- [ ] (http > `open_file_cache max=100 inactive=60s;` )



### /etc/sysctl.conf

参考：https://kazeburo.hatenablog.com/entry/2014/10/14/170129 , インフラの基本 p248,  https://qiita.com/sion_cojp/items/c02b5b5586b48eaaa469

- [ ] /etc/sysctl.conf に以下を書き込む

```
net.ipv4.tcp_max_tw_buckets = 2000000
net.ipv4.ip_local_port_range = 10000 65000
net.core.somaxconn = 32768
net.core.netdev_max_backlog = 8192
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 10
```

- [ ] 以下のコマンドで設定反映

```
sudo /sbin/sysctl -p
```

### /etc/mysql/my.cnf

- [ ] 初期値を確認
```
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW VARIABLES LIKE 'innodb_flush%';
SHOW VARIABLES LIKE 'innodb_support%';
SHOW VARIABLES LIKE 'max_connections';
```

- [ ] /etc/mysql/my.cnf に以下を書き込む

```
[mysqld]
innodb_buffer_pool_size=1G
innodb_flush_log_at_trx_commit=0
innodb_flush_method=O_DIRECT
innodb_support_xa=OFF
max_connections=10000
```



## 改善

### app
- [ ] ミドルウェアなどのロギングがあれば止める (めっちゃ効く)
- [ ] GOMAXPROCS の値を確認する

### mysql

- [ ] indexを張る
- [ ] SELECTするカラムを指定する
- [ ] アプリケーションへキャッシュする
- [ ] redisへキャッシュする
- [ ] join句を使ってクエリを減らす
- [ ] replace句を使ってinsertクエリとupdateクエリをまとめる

### nginx

- [ ] （ログに `Too many open files` と出た場合） `worker_rlimit_nofile` を増やす
- [ ] （ログに `worker_connections are not enough` と出た場合） `worker_connections` を増やす
- [ ] （リクエストサイズが1MBより大きい場合, nginxが 413 Request Entity Too Large を返している場合）`client_max_body_size` を増やす
- [ ] （ログに `[warn] ... client request body is buffered to a temporary file ...` と出た場合） `client_body_buffer_size` を増やす
- [ ] （ログに `[warn] ... upstream response is buffered to a temporary file ...` と出た場合） `proxy_buffers` のサイズを増やす

## 再起動試験
- [ ] ベンチ実施後、全台で `sudo reboot` を実行する
- [ ] 起動後、アプリを操作し、ベンチによる変更が保存されていることを確認する

## 提出前

- [ ] slowlog を切る

- [ ] nginx のアクセスログの出力をオフにする (http > `access_log off;` )
- [ ]  netdata を切る
```sudo systemctl disable netdata```

- [ ] appのログを切る (go の middlewareのコード削除など)
- [ ]  VSCode Remote SSH をサーバーから削除する (**Remote-SSH: Uninstall VS Code Server from Host...**)
https://code.visualstudio.com/docs/remote/troubleshooting#_cleaning-up-the-vs-code-server-on-the-remote

<!--stackedit_data:
eyJoaXN0b3J5IjpbNjMxMTAxMjc4LC02MjA4MTQ3MDksLTEzNT
UxODA3OTEsLTE5MjI5MjAwMTAsMTE2NzE2MDcxMSwxNTM0MTMx
OTQ4LDUwNTg0Mjk1NSwtMTc4ODM5NjczNiwtODA5MjM4ODcsMT
g3NTU1MTM0MCwtMTUxOTg1MDM2MSw0MzA0MTgwMTQsMTI2ODIx
MDE5OSwxMjMwNzYzNTQ0LC01NDIyOTY5MzcsMTUzMTc5NTcxMi
wtMTEwNzc4OTY3OCwtMTU4NTY2NTAxMCwtMTUwMzk4ODY3MCwt
NjgxMzg4NDg4XX0=
-->