# Systemd

ISUCONではsystemdを使ってgoの実行ファイルをサービスとして登録している。

goでprintデバッグしたければ、ここのログを見れば良い。



## conf

設定ファイルは`xx.service` というファイル名で  `/etc/systemd/system/` に配置される。

標準出力をファイルに出力するには以下のように設定する。

`StandardOutput = file:/path/to/logfile`

## ログ

ログを見るには以下のコマンド。

-f は tail -fと同じオプション。

`sudo journalctl -f -u service名`

*例*
`sudo journalctl -f -u isucari.golang` 


## Dockerはがし
アプリケーションの実行にdockerが利用されている場合にgoの実行ファイルを利用する

docker の build の時間分、deploy のスピードが上がる
アプリケーションのスピードも上がるはずだが、dockerも結構速いのでスコアはそこまで変わらないかも

- ExecStart を実行ファイルに書き換える
- ExecStop を `/bin/kill -s QUIT $MAINPID` に書き換える
- EnvironmentFile で環境変数を指定する

ex)
Before
```
[Unit]
Description=isucon12 qualify webapp
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
Type=simple
User=isucon
Group=isucon
WorkingDirectory=/home/isucon/webapp
ExecStart=docker compose -f docker-compose-go.yml up --build
ExecStop=docker compose -f docker-compose-go.yml down
Restart=always
```

After
```
[Unit]
Description=isucon12 qualify webapp
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
Type=simple
User=isucon
Group=isucon
WorkingDirectory=/home/isucon/webapp/go
EnvironmentFile=/home/isucon/webapp/env.sh
ExecStart=/home/isucon/webapp/go/isuports
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=always
```

env.sh
```
ISUCON_DB_HOST=127.0.0.1
ISUCON_DB_PORT=3306
ISUCON_DB_USER=isucon
ISUCON_DB_PASSWORD=isucon
ISUCON_DB_NAME=isuports
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0Mzg4MTU2MDUsOTI4Nzg5NDk2LDk4OT
cxMTcyMV19
-->