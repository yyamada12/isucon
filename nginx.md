# nginx

## 設定の正しさ確認
`nginx -t` とすると設定ファイルのミスを指摘してくれる


## ロードバランス

[参考](https://www.nedia.ne.jp/blog/tech/2016/08/04/7938)

### 設定方法

1. `upstream` にサーバーグループ名とサーバ群を定義する
2. proxy_passをサーバーグループ名に変更する

```nginx
http {
		upstream isucon_servers {
			server 192.168.0.1:8000 weight=1;
    		server 192.168.0.2:8000 weight=8;
    		server 192.168.0.3:8000 weight=1;
		}

		server {
	      	listen          80;

	      	location / {
      			proxy_set_header Host $host;
      			proxy_pass	http://isucon_servers;
	      	}
		}
}
```



### カスタム

* weightを設定することで、振り分けの重みを変更することができる
* `upstream` ディレクティブ内に `least_conn` を設定することでコネクション数の少ないサーバに優先的に振り分けるようになる

```
upstream isucon_servers {
	least_conn;
  server 192.168.0.1:8000;
  server 192.168.0.2:8000;
  server 192.168.0.3:8000;
}
```





## http2

[参考](https://www.rem-system.com/nginx-http2/)

https通信を利用している場合は、http2が利用できる

http1.1よりずっと高速になるらしい

### 設定方法

`server` ディレクティブ内の `listen` ディレクティブに `http2` を追記するだけ

```nginx
http {
  server {
    listen 443 ssl;
    …
  }
}

```

↓

```nginx
http {
  server {
    listen 443 ssl http2;
    …
  }
}
```



## 静的配信 & ブラウザキャッシュ

### 静的配信設定方法

* `root` ディレクティブでパスを指定すれば、パスに合致した場合に静的配信される。基本これでいけるはず。

```nginx
http {
	server {
    root /home/isucon/isucari/webapp/public
  }
}
```

* ファイルを指定したい場合は `location` ディレクティブを併用して指定する(アンチパターン？)

```nginx
location /static/ {
    root /path/to/app/static;
}

location ~ ^/(img|css|js|favicon.ico) {
	root /home/isucon/webapp/static;
}
```

**※ pathはlocation の手前で指定する必要あり**
例: https://github.com/yyamada12/isucon11q_re/commit/4b822d909286c549669eeff1fae20fbc6248449c

path/to/public/assets/xxx.js をNginxで返す場合
OK: 
```
location /assets {
  root /home/isucon/webapp/public/;
}
```
NG: 
```
location /assets {
  root /home/isucon/webapp/public/assets;
}
```

**※ /の有無でも挙動が変わるので注意**
OK: 
```
location / {
  root /home/isucon/webapp/public;
  try_files $uri /index.html;
}
```
NG: 
```
location / {
  root /home/isucon/webapp/public/;
  try_files $uri index.html;
}
```

### ブラウザキャッシュ設定方法

方法1:  `expires` ディレクティブを設定する

→ ファイルが変更されていようがお構いなくキャッシュする



```nginx
http {
	server {
		location ~ .*\.(css|gif|jpeg|jpg|js|png) {
        expires 7d;
    }
  }
}
```

`expires -1` と設定すると、ファイルがアップデートされているかサーバーに問い合わせる。



方法2: `Cache-Control` ディレクティブを設定する

```nginx
http {
	server {
		location ~ .*\.(css|gif|jpeg|jpg|js|png) {
        add_header Cache-Control "public max-age=86400";
    }
  }
}
```

#### ブラウザキャッシュ確認方法

ヘッダに `Expires:` や `Cache-Control: ` が付与されていることを確認する

```bash
curl -I http://url
```





### gzip圧縮

基本は gzip_static を on/always にして、publicフォルダの中を `gzip -r *` で圧縮しておけば良い

```nginx
http {
	gzip_staic always
}
```



nginxにgzipさせる場合の設定は以下の通り

[参考](https://qiita.com/cubicdaiya/items/2763ba2240476ab1d9dd)

```nginx
http {
  gzip on;
  gzip_types text/css text/javascript application/json application/javascript image/gif image/png image/jpeg;
}
```

**注意**

`text/html` は常に圧縮される。重複していると警告が出るので書かない。

また、 `image/jpeg` などすでに圧縮されているフォーマットについては圧縮率が高くならない場合があり、 `gzip` することで削減できる通信量と、圧縮することによって増える負荷とのバランスを検討すべき。



#### gzip圧縮確認方法

HTTPレスポンスヘッダー内のContent-Encodingで
gzipが設定されていればgzip圧縮有効。

```bash
curl -I -H 'Accept-Encoding: gzip,deflate' http://url
```

### ファイルがある場合はファイルから、なければアプリサーバに投げる

https://qiita.com/kaikusakari/items/cc5955a57b74d5937fd8
```
location / {
    root /home/user/app/public/;
    try_files $uri $uri/ @dinamic;
}

location @dinamic {
    proxy_pass http://upstream;
}
```

GET /image/{id}.{ext}  に対する静的配信の例
ファイルは、 /home/user/private-isu/webapp/static/image/{id}.{ext} で保存す
https://github.com/yyamada12/private-isucon/commit/d22933069832b300fa7a387c370f6fdfe18cc82a

```
		location /image/ {
			root /home/user/private-isu/webapp/static/;
			try_files $uri @dinamic;
		}

		location @dinamic {
			internal;
			proxy_pass http://localhost:8080;
		}
```





## 例) 

### isucon7
https://github.com/yyamada12/isucon7_re2/commit/140bc086ad1cb48dd469f7296fd4044bd68f3f35

### isucon9

#### 変更前

```nginx
# /etc/nginx/nginx.conf

user www-data;
worker_processes 1;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

error_log  /var/log/nginx/error.log error;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server_tokens off;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 120;
    client_max_body_size 10m;

    access_log /var/log/nginx/access.log;

    # TLS configuration
    ssl_protocols TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';

    include conf.d/*.conf;
    include sites-enabled/*.conf;
}
```



```nginx
# /etc/nginx/sites-enabled/isucari.conf

server {
    listen 443 ssl;
    server_name isucon9.catatsuy.org;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:8000;
    }
}
```



#### 変更後

```nginx
# /etc/nginx/nginx.conf

user www-data;
worker_processes 1;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
worker_rlimit_nofile 100000;

error_log  /var/log/nginx/error.log error;

events {
	worker_connections 4096;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server_tokens off;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 120;
    client_max_body_size 10m;

		open_file_cache max=100 inactive=65s;
		gzip_static on;

		access_log off;

    # TLS configuration
    ssl_protocols TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';

	upstream app {
		server 127.0.0.1:8000;
	}
	upstream login_app {
		server 127.0.0.1:8000 weight=2;
		server 172.24.83.44:8080 weight=7;
		server 172.24.83.45:8080 weight=1;
	}

	server {
	    listen 443 ssl http2;
	    server_name isucon9.catatsuy.org;

	    ssl_certificate /etc/nginx/ssl/fullchain.pem;
	    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

		root /home/isucon/isucari/webapp/public;
		location /static/ {
			add_header Cache-Control "public max-age=86400";
		}
		location /upload/ {
			add_header Cache-Control "public max-age=86400";
		}

	    location / {
		proxy_pass http://app;
		proxy_set_header Host $host;
	    }
	    location /login {
		proxy_pass http://login_app;
		proxy_set_header Host $host;
	    }
	}
}
```

##  nginx -> go の接続を unix ドメインソケット化


https://github.com/yyamada12/isucon10_re3/commit/7659e80cd8d9d0ebe3aaab5afb2e3e6220c8b050


https://kaneshin.hateblo.jp/entry/2016/05/29/020302

https://qiita.com/gky360/items/dccb88f4aecd50970915#nginx---go-%E3%81%AE%E6%8E%A5%E7%B6%9A%E3%82%92-tcp-socket-%E3%81%8B%E3%82%89-unix-domain-socket-%E3%81%AB%E5%A4%89%E6%9B%B4


## too many open files 対策
too many open files が出てしまうと、それ以上 connection を貼れなくなってしまう。

変更するべき設定としては以下の3つ。

- OSの ulimit -n の値
- Nginx の worker_connections
- Nginx の worker_rlimit_nofile 

### OS の ulimit -n の値

 /etc/security/limits.conf に以下を追記して、サーバーを再起動 (`sudo reboot` する)

``` /etc/security/limits.conf
* soft nproc 65535
* hard nproc 65535
* hard nofile 65535
* soft nofile 65535
```

`ulimit -n` して値が65535になっていればOK


### Nginx のパラメータ
以下のように設定すればOK
worker数は woker_processes で決まる。
auto になっている時は、 `ps aux | grep nginx` でプロセス数を数えればOK。autoであればCPU のコア数と同じになるはず。

1connection あたり 2つのファイルディスクリプタが必要になるため、 worker_connections は worker_rlimit_nofile の半分になるように設定する

```
worker_rlimit_nofile {65535 / worker数};

events {
    worker_connections {worker_rlimit_nofile / 2};
}

```

https://qiita.com/mikene_koko/items/85fbe6a342f89bf53e89
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMzY1OTMyODIsLTI4MTIzMTc4MSwtOT
c3Njg0OTcxLC00OTMyNzgzNjQsMTAxOTg5MTA4NSwtMTAxNTc5
MzgxMywtNTQ5OTgxOTM1LDE4MTE3OTc1NDIsLTE3NzYzMzM3Nj
BdfQ==
-->