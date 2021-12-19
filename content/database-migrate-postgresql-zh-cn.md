---
title: "Database Migrate Postgresql"
subtitle: ''
date: 2021-12-14T19:29:16+08:00
lastmod: 2021-12-14T19:29:16+08:00
draft: true
author: "gkirito"
authorLink: "https://www.gkirito.com"
description: ""
images: ""
tags: []
categories: []
twemoji: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: ""
hiddenFromHomePage: false
rssFullText: true
hiddenFromSearch: false
featuredImage: ""
featuredImagePreview: ""
toc:
  enable: true
math:
  enable: true
comment:
  enable: true
lightgallery: false
license: ""
---

## 前沿







### 创建迁移容器

1. 在K8s集群上创建迁移同步容器

   ![image-20211214CWgISY5V@2x](https://libget.com/gkirito/blog/image/2021/image-20211214CWgISY5V@2x-20211214193743923.png)

​	在启动命令中加入运行命令写入保活命令

​	类似：

```shell
/bin/sh,-c,while true; do echo hello world; sleep 1; done
```

之后创建容器

2. 配置容器内环境

   通过Kubectl连入上一步创建容器

```bash
 kubectl exec -it database-migration-xxx-xxx  --namespace='XX' -- /bin/bash
```

开始安装环境

由于我们使用的是比较基础的ubuntu镜像，需要工具需要安装一下

```bash
apt-get update
apt-get install git vim curl sudo gettext-base
```

下载脚本(建议在home目录下)

```bash
mkdir /home/gkirito
cd /home/gkirito
git clone https://github.com/bluegroundltd/postgresql-migration-guide
cd postgresql-migration-guide/
```

修改vars.sh

```shell
export BUCARDO_SYNC_NAME=database_migration_2021_XXX
export BUCARDO_LOCAL_PASSWORD=postgres
export BUCARDO_OLD_HOSTNAME=xx.xxx.xx.xx
export BUCARDO_OLD_USERNAME=postgres
export BUCARDO_OLD_PASSWORD=password
export BUCARDO_OLD_DATABASE=
export BUCARDO_NEW_HOSTNAME=coming-im-prod.cinuzspwpcip.us-east-2.rds.amazonaws.com
export BUCARDO_NEW_USERNAME=postgres
export BUCARDO_NEW_PASSWORD=9rHAzvg7rdKKXzpGAKio8XmR
export BUCARDO_NEW_DATABASE=wallet
# export APP1_PASS=postgres
# export APP1_RO_PASS=postgres
# export APP2_PASS=postgres
# export APP2_RO_PASS=postgres
```

修改install.sh

```bash
#!/bin/bash -ex

#
# Note: Need to run this as root
#

[ -z $BUCARDO_LOCAL_PASSWORD ] && echo "Plase set BUCARDO_LOCAL_PASSWORD" && exit 1

# Install prerequisites
apt update
apt install -y \
        cpanminus \
        gcc \
        libdbi-perl \
        libpq-dev \
        make \
        postgresql \
        postgresql-client-common \
        postgresql-plperl-12

# start postgresql service
service postgresql start
# Create bucardo user in the local Postgresql
sudo -u postgres psql -w postgres <<EOF
CREATE USER bucardo WITH LOGIN SUPERUSER ENCRYPTED PASSWORD '$BUCARDO_LOCAL_PASSWORD';
EOF

# Create bucardo database in the local Postgresql
sudo -u postgres psql -w postgres <<EOF
CREATE DATABASE bucardo;
EOF

# Install Perl modules required by Bucardo
cpanm CGI
cpanm DBD::Pg --force
cpanm DBIx::Safe
cpanm Encode::Locale

# Download and install Bucardo from source
curl -L https://github.com/bucardo/bucardo/archive/5.5.0.tar.gz -o bucardo.tar.gz
tar -zxvf bucardo.tar.gz
pushd bucardo-5.5.0
perl Makefile.PL
make
make install
popd

mkdir -p /var/run/bucardo
mkdir -p /var/log/bucardo
```

在第33行，在cpanm DBD::Pg后添加 –force，用来防治安装完后校验

完成以上修改后，把vars.sh中的环境变量导入shell环境中，执行

```bash
source vars.sh
```

执行安装脚本

```bash
./install.sh
```

安装完成后连接本地同步器所用postgresql数据库

```bash
psql -U bucardo -w
```

如果出现错误，无法连接，但是确定自己输入的密码正确

说明postgresql没有配置好，本地连接无法连接上

修改postgresql设置

```bash
vim /etc/postgresql/12/main/pg_hba.conf
```

到文件最下方，修改配置如下

```shell
# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     md5
host    all             bucardo         127.0.0.1/32            trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```

在之后的运行中出现过错误

根据这个[issues](https://github.com/bucardo/bucardo/issues/133)，需要在配置中加入

host    all             bucardo         127.0.0.1/32            trust

之后重启数据库服务，并检查能否使用psql登录bucardo账号

```bash
service postgresql restart
```

执行

```bash
bucardo install
```

安装bucardo，并根据提示配置

![image-20211214uf0tY19P@2x](https://libget.com/gkirito/blog/image/2021/image-20211214uf0tY19P@2x.png)

将Host改为127.0.0.1

之后输入P，显示Installation is now complete. 说明完成安装

![image-20211214KWvLw8ye@2x](https://libget.com/gkirito/blog/image/2021/image-20211214KWvLw8ye@2x.png)

然后修改setup_new_database.template文件

```bash
create database signal;
create extension plperl; /* Required by Bucardo */

/* 
create user app1 with login encrypted password '$APP1_PASS';
grant usage on schema public to app1;
grant select, update, delete, insert on all tables in schema public to app1;
grant connect, temp on database bgdb to app1;
grant execute on all functions in schema public to app1;
grant usage, select, update on all sequences in schema public to app1; 
*/
```

修改需要创建的数据库明，如果不需要创建其他账户可以注释之后的内容

修改configure.sh

```bash
#!/bin/bash -ex
DUMPFILE="mydb-$(date +%s).dump"
export BUCARDO_DEBUG=4

[ -z $BUCARDO_SYNC_NAME ] && echo "Please set BUCARDO_SYNC_NAME" && exit 1

[ -z $BUCARDO_OLD_HOSTNAME ] && echo "Please set BUCARDO_OLD_HOSTNAME" && exit 1
[ -z $BUCARDO_OLD_USERNAME ] && echo "Please set BUCARDO_OLD_USERNAME" && exit 1
[ -z $BUCARDO_OLD_PASSWORD ] && echo "Please set BUCARDO_OLD_PASSWORD" && exit 1
[ -z $BUCARDO_OLD_DATABASE ] && echo "Please set BUCARDO_OLD_DATABASE" && exit 1

[ -z $BUCARDO_NEW_HOSTNAME ] && echo "Please set BUCARDO_NEW_HOSTNAME" && exit 1
[ -z $BUCARDO_NEW_USERNAME ] && echo "Please set BUCARDO_NEW_USERNAME" && exit 1
[ -z $BUCARDO_NEW_PASSWORD ] && echo "Please set BUCARDO_NEW_PASSWORD" && exit 1
[ -z $BUCARDO_NEW_DATABASE ] && echo "Please set BUCARDO_NEW_DATABASE" && exit 1

# Setup an alias for bucardo the explicitly specifies the local password so we don't have to retype it each time
bucardo_cmd() {
        bucardo -h localhost -p 5432 -P $BUCARDO_LOCAL_PASSWORD $@
}
export -f bucardo_cmd

# Create the .pgpass file in order for postgres CLI to not ask for password each time
echo "" > ~/.pgpass
cat >> ~/.pgpass <<EOF
$BUCARDO_OLD_HOSTNAME:5432:*:$BUCARDO_OLD_USERNAME:$BUCARDO_OLD_PASSWORD
$BUCARDO_NEW_HOSTNAME:5432:*:$BUCARDO_NEW_USERNAME:$BUCARDO_NEW_PASSWORD
EOF
# Set correct permissions on the .pgpass file
chmod 0600 ~/.pgpass

# Create the database and the user accounts in the new (empty) database
envsubst < setup_new_database.template > setup_new_database.sql
psql -h ${BUCARDO_NEW_HOSTNAME} -U ${BUCARDO_NEW_USERNAME} -f setup_new_database.sql postgres

# Setup Bucardo multi-master replication
# XXX: Unfortunately Bucardo does not read the .pgpass file,
#      so we must explicitly specify the password in the command.
bucardo_cmd add db source_db \
        dbhost=$BUCARDO_OLD_HOSTNAME \
        dbport=5432 \
        dbname=$BUCARDO_OLD_DATABASE \
        dbuser=$BUCARDO_OLD_USERNAME \
        dbpass=$BUCARDO_OLD_PASSWORD
bucardo_cmd add db target_db \
        dbhost=$BUCARDO_NEW_HOSTNAME \
        dbport=5432 \
        dbname=$BUCARDO_NEW_DATABASE \
        dbuser=$BUCARDO_NEW_USERNAME \
        dbpass=$BUCARDO_NEW_PASSWORD

# List the bucardo databases
echo "The bucardo databases are:"
bucardo_cmd list databases

# Setup bucardo to replicate all tables and all sequences
bucardo_cmd add table all --db=source_db --herd=my_herd
bucardo_cmd add sequence all --db=source_db --herd=my_herd

# Remove any unused tables or the ones that don't have indexes
bucardo_cmd remove table public.table_foo
bucardo_cmd remove table public.table_bar
bucardo_cmd remove table public.databasechangelog

# Setup bucardo dbgroups
bucardo_cmd add dbgroup my_group
bucardo_cmd add dbgroup my_group source_db:source
bucardo_cmd add dbgroup my_group target_db:source

# Transfer the database schema
pg_dump -v -h $BUCARDO_OLD_HOSTNAME -U $BUCARDO_OLD_USERNAME --schema-only $BUCARDO_OLD_DATABASE --file=schema.sql
psql -h $BUCARDO_NEW_HOSTNAME -U $BUCARDO_NEW_USERNAME -d $BUCARDO_NEW_DATABASE -f schema.sql

# Setup bucardo sync (autokick=0) ensures that nothing is transfered yet
bucardo_cmd add sync $BUCARDO_SYNC_NAME herd=my_herd dbs=my_group autokick=0

# Migrate the data using compression in order to minimize file size. Make sure
# that you don't transfer Bucardo's data or anything else that is not managed by
# you.
echo "Dumping data to compressed file"
time pg_dump -v \
    -U $BUCARDO_OLD_USERNAME \
    -h $BUCARDO_OLD_HOSTNAME \
    --file=$DUMPFILE \
    -N bucardo -N schema_baz \
    -Fc \
    $BUCARDO_OLD_DATABASE
echo "Restoring data from compressed file"
# Treat the new database as replica until the data restoration is over, to avoid
# re-running triggers. `-j 8` is the parallelization factor, you can change it
# according to your system's number of CPUs.
export PGOPTIONS='-c session_replication_role=replica'
time pg_restore -v \
    -U $BUCARDO_NEW_USERNAME \
    -h $BUCARDO_NEW_HOSTNAME \
    -j 8 \
    --data-only \
    -d $BUCARDO_NEW_DATABASE \
    $DUMPFILE
export PGOPTIONS=''

# Reset autokick flag and start continuous multi-master replication
echo "Starting Bucardo"
bucardo_cmd start
bucardo_cmd update sync $BUCARDO_SYNC_NAME autokick=1
bucardo_cmd reload config
bucardo_cmd restart
```

在60行下根据提示插入没有主键的表，没有主键的表无法使用bucardo迁移，一般public.databasechangelog都没有主键

修改uninstall.template

```
DROP TRIGGER bucardo_delta_namemaker ON bucardo.bucardo_delta_names;

DROP TRIGGER bucardo_note_trunc_$BUCARDO_SYNC_NAME ON table_1;
DROP TRIGGER bucardo_delta ON table_1;
DROP TRIGGER bucardo_kick_$BUCARDO_SYNC_NAME ON table_1;

DROP TRIGGER bucardo_note_trunc_$BUCARDO_SYNC_NAME ON table_2;
DROP TRIGGER bucardo_delta ON table_2;
DROP TRIGGER bucardo_kick_$BUCARDO_SYNC_NAME ON table_2;

DROP TRIGGER bucardo_note_trunc_$BUCARDO_SYNC_NAME ON table_2_lock;
DROP TRIGGER bucardo_delta ON table_2_lock;
DROP TRIGGER bucardo_kick_$BUCARDO_SYNC_NAME ON table_2_lock;

DROP TRIGGER bucardo_note_trunc_$BUCARDO_SYNC_NAME ON payment;
DROP TRIGGER bucardo_delta ON payment;
DROP TRIGGER bucardo_kick_$BUCARDO_SYNC_NAME ON payment;

DROP TRIGGER bucardo_note_trunc_$BUCARDO_SYNC_NAME ON booking;
DROP TRIGGER bucardo_delta ON booking;
DROP TRIGGER bucardo_kick_$BUCARDO_SYNC_NAME ON booking;
DROP SCHEMA bucardo CASCADE;

```

把所有涉及到的表都填上去

之后执行configure.sh

如果中途重复修改执行configure.sh，会导致bucardo中可能已经创建了一些database和XX，可以用一下命令删除

```bash
bucardo -h localhost -p 5432 -P postgres remove db source_db --force
bucardo -h localhost -p 5432 -P postgres remove db target_db --force
bucardo -h localhost -p 5432 -P postgres remove dbgroup my_group
bucardo -h localhost -p 5432 -P postgres remove sync $BUCARDO_SYNC_NAME
```

同时删除同步的新库，重新执行configure.sh

如果删除新库发现已有连接，可以在新库执行以下命令

```sql
SELECT
    pg_terminate_backend(pid)
FROM
    pg_stat_activity
WHERE
    pid <> pg_backend_pid()
    AND datname = 'signal';
```



等configure.sh执行完成后，





