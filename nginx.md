# nginx



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

_
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3NzYzMzM3NjBdfQ==
-->