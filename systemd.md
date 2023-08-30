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

docker 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI3MDQ2MzE1NSw5ODk3MTE3MjFdfQ==
-->