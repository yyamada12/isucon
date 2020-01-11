# Systemd

ISUCONではsystemdを使ってgoの実行ファイルをサービスとして登録している。

goでprintデバッグしたければ、ここのログを見れば良い。



## conf

ISUCON9では `/etc/systemd/system/isucari.golang.service` に設定ファイルがあった。

標準出力をファイルに出力するには以下のように設定する。

`StandardOutput = file:/path/to/logfile`

## ログ

ログを見るには以下のコマンド。

-f は tail -fと同じオプション。

`sudo journalctl -f -u isucari.golang.service`