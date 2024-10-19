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
   - `als`  実行した上でslack


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjE1NzU1NzkyLC0xMzkwNDI2MjEyXX0=
-->