# snippets

## rotate_log.sh

```bash
#!/bin/sh

set -eux

if sudo [ -e $1 ]; then
  sudo mv $1 ${1%.*}_$(date +"%Y%m%d%H%M%S").${1##*.}
fi
```



## deploy.sh

```bash
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
make

# restart services
sudo systemctl restart mysql
sudo systemctl restart アプリのサービス
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


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYzNzI5MTMyM119
-->