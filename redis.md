# redis
## install
ubuntuの場合

aptのデフォルトだとバージョンが古い可能性が高い

```
sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt update
```
としてバージョンをあげた後、
```
sudo apt install redis-server
```
でOK

後はsystemctlを叩く
```
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

## 外部接続
/etc/redis/redis.conf で bindの設定を変更する
```
bind 0.0.0.0
```

## 初期データ読み込み
### /initializeでのRedisデータ初期化

ベンチ開始時に /initialize にアクセスが来て、そこで初期データ以外のデータは削除されるのですが、そのタイミングでRedisのデータも整合性を取って初期化する必要があります。

そこで普通にMySQLを読んでRedisに書き込みをすると30秒の時間制限を超えてしまうので、MySQLの初期データに対応するRedisの初期データセットを作った上で  `redis-cli config set appendoonly yes`  にしてaofファイルを吐き出しておき、initalize時にはそれを redis-cli で読み込んで一気にロードしました。2秒で終わります。

```
$redis->flushall();
system("/usr/bin/redis-cli --pipe < /home/isucon/appendonly.aof")
    if -e "/home/isucon/appendonly.aof";
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjIyMjc0NTA2LC0xMTYyMDQ1NjQ2LC0xND
M3MDc2MDVdfQ==
-->