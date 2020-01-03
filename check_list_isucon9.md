# ISUCON チェックリスト

## 開始直後

- [ ] 環境構築

- [x]  ssh/config

```
Host isucon9-0
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
Host isucon9-1
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
Host isucon9-2
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
```



* [x] 各種インストール

ubuntu

```
sudo apt-get install -y vim tmux htop dstat glances unzip
```

centOS

```
sudo yum install -y vim tmux htop dstat glances unzip
```

* [x] bashrc

```
alias vim='sudo vim'
alias g='sudo git'
alias ga='sudo git add'
alias gd='sudo git diff'
alias gs='sudo git status'
alias gp='sudo git push'
alias gb='sudo git branch'
alias gst='sudo git status'
alias gco='sudo git checkout'
alias gf='sudo git fetch'
alias gci='sudo git commit'
alias git='sudo git'
```

* [x] gitconfig

```
[color]
  ui = auto
[alias]
  co = checkout
  ci = commit
  st = status
  bra = branch
  gr = log --all --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
```



- [x] webサーバが何か確認する

  nginx or go

- [x] 言語をgoに変更する



- [x] app レポジトリ作る
    - **public など、 go より上のフォルダもレポジトリに含めるようにする**

.gitignore

```
/webapp/nodejs/
/webapp/python/
/webapp/perl/
/webapp/php/
/webapp/ruby/
```



- [x] etc レポジトリ作る
    - /etcで `git init ` し、gitignoreで使用する設定ファイルのみ残す

.gitignore

```
/*
!.gitignore
!nginx/
!mysql/
mysql/debian.cnf
!sysctl.conf
!my.cnf
!my.cnf.d/
!h2o/
```



- [ ] デプロイスクリプトの準備

deploy.sh

```
#!/bin/sh

set -eux

SCRIPT_DIR=$(dirname "$0")

cd $SCRIPT_DIR

date -R
echo "Started deploying."

# rotate logs
./rotate_log.sh /var/log/nginx/access.log
./rotate_log.sh /var/log/nginx/error.log

./rotate_log.sh /var/log/mysql/mysqld.log
./rotate_log.sh /var/log/mysql/slow.log

./rotate_log.sh ../log/cpu.pprof

# build go app
cd ..
go build app.go
cd $SCRIPT_DIR

# restart services
sudo systemctl restart mysql
sudo systemctl restart アプリのサービス
sudo systemctl restart nginx

date -R
echo "Finished deploying."
```

rotate_log.sh

```
#!/bin/sh

set -eux

if sudo [ -e $1 ]; then
  sudo mv $1 ${1%.*}_$(date +"%Y%m%d%H%M%S").${1##*.}
fi
```



## 計測

### alp

- [ ] インストール

```
mkdir -p ~/tmp
cd ~/tmp
wget https://github.com/tkuchiki/alp/releases/download/v0.3.1/alp_linux_amd64.zip
unzip alp_linux_amd64.zip
sudo install ./alp /usr/local/bin
```



- [ ] nginx/h2o のログフォーマットを変更する

* nginxの場合

/etc/nginx/nginx.conf

```
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
```

* h2oの場合

/etc/h2o/h2o.conf

```
access-log:
  path: /var/log/h2o/access.log
  format: "time:%t\thost:%h\tua:\"%{User-agent}i\"\tstatus:%s\treq:%r\turi:%U\treqtime:%{duration}x\tsize:%b\tmethod:%m\t"
```



- [ ] nginx/h2o 再起動

デプロイスクリプトを回す

または `sudo systemctl restart nginx` 



- [ ] 計測

nginx

```
sudo alp -r --sum -f /var/log/nginx/access.log 
```

h2o

```
sudo alp -r --sum -f /var/log/h2o/access.log 
```



**sudo: alp: command not found** になる場合

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

- [ ] pt-query-digest のinstall

```
mkdir -p ~/tmp
cd ~/tmp
wget percona.com/get/pt-query-digest
sudo install ./pt-query-digest /usr/local/bin
```

- [ ] my.cnf でslow log を設定

/etc/my.cnf

```
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 0
```

**mariaDBの場合** 

logの場所を `/var/log/mariadb/slow.log` にする



my.cnfの場所を調べるには

```
mysql --help | grep my.cnf
```

一番左から順に読み込まれている



- [ ] mysql再起動

デプロイスクリプト回す

または `sudo systemctl restart mysql; sudo sytemctl restart アプリのサービス` 



- [ ] 計測

```
sudo pt-query-digest --limit 5 /var/log/mysql/slow.log | less
```



### netdata

CPUとかブラウザから観れるらしいから入れてみると良い

=> isucon9では断念

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


## 提出前

- [ ] slowlogを切る

```
# /etc/mysql/my.cnf を編集
sudo vim /etc/mysql/my.cnf

[mysqld]
slow_query_log = 0

# ログを削除
sudo rm -f /var/log/mysql/slow.log
```
- [ ] nginxのアクセスログの出力をオフにする (http > `access_log off;` )