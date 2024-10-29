# ISUCON チェックリスト

## 前日までにやること

- [ ] レギュレーションを読み込んでおく
- [ ] 前年のマニュアルを読み込んでおく
- [ ] github リポジトリを作っておく
  - [ ] 2台目、3台目用のgitコマンドを作っておく

## 開始直前にやること

- [ ] CloudFormation による構築準備 (前年度のマニュアル読んで手順確認&AWS マネコンを開いておく)
- [ ] 前回のマニュアルを開いておき、VSCodeで差分を比較できるようにしておく
- [ ] ssh/configを開いておき、編集できる状態にしておく
  - [ ]  ssh/config のIdentityFileを、github に登録している鍵のパスに設定しておく
- [ ] local で go の sandbox を開いておき、簡単な実装の動作確認が手元でできるようにしておく
- [ ] slackのAPI Token 画面を用意しておく

## 開始直後

### 環境構築

- [ ] ssh/config

```
Host isu1
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
  LocalForward  11234 localhost:1234
  LocalForward  19999 localhost:19999
Host isu2
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
  LocalForward  21234 localhost:1234
  LocalForward  29999 localhost:19999
Host isu3
  HostName ホスト名
  IdentityFile ~/.ssh/鍵名
  User isucon
  LocalForward  31234 localhost:1234
  LocalForward  39999 localhost:19999
```

- [ ]  itermの1つのタブを3分割し、3つのサーバーに入って Cmd + Opt + i で全サーバーに同一コマンドを実行できるようにする

- [ ] 初期のsshユーザーがubuntuなどになっている場合は、authorized_keysをisuconユーザーのホームディレクトリにコピーしてisuconユーザーでsshできるようにする
```
sudo -u isucon mkdir -p /home/isucon/.ssh
sudo cp /home/ubuntu/.ssh/authorized_keys /home/isucon/.ssh/authorized_keys
sudo chown isucon:isucon /home/isucon/.ssh/authorized_keys
```

- [ ] ツールのインストール
https://github.com/yyamada12/isucon-settings
  
- [ ] 必須ツールと設定 
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/yyamada12/isucon-settings/master/install_essentials.sh)"
```

- [ ] 計測ツール

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/yyamada12/isucon-settings/master/install_tools.sh)"
```

arm ( mac 上の multipass で素振りする時はこっち)
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/yyamada12/isucon-settings/master/install_tools_arm.sh)"
```

- [ ] 環境変数設定
↑の計測ツールのinstallの最後で設定する

| 環境変数 | 設定内容 |
|--|--|
| APP_DIR | アプリケーションのディレクトリ |
| APP_BUILD_CMD | go build or make or else |
| APP_SERVICE_NAME | isucon-go.service 的なやつ |
| SLACK_TOKEN | https://api.slack.com/apps/A05GTSY2MKJ/oauth の User OAuth Token|

指定した環境変数は ~/.bash_profile で管理される
( ~/set_env.sh 参照)

- [ ] webサーバが何か確認する

```bash
sudo systemctl list-units --type=service --state=running
```

- [ ] 言語をgoに変更する

- [ ] gitにsshできるようにする
- 既に存在する鍵を利用する
```
scp ~/.ssh/git_rsa isu1:.ssh/id_rsa
scp ~/.ssh/git_rsa isu2:.ssh/id_rsa
scp ~/.ssh/git_rsa isu3:.ssh/id_rsa
```

- ssh できることを確認
```
ssh -T git@github.com
```


- [ ] レポジトリ作る
    - ホームディレクトリを git root とする
    -  `du -sh *` で大きなファイルが有る場合はgitignoreする
    - `GOPATH` 配下もgitignoreすべし

```.gitignore
.*
!.gitignore
/pprof
```


- [ ] etcをレポジトリに追加する
  - /etcからファイルを移動し、シンボリックリンクをはる
  -  mysqlはシンボリックリンクだと設定が反映されないため、デプロイスクリプトで ~/etc から /etc にコピーする運用とする

```
~/manage_etc_files.sh
```
- mysql は設定ファイルの場所がまちまちなので気をつける
以下のファイルの場合もあり
```
sudo cp /etc/my.cnf /etc/my.cnf.org
sudo cp /etc/my.cnf ~/etc/
sudo chmod 666 ~/etc/my.cnf
```

- benchが動くことを一応確認しておく

ベンチ実行して問題なければOK

- [ ] レポジトリを他のサーバーにpullする
2台目、3台目のサーバーで以下の手順を実施
```
git init
git remote add origin レポジトリURL
git fetch origin main
git reset --hard origin/main
git branch -M main
git branch --set-upstream-to=origin/main main
```

2台目、3台目でデプロイしてベンチが通ればOK


## VSCode Remote 開発環境構築

- [ ] remote ssh につないだ状態で拡張機能 Goを追加する
- [ ] 不要なファイルの追跡を止める
```
{
    "files.watcherExclude": {
        "**/webapp/nodejs/**": true,
        "**/webapp/perl/**": true,
        "**/webapp/php/**": true,
        "**/webapp/python/**": true,
        "**/webapp/ruby/**": true,
        "**/webapp/rust/**": true,
        "**/bin/**": true,
        "**/pkg/**": true,
        "**/src/**": true,
    },
    "files.exclude": {
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

→ watcherExcludeは上手く効かないことも多い
その場合は、workディレクトリを作成し、VSCodeで編集するファイルだけシンボリックリンク を貼っておく
```
mkdir ~/work
cd ~/work
ln -s ~/etc
ln -s ~/.gitignore
ln -s ~/alp.yml
ln -s ~/.bashrc
ln -s ~/.alias.bash
ln -s ~/deploy.sh
```

- [ ] remote に Go, GitHub Copilot の拡張を追加する
GitHub Copilot は認証まで済ませておく

- [ ] remote用のsettings.jsonにGOROOT, GOPATHを必要であれば設定する

```json
{
    "go.goroot": "/home/isucon/local/go",
    "go.gopath": "/home/isucon/go:/home/isucon/webapp/go/"
}
```

※ import文で指定したパッケージはGOPATHが通っている箇所であればコンパイラが見つけることができ、go get により外部のパッケージをダウンロード・インストールする場合はGOPATHの１つ目のPATHのみ有効となる


go.modの場合、VSCodeで開いたフォルダに go.workを配置しないと、サブディレクトリにあるgoモジュールを認識してくれない
```
go work init ./webapp/golang
```
https://qiita.com/chanhama/items/a21ca7d5cd43d6f3f90d#comment-55530e230c93991ef5c0


## 計測

### alp

- [ ] そもそもbenchがnginxを通っているかどうか確認する
通っていなければ、nginx をリバプロとして設定する
https://github.com/yyamada12/isucon10/commit/76ce0248358e683dc261a694e51472583984e395

- [ ] nginxのログフォーマットを変更する

```nginx.conf
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


- [ ] alp.yml の設定で path parameter のあるAPIをまとめる
/xxx/:id
/xxx/:id/yyy
のような場合は、 /xxx/:id を最後に記載するとうまくいく

(ex)
```
- '/api/user/.+/theme'
- '/api/user/.+/statistics'
- '/api/user/.+/icon'
- '/api/user/.+'
```


### slow log
- [ ] mysql でslow log を設定
```
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 0
```


- [ ] mysql再起動

デプロイスクリプト回す

- [ ]  mysql で以下のコマンドを実行し、設定が変わっていることを確認
```
show variables like  'slow_query%';
show variables like  'long%';
```



### pprof
benchを回した時に initializeが呼ばれるときに30秒間のprofileを取得して以下の場所に出力するように設定する
```
/home/isucon/pprof/pprof.pb.gz
/home/isucon/pprof/fgprof.pb.gz
```

- [ ] コードに埋め込む

```
import (
        _ "net/http/pprof"
        "github.com/felixge/fgprof"
)

func main() {
    http.DefaultServeMux.Handle("/debug/fgprof", fgprof.Handler())
    go func() {
        fmt.Println(http.ListenAndServe("localhost:6060", nil))
    }()
```

- [ ] fgprof を go mod に追加
```
go mod tidy
```

- [ ] initializeに仕込む
```
go  func() {
  if out, err := exec.Command("go", "tool", "pprof", "-seconds=30", "-proto", "-output", "/home/isucon/pprof/pprof.pb.gz", "localhost:6060/debug/pprof/profile").CombinedOutput(); err != nil {
    fmt.Printf("pprof failed with err=%s, %s", string(out), err)
  } else {
    fmt.Printf("pprof.pb.gz created: %s", string(out))
  }
}()
go  func() {
  if out, err := exec.Command("go", "tool", "pprof", "-seconds=30", "-proto", "-output", "/home/isucon/pprof/fgprof.pb.gz", "localhost:6060/debug/fgprof").CombinedOutput(); err != nil {
    fmt.Printf("fgprof failed with err=%s, %s", string(out), err)
  } else {
    fmt.Printf("fgprof.pb.gz created: %s", string(out))
  }
}()
```

go の パスが通っていない場合はenv.shでPATHを指定するか、goの絶対パスを指定する

- [ ] ビルド

デプロイスクリプトを回す


## とりあえずやっとく
### /etc/mysql/my.cnf
- [ ] mysql8対策
```
disable-log-bin 
innodb_doublewrite = 0 
innodb_flush_log_at_trx_commit = 0
```

### too many open files 対策
1: ulimit の数を上げる
/etc/security/limits.conf
```
* soft nproc 65535
* hard nproc 65535
* hard nofile 65535
* soft nofile 65535
```

`sudo reboot`

`ulimit -n` して値が65535になっていればOK


2: nginx.conf をいじる

worker数は woker_processes で決まる。
auto になっている時は、 `ps aux | grep nginx` でプロセス数を数えればOK。autoであればCPU のコア数と同じになるはず。

```
worker_rlimit_nofile {65535 / worker数};
events {
    worker_connections {worker_rlimit_nofile / 2};
}
```

worker数が2の場合
```
worker_rlimit_nofile 32768;
events {
    worker_connections 16384;
}
```

worker数が4の場合
```
worker_rlimit_nofile 16384;
events {
    worker_connections 8192;
}
```

### 再起動対策

```
	// db.Open() が成功した直後にこれを入れる.
	for {
		err := db.Ping()
		if err == nil {
			break
		}
		log.Print(err)
		time.Sleep(time.Second * 2)
	}
	log.Print("DB ready!")
```


### DBへのCRUDの可視化
https://github.com/mazrean/isucrud/
```
go install github.com/mazrean/isucrud@latest
```
isucrud ./...


## やることなくなったら

### app
- [ ] コネクションプールの数を増やしてみる
- [ ] GOMAXPROCS の値を確認する

### nginx

- [ ] error log を確認して対応する
- [ ] 秘伝のタレのパラメータ試してみる

### mysql
- [ ] 秘伝のタレのパラメータ試してみる
- [ ] mysqltuner試してみる

## 再起動試験
- [ ] ベンチ実施後、全台で `sudo reboot` を実行する
- [ ] 起動後、アプリを操作し、ベンチによる変更が保存されていることを確認する

## 提出前

- [ ] slowlog を切る
- [ ] nginx のアクセスログの出力をオフにする (http > `access_log off;` )
- [ ] netdata を切る
```sudo systemctl stop netdata```
```sudo systemctl disable netdata```
or
```/usr/libexec/netdata/netdata-uninstaller.sh --yes --env /etc/netdata/.environment```

- [ ] appのログを切る (go の middlewareのコード削除など)
- [ ] VSCode Remote SSH をサーバーから削除する (**Remote-SSH: Uninstall VS Code Server from Host...**)
https://code.visualstudio.com/docs/remote/troubleshooting#_cleaning-up-the-vs-code-server-on-the-remote


```
kill -9 $(ps aux | grep vscode-server | grep $USER | grep -v grep | awk '{print $2}')
rm -rf ~/.vscode-server # Or ~/.vscode-server-insiders
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNDgyOTE0MjEsMTcyNzIyNjgxNiwtMT
UzNzc2ODQ3Myw0MTQ0Nzg1NzMsNDEzNDIxNTU3LC0zOTA2MDc1
MTksLTg1NDEyMzY2MywtMTU3ODE0NzcwNSwtOTc0MjcwMDU3LC
0xMDM3MjgwNjkyLC04OTM2MDk0MzQsMTk1ODgxNjU5Miw1NzMx
NTcxNDAsMjEwNzM3NzAwMCwxMTM4MTYyMzM2LC0xNDI3OTAzMD
gsLTc3ODkwMzU2NiwyMTM1ODI3MTIzLC0xODkxOTc1NDc3LDE0
NTg5Mzg2NDZdfQ==
-->