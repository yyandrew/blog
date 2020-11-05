---
layout: post
title: postgres小抄
comments: true
categories: postgres
---
## 常用操作

* 进入psql终端

    ```sh
    psql -U username dbname
    ```
* 创建新数据库

    ```sh
    su postgres # 切换posgresql管理员
    psql # 进入终端

    postgres=# create database "new_db" owner "owener_name"; # 创建新数据库new_db，所有者是owner_name
    ```
* 表操作

    ```sh
    dbname=# \d tablename; # 显示表的结构
    dbname=# \du; # 显示所有用户
    dbname=# ALTER TABLE tablename ADD COLUMN newcolumn integer; # [字段类型](http://www.postgresql.org/docs/9.3/static/datatype.html#DATATYPE-TABLE)
    dbname=# CREATE USER new_user with password 'new password'; # 创建一个新用户
    dbname=# ALTER ROLE new_user with Replication; # 为用户new_user增加Replication权限
    dbname=# ALTER TABLE tablename DROP COLUMN oldcolumn; # 删除字段
    dbname=# ALTER TABLE tablename ALTER COLUMN last_updated_at SET DEFAULT now(); # 设置默认值
    dbname=# SELECT * FROM pg_indexes WHERE TABLENAME='your_table'; # 显示your_table的索引
    dbname=# EXPLAIN ANALYZE SELECT * FROM your_table WHERE name='andrew'; # 显示查询详细步骤
    dbname=# SELECT to_tsvector('english', 'a fat  cat sat on a mat - it ate a fat rats'); # 转换一句话转换成tsvector格式
    ```

* 重新启动postgres(Mac OSX系统，brew安装)

    ``` sh
    launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist # 停止postgres服务
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist # 启动postgres服务
    tail -f /usr/local/var/postgres/server.log # postgresql的日志文件
    ```

* 重新启动postgres(ubuntu)

    ``` sh
    /etc/init.d/postgresql restart
    ```

* 重新初始化一个数据库集群

    ``` sh
    initdb /usr/local/var/postgres -E utf8 --locale=zh_CN.UTF-8 --lc-collate=zh_CN.UTF-8
    ```

* 重新导入数据库文件

    ``` sh
    pg_upgrade -b /usr/local/Cellar/postgresql/9.4.5/bin -B /usr/local/Cellar/postgresql/9.6.1/bin/ -d /usr/local/var/postgres9.4/postgres94 -D /usr/local/var/postgres
    ```

* 数据库备份及恢复

    ``` sh
    pg_dump -U username -d database_name -t table_name > ~/sites.sql # 导出单独的表
    psql -U username -d database_name < sites.sql # 导入单独的表
    ```

## 流备份配置

* 主服务器的IP是:`1.2.3.4`
* 从服务器的IP是:`5.6.7.8`
* Postgresql的版本为：9.5.14

#### 首先设置主服务器的postgres
1. 创建一个备份的用户

    ```
    sudo -u postgres psql -c "CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'replicatorpassword';"
    ```

2. 修改`/etc/postgresql/9.5/main/postgresql.conf`

    ```
    listen_address = 5.6.7.8
    wal_level = hot_standby
    max_wal_senders = 3
    wal_keep_segments = 8
    ```

3. 修改`/etc/postgresql/9.5/main/pg_hba.conf`

    ```
    hostssl replication     replicator      5.6.7.8/32            md5
    ```
    > 注意:`5.6.7.8`是从服务器的IP地址

#### 接下来设置从服务器的postgres

1. 修改`/etc/postgresql/9.5/main/postgresql.conf`

    ``` sh
    wal_level = hot_standby
    max_wal_senders = 3
    wal_keep_segments = 8
    hot_standby = on
    ```

2. 接下来运行下面的命令

    ```
    sudo service postgresql stop
    sudo -u postgres rm -rf /var/lib/postgresql/9.5/main
    sudo -u postgres pg_basebackup -h 1.2.3.4 -D /var/lib/postgresql/9.5/main -U replicator -v -P
    sudo -u postgres bash -c "cat > /var/lib/postgresql/9.5/main/recovery.conf <<- _EOF1_
      standby_mode = 'on'
      primary_conninfo = 'host=1.2.3.4 port=5432 user=replicator password=replicatorpassword sslmode=require'
      trigger_file = '/tmp/postgresql.trigger'
    _EOF1_
    "
    sudo service postgresql start
    ```

#### 检查备份是不是正常运行

在主服务器上运行下面命令`sudo -u postgres psql -x -c "select * from pg_stat_replication;"`

#### 将数据库导出为gzip及导入

```
sudo -u postgres pg_dump database | gzip > database.sql.gz
dropdb database && createdb database
gunzip < database.sql.gz | psql database
```
