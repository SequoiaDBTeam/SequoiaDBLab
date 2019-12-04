---
show: step
version: 1.0
enable_checker: true
---

# 使用 Docker 部署 SequoiaDB

## 实验介绍

为方便用户快速体验，SequoiaDB 巨杉数据库提供基于 Docker 的镜像。本实验将带领你一步步在Docker 环境下部署 SequoiaDB 分布式集群环境。

本实验基于以下内容进行改编：

* [SequoiaDB 巨杉数据库Docker镜像使用教程](http://blog.sequoiadb.com/cn/detail-id-97)

## 实验准备

#### 集群规划

我们准备在五个容器中部署一个多节点高可用 SequoiaDB 集群。如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449277128/wm)

集群包含一个协调节点与编目节点，三个三副本数据节点，与一个 MySQL 实例节点。

#### 实验环境

* Docker 环境：19.03.1
* 实验环境操作系统：Ubuntu 16.04
* 数据库版本：SequoiaDB 3.2.1
* 集群部署：一个运行协调和编目节点，三个运行数据节点，一个运行 MySQL 实例

Docker 在 Linux/Windows/MacOS 平台安装方法可参考官方文档。

## 拉取镜像

```
$ docker pull sequoiadb/sequoiadb
$ docker pull sequoiadb/sequoiasql-mysql
```

操作结果截图：

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449293514/wm)

## 启动四个 SequoiaDB 容器

```
$ docker run -it -d --name coord_catalog sequoiadb/sequoiadb:latest
$ docker run -it -d --name sdb_data1 sequoiadb/sequoiadb:latest
$ docker run -it -d --name sdb_data2 sequoiadb/sequoiadb:latest
$ docker run -it -d --name sdb_data3 sequoiadb/sequoiadb:latest
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449313626/wm)

查看四个容器的容器 ID：

```
$ docker ps -a | awk '{print $NF}'
```

运行结果：

```
NAMES
sdb_data3
sdb_data2
sdb_data1
coord_catalog
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449338894/wm)

## 查看四个容器的容器对应的 IP 地址

```
$ docker inspect coord_catalog | grep IPAddress |awk 'NR==2 {print $0}'
$ docker inspect sdb_data1 | grep IPAddress |awk 'NR==2 {print $0}'
$ docker inspect sdb_data2 | grep IPAddress |awk 'NR==2 {print $0}'
$ docker inspect sdb_data3 | grep IPAddress |awk 'NR==2 {print $0}'
```

四条命令的输出结果分别为各个容器自身的 IP 地址：

```
"IPAddress": "172.17.0.2",
"IPAddress": "172.17.0.3",
"IPAddress": "172.17.0.4",
"IPAddress": "172.17.0.5",
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449358902/wm)

## 部署 SequoiaDB 集群

根据集群规划以及各个容器的 IP 地址，在对应参数填入各自的地址与端口号。

```
$ docker exec coord_catalog "/init.sh" \
      --coord='172.17.0.2:11810' \
      --catalog='172.17.0.2:11800' \
      --data='group1=172.17.0.3:11820,172.17.0.4:11820,172.17.0.5:11820;group2=172.17.0.4:11830,172.17.0.5:11830,172.17.0.3:11830;group3=172.17.0.5:11840,172.17.0.3:11840,172.17.0.4:11840'
```

该命令输出结果为：

```
Begin generating SequoiaDB conf file
Finish generating SequoiaDB conf file
Restarting sdbcm process, it will take 10 seconds
Deploy...
Execute command: /opt/sequoiadb/tools/deploy/../../bin/sdb -f /opt/sequoiadb/tools/deploy/quickDeploy.js -e ''

************ Deploy SequoiaDB ************************
Create catalog: 172.17.0.2:11800
Create coord:   172.17.0.2:11810
Create data:    172.17.0.3:11820
Create data:    172.17.0.4:11820
Create data:    172.17.0.5:11820
Create data:    172.17.0.4:11830
Create data:    172.17.0.5:11830
Create data:    172.17.0.3:11830
Create data:    172.17.0.5:11840
Create data:    172.17.0.3:11840
Create data:    172.17.0.4:11840
```

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449372243/wm)

## 启动一个 MySQL 实例容器

```
$ docker run -it -d -p 3306:3306 --name mysql sequoiadb/sequoiasql-mysql:latest
```

查看启动容器的 ID

```
$ docker ps -a | awk '{print $NF}';
```

输出结果为包括 MySQL 实例在内的所有容器名：

```
NAMES
mysql
sdb_data3
sdb_data2
sdb_data1
coord_catalog
```

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449388348/wm)

查看容器 IP 地址

```
$ docker inspect mysql | grep IPAddress | awk 'NR==2 {print $0}'
```

输出结果为 MySQL 实例的 IP 地址：

```
"IPAddress": "172.17.0.6",
```

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449401446/wm)

## 将 MySQL 实例注册入协调节点

```
$ docker exec mysql "/init.sh" --port=3306 --coord='172.17.0.2:11810'
```

输出结果为：

```
Creating SequoiaSQL instance: MySQLInstance
Modify configuration file and restart the instance: MySQLInstance
Restarting instance: MySQLInstance
Opening remote access to user root
Restarting instance: MySQLInstance
Instance MySQLInstance is created on port 3306, default user is root
```

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449414099/wm)

## 本地登陆 MySQL 测试

```
$ mysql -h 127.0.0.1 -P 3306 -u root
```

可以得到 MySQL 连接成功的输出：

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.25 Source distribution

right (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449428955/wm)

用户可以使用 MySQL 命令创建数据库与表：

```
mysql> create database sample;
Query OK, 1 row affected (0.00 sec)

mysql> use sample;
Database changed
mysql> create table t1 (c1 int);
Query OK, 0 rows affected (0.59 sec)

mysql> show table status;
+------+-----------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+-------------+-------------+------------+-------------+----------+----------------+---------+
| Name | Engine    | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time | Update_time | Check_time | Collation   | Checksum | Create_options | Comment |
+------+-----------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+-------------+-------------+------------+-------------+----------+----------------+---------+
| t1   | SequoiaDB |      10 | Fixed      |    0 |              0 |           0 |   8796093022208 |       131072 |         0 |           NULL | NULL        | NULL        | NULL       | utf8mb4_bin |     NULL |                |         |
+------+-----------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+-------------+-------------+------------+-------------+----------+----------------+---------+
1 row in set (0.16 sec)
```

![图片描述](https://doc.shiyanlou.com/courses/uid1207281-20191204-1575449436849/wm)

## 重置镜像

为方便用户重置已经创建了数据库节点的容器，用户可以使用 cleanup.sh 脚本进行本地容器的重置。

```
$ docker exec mysql /cleanup.sh
$ docker exec coord_catalog /cleanup.sh
$ docker exec sdb_data1 /cleanup.sh
$ docker exec sdb_data2 /cleanup.sh
$ docker exec sdb_data3 /cleanup.sh
```

## 总结

为方便用户快速试用 SequoiaDB 分布式数据库，用户可直接拉取 SequoiaDB 的 Docker 镜像创建一个分布式集群。

该集群仅为测试使用，不可直接应用于生产环境。




