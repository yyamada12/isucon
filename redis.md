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

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MzcwNzYwNV19
-->