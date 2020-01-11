

## init.sh

### ubuntuの場合

```
#!/bin/sh -eux

# 各種インストール
sudo apt update -y
sudo apt upgrade -y
sudo apt install -y vim tmux htop dstat glances unzip
# alp インストール
mkdir -p ~/tmp
cd ~/tmp
wget https://github.com/tkuchiki/alp/releases/download/v0.3.1/alp_linux_amd64.zip
unzip alp_linux_amd64.zip
sudo install ./alp /usr/local/bin

# percona-toolkit のインストール
mkdir -p ~/tmp
cd ~/tmp
wget percona.com/get/pt-query-digest
sudo install ./pt-query-digest /usr/local/bin
```



## rotate_log.sh

```
#!/bin/sh

set -eux

if sudo [ -e $1 ]; then
  sudo mv $1 ${1%.*}_$(date +"%Y%m%d%H%M%S").${1##*.}
fi
```



## deploy.sh

```
#!/bin/sh

set -eux

SCRIPT_DIR=$(dirname "$0")

cd $SCRIPT_DIR

date -R
echo "Started deploying."

# rotate logs
./rotate_log.sh /var/log/nginx/access.log
./rotate_log.sh /var/log/nginx/error.log

./rotate_log.sh /var/log/mysql/mysqld.log
./rotate_log.sh /var/log/mysql/slow.log

# build go app
cd ..
go build app.go
cd $SCRIPT_DIR

# restart services
sudo systemctl restart mysql
sudo systemctl restart isuxi.go
sudo systemctl restart nginx

date -R
echo "Finished deploying."
```



## etc gitignore

```
/*
!.gitignore
!nginx/
!mysql/
mysql/debian.cnf
!sysctl.conf
!my.cnf
!my.cnf.d/
```



## app gitignore

```
/webapp/nodejs/
/webapp/python/
/webapp/perl/
/webapp/php/
/webapp/ruby/
/webapp/go/vendor/
```



## bashrc

```
alias g='git'
alias ga='git add'
alias gd='git diff'
alias gs='git status'
alias gp='git push'
alias gb='git branch'
alias gst='git status'
alias gco='git checkout'
alias gf='git fetch'
alias gci='git commit'
```



## gitconfig

```
[color]
  ui = auto
[alias]
  co = checkout
  ci = commit
  st = status
  bra = branch
  gr = log --all --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
[user]
  name = yyamada12
  email = 12yacropolisy@gmail.com
```



## tmux.conf

```
# マウススクロールをよしなに
set-option -g mouse on
bind -T root WheelUpPane   if-shell -F -t = "#{alternate_on}" "send-keys -M" "select-pane -t =; copy-mode -e; send-keys -M"
bind -T root WheelDownPane if-shell -F -t = "#{alternate_on}" "send-keys -M" "select-pane -t =; send-keys -M"
set -g @plugin 'nhdaly/tmux-better-mouse-mode'

# プリフィックスキー
set -g prefix C-q
unbind C-b

# 設定ファイルをリロードする
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# キーストロークのディレイを減らす
set -sg escape-time 1

# ウィンドウのインデックスを1から始める
set -g base-index 1

# ペインのインデックスを1から始める
setw -g pane-base-index 1

# Vimのキーバインドでペインを移動する
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
bind -r C-h select-window -t :-
bind -r C-l select-window -t :+

# Vimのキーバインドでペインをリサイズする
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# コピーモードをvimのキーバインドにする
set-window-option -g mode-keys vi

# アクティブなウィンドウを目立たせる
setw -g window-status-current-fg white
setw -g window-status-current-bg red
setw -g window-status-current-attr bright

# ペインボーダーの色を設定する
set -g pane-border-fg green
set -g pane-border-bg black

#デフォルトシェルをbashにする
set-option -g default-shell /bin/bash
set-option -g default-command /bin/bash


```

