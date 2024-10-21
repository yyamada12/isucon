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
 
 - alp
   - `al`: ~/alp.yml の設定でalpを実行。対象は /var/log/nginx/access.log
   - `al_bak` 対象を /var/log/nginx/access_bak.log で実行
   - `als`  実行した上でslackに送信
   - `als_bak` access_bak.log の結果をslackに送信

-  pt-query-digest
  - `pt`  pt-query-digest を --limit 10 -format profile,query_report で実行し、 less に流す。対象は /var/log/mysql/slow.log
  - `pt_bak` 対象を/var/log/mysql/slow_bak.log にして実行
  - `pts` 実行した上でslackに通知
  - `pts_bak` slow_bak.log の結果をslackに通知

- pprof
  - `pp` ~/pprof/pprof.pb.gz に対して pprof を実行し、  localhost:1234 で結果をhosting
  - `pp_bak` 対象を ~/pprof/pprof_bak.pb.gz にして実行
  - `pps` png にoutput してslackに送信
  - `pps_bak` pprof_bak.pb.gz の結果をslackに送信


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEwNTM1NzgxLDYxNTc1NTc5MiwtMTM5MD
QyNjIxMl19
-->