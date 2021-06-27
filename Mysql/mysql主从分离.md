#### Mysql主从搭建，基于（docker）

#####        一 安装操作

Mysql 主从配置的流程大体如下

1. master会变动记录到 二进制日志（binlog）里面
2. master有一个I/O线程将二进制日志发送到slave
3. slave有一个I/O线程把 master发送的二进制写入到relay日志里面
4. slave有一个SQL线程，按照relay日志处理slave的数据

![](/Users/hpinke/Documents/img/mysql_master.jpg)

##### 	二 操作步骤

首先我们直接拉取最新的mysql 的最新镜像

`docker pull mysql`

然后启动此镜像的容器，这里需要分别启动主从两个容器

Master 主

`docker run --name mysql_master -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql`

Slave 从

`docker run --name mysql_slave1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql`

命令中 -p是映射端口 -e设置root用户密码 -d是后台启动

##### 	配置Master(主)

通过使用 `docker ps` 查看运行的容器

通过 `docker exec -it 容器ID /bin/bash` 命令进入容器

然后找到 my.cnf 文件然后对他进行编辑 如果无法使用 vim指令请执行

`apt-get update` 然后 `apt_get install vim`

这样我们就可以进入到my.cnf里面查看 

```
[mysqld]
## 同一局域网内注意要唯一 可以使用IP地址
server-id=100  
## 开启二进制日志功能，可以随便取（关键）
log-bin=mysql-bin
```

配置完成后可以重启容器 `docker restart 容器ID或容器名`

下一步在Master数据库创建数据同步用户，授予用户 slave REPLICATION SLAVE权限和REPLICATION CLIENT权限，用于在主从库之间同步数据

`CREATE USER 'slave'@'%' IDENTIFIED BY '123456';`

`GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';`

##### 	配置slave（从）

一样我们进入从服务器 找到my.cnf文件

```
[mysqld]
## 设置server_id,注意要唯一 可以使用IP
server-id=101  
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin   
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin  
```

然后重启从容器

##### 	连接主服务和从服务

- 进入Master里面的mysql 执行 `show master status;`记录下File和Position，等下要用。
- 然后在进入从服务器执行

```
change master to master_host='172.17.0.3',master_user='slave',master_password='123456', master_port=3306,master_log_file='mysql-bin.000002',master_log_pos= 156;
```

**master_host** ：Master的地址，指的是容器的独立ip,可以通过`docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称|容器id`查询容器的ip

**master_port**：Master的端口号，指的是容器的端口号

**master_user**：用于数据同步的用户

**master_password**：用于同步的用户的密码

**master_log_file**：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值

**master_log_pos**：从哪个 Position 开始读，即上文中提到的 Position 字段的值

**master_connect_retry**：如果连接失败，重试的时间间隔，单位是秒，默认是60秒

- 最后我们启动主从， `start slave`  然后 `show slave status \G;`查看结果

重点看两个参数 ==Slave_IO_Running== 和 ==Slave_SQL_Running== 这两个参数都是Yes的话说明主从配置完成。你也可以在主服务器上新建数据库，看从服务器有没有同步。

如果两个参数没有为Yes，可以排查下端口 使用ping, telnet等工具。