---
layout: post
title: 配置PostgreSQL流备份
date: 2020-11-05 16:56 +0800
---

假设我们有两台数据库服务器：

1. 主数据库服务器为：172.20.0.2(等价于docker-compose里的pg_master，其5432端口映射为本机54322端口)
2. 从数据库服务器为：172.20.0.3(等价于docker-compose里的pg_slave，其5432端口映射为本机54321端口)

#### 运行docker容器

```
docker-compose up
```

#### 设置主数据库

1. 设置主数据库监听地址

```
bash-5.0# psql -U postgres
postgres=# ALTER SYSTEM SET listen_addresses TO '*';
```

2. 创建一个用来复制的postgresql用户

```
su - postgres
createuser --replication -P -e replicator
exit
```

3. 为主数据库启用复制功能
   修改`/var/lib/postgresql/data/pg_hba.conf`，添加一行，内容如下：

```
host    replication     replicator      172.20.0.3/24     md5
```

#### 设置从数据库

1. 备份从数据库上的数据

```
su - postgres
cp -r /var/lib/postgresql/data /var/lib/postgresql/data_orig
```

2. 将主数据库的数据同步到从数据库

```
rm -r /var/lib/postgresql/data/* && \
  pg_basebackup -h 172.20.0.2 -D /var/lib/postgresql/data -U replicator -P -v  -R -X stream -C -S pgstandby1
```

3. 从输出可以看到

```
pg_basebackup: initiating base backup, waiting for checkpoint to complete
pg_basebackup: checkpoint completed
pg_basebackup: write-ahead log start point: 0/2000028 on timeline 1
pg_basebackup: starting background wal receiver
pg_basebackup: created replication slot "pgstandby1"
24335/24335 kb (100%), 1/1 tablespace
pg_basebackup: write-ahead log end point: 0/2000138
pg_basebackup: waiting for background process to finish streaming ...
pg_basebackup: syncing data to disk ...
pg_basebackup: renaming backup_manifest.tmp to backup_manifest
pg_basebackup: base backup completed
```

出现了**created replication slot "pgstandby1"** 就表示已经从数据库已经创建成功

#### 测试主从备份是否正常工作

1. 查看pg_slave服务器上的数据库`cf`的表

   ```
   postgres@localhost:postgres> \l cf;
   +--------+---------+------------+-----------+---------+---------------------+
   | Name   | Owner   | Encoding   | Collate   | Ctype   | Access privileges   |
   |--------+---------+------------+-----------+---------+---------------------|
   +--------+---------+------------+-----------+---------+---------------------+
   SELECT 0
   ```

   不出意外，没有找到`cf`这个数据库，因为`cf`还有创建。

2. 在pg_master服务器创建一个`cf`数据库

   ```
   postgres@localhost:postgres> CREATE DATABASE cf;         
   postgres@localhost:postgres> use cf;
   postgres@localhost:cf> CREATE TABLE store_informations (store_name VARCHAR(40), sales integer)
   postgres@localhost:cf> INSERT INTO store_informations VALUES ('a', 12)
   ```

   数据库创建成功。

3. 再次查看pg_slave服务器上的数据库`cf`的表

   ```shell 
   postgres@localhost:postgres> \l cf;                        
   +--------+----------+------------+------------+------------+---------------------+
   | Name   | Owner    | Encoding   | Collate    | Ctype      | Access privileges   |
   |--------+----------+------------+------------+------------+---------------------|
   | cf     | postgres | UTF8       | en_US.utf8 | en_US.utf8 | <null>              |
   +--------+----------+------------+------------+------------+---------------------+
   SELECT 1
   postgres@localhost:postgres> use cf; 
   postgres@localhost:cf> SELECT * FROM store_informations;   
   +--------------+---------+
   | store_name   | sales   |
   |--------------+---------|
   | a            | 12      |
   +--------------+---------+
   SELECT 1
   ```

   可以看出数据已经从主数据库(pg_master)同步到从数据库(pg_slave)了。备份功能已经实现了。

   ![stream-replication-in-posgres.png](/images/stream-replication-in-posgres.png)

