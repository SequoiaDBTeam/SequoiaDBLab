#SequoiaDB巨杉数据库的安装以及使用 MySQL shell进行增删改查操作

下载 SequoiaDB 最新数据库安装包，并上传到目标主机上  
`$ wget --content-disposition http://download.sequoiadb.com/cn/sequoiadb_latest`

解压SequoiaDB安装包  
`$ tar zxvf sequoiadb-3.4-linux_x86_64.tar.gz`

转到文件夹  
`$ cd sequoiadb-3.4/`

增加可执行权限  
```
$ chmod u+x sequoiadb-3.4-linux_x86_64-installer.run
$ chmod u+x sequoiasql-mysql-3.4-linux_x86_64-installer.run
$ chmod u+x sequoiasql-postgresql-3.4-x86_64-installer.run
$ chmod u+x setup.sh
```

运行安装脚本  
`$ sudo ./setup.sh`

按照步骤一步步往下
![](https://ftp.bmp.ovh/imgs/2019/12/aee0543dc3bbf5d6.png)  
![](https://ftp.bmp.ovh/imgs/2019/12/6b4d336b14ec6d67.png)  
![](https://ftp.bmp.ovh/imgs/2019/12/45a25e5edac3d7bf.png)  
![](https://ftp.bmp.ovh/imgs/2019/12/f84d1089df160baa.png)  
![](https://ftp.bmp.ovh/imgs/2019/12/642a5e309cb9894e.png)

#使用MySQL shell进行操作

使用sdbadmin用户登录主机
```
$ su sdbadmin
password : sdbadmin
```

登录MySQL shell  
`$ /opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root`

创建数据库实例  
`mysql> create database cs;`  
`mysql> use cs;`

创建表  
`mysql> create table cl(a int, b int, c text, primary key(a, b) ) ;`

使用SQL语句进行增删改查操作  
```
mysql> insert into cl values(1, 101, "SequoiaDB test");
mysql> insert into cl values(2, 102, "SequoiaDB test");
mysql> select * from cl order by b asc;
mysql> update cl set c="My test" where a=1;
mysql> delete from cl where b=102;
mysql> select * from cl order by b asc;
```
