## 独自に設定している項目

### ツール系のインストール
以下のリポジトリで自動インストール用スクリプトを管理
https://github.com/yyamada12/isucon-settings

- install_essentials
各種設定ファイルと、git, vim, tmux のinstall, 自動化スクリプトの配置 などを行う

- install_tools
alp, pt-query-digest, netdata のinstallなどを行う

### alias
~/.alias.bash に配置
 
#### alp
nginx の log format を変更し、 /var/log/nginx/access.log に出力している前提
deploy script で deploy 時に access.log -> access_bak.log に mv する
   - `al`: ~/alp.yml の設定でalpを実行。対象は /var/log/nginx/access.log
   - `al_bak` 対象を /var/log/nginx/access_bak.log で実行
   - `als`  実行した上でslackに送信
   - `als_bak` access_bak.log の結果をslackに送信



#### pt-query-digest
mysql の slow log を設定し、 /var/log/mysql/slow.log に出力している前提
deploy script で deploy 時に slow.log -> slow_bak.log に mv する
  - `pt`  pt-query-digest を --limit 10 -format profile,query_report で実行し、 less に流す。対象は /var/log/mysql/slow.log
  - `pt_bak` 対象を/var/log/mysql/slow_bak.log にして実行
  - `pts` 実行した上でslackに送信
  - `pts_bak` slow_bak.log の結果をslackに送信

#### pprof
main.go に pprof + fgprof を設定し、 /initialize を契機に ~/pprof/pprof.pb.gz, ~/pprof/fgprof.pb.gz に出力している前提  
deploy script で deploy 時に pprof.pb.gz -> pprof_bak.pb.gz, fgprof.pb.gz -> fgprof_bak.pb.gz に mv する  
ssh config で、ssh時に localhost:1234, localhost:1235 を port forward させる
  - `pp` ~/pprof/pprof.pb.gz に対して pprof を実行し、  localhost:1234 で結果をhosting
  - `pp_bak` 対象を ~/pprof/pprof_bak.pb.gz にして実行
  - `pps` png にoutput してslackに送信
  - `pps_bak` pprof_bak.pb.gz の結果をslackに送信

- fgprof
  - `fgp` ~/pprof/fgprof.pb.gz に対してpprofを実行し、localhost:1235で結果をhosting
  - `fgp_bak` fgprof_bak.pb.gz に対して実行


- `deploy`
	- ~/deploy.sh を実行する
- `applog`
  - sudo journalctl で app の log を表示する
  - applog -f で垂れ流しにさせる想定

- S


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYwMzM5NzcwMiwtNDI0MTA3Myw2MTU3NT
U3OTIsLTEzOTA0MjYyMTJdfQ==
-->