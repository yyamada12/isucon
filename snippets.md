

## init.sh

### ubuntuの場合

```
#!/bin/sh -eux

# 各種インストール
sudo apt-get install -y vim tmux htop dstat glances unzip

# alp インストール
sudo apt-get install -y unzip
mkdir -p ~/tmp
cd ~/tmp
wget https://github.com/tkuchiki/alp/releases/download/v0.3.1/alp_linux_amd64.zip
unzip alp_linux_amd64.zip
sudo install ./alp /usr/local/bin

# percona-toolkit のインストール
# sudo apt-get install -y percona-toolkit
# apt で入れるとバグのあるバージョンをインストールしてしまう模様
# なので以下のように自前でビルドする
cd ~/tmp
git clone https://github.com/percona/percona-toolkit.git
cd percona-toolkit
sudo perl Makefile.PL
make
make test
sudo make install
```

### centOSの場合

```
#!/bin/sh -eux

# 各種インストール
sudo yum install -y vim tmux htop dstat glances unzip

# alp インストール
mkdir -p ~/tmp
cd ~/tmp
wget https://github.com/tkuchiki/alp/releases/download/v0.3.1/alp_linux_amd64.zip
unzip alp_linux_amd64.zip
sudo install ./alp /usr/local/bin




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
alias vim='sudo vim'
alias g='sudo git'
alias ga='sudo git add'
alias gd='sudo git diff'
alias gs='sudo git status'
alias gp='sudo git push'
alias gb='sudo git branch'
alias gst='sudo git status'
alias gco='sudo git checkout'
alias gf='sudo git fetch'
alias gci='sudo git commit'
alias git='sudo git'
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
```





## pt-query-digest

isucon8でcentOSに入れたけど、

```
mkdir ~/bin
cd ~/bin
wget percona.com/get/pt-query-digest
chmod +x pt-query-digest
```

でいける説



## netdata

CPUとかブラウザから観れるらしいから入れてみる



## pprof

入れてみる

