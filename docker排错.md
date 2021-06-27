##### docker 搭建LNMP时候出现的问题

首页当我们使用代理的时候 

- 要先看看端口是不是通的 telnet ip 端口
- 如果端口不通我们要看 云服务器上面的 安全组是否有开启
- 如果开启了还不行，在看看代理的ip是不是有问题
- 使用 ifconfig 查看下 docker0点桥 ip 这个是容器访问宿主机时候的ip，看看自己的代理ip是不是和这个一致

##### mysql 用户的问题

我们在top查看的时候 发现 mysqld的进程用户一直是polkitd，可本应该是mysql用户才对，为什么是这个用户呢

我们进入容器执行 

```
root@12350740270e:/# cat /etc/passwd | grep mysql
mysql:x:999:999::/home/mysql:/bin/sh
```

然后出容器在执行

```
[root@iZwz9j1nogjvoy83vrfkzeZ home]# cat /etc/passwd |grep 999
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
```

从上面可以看出，在容器内部用户为 mysql,它的用户ID为999，然后退出容器，在宿主机上，可以看到ID为999的用户ID对应的用户变成了polkitd。所以实际上内部和外部用的同一套用户，名字可能不同，但是ID用的是同一个，从而导致，ID虽然相同，但是用户不一致，从而权限也出现了差别

###### 	问题解决

- 修改 docker-compose.yml文件，将用户映射进去，一定要注意: /etc/passwd 也要映射进去，不然找不到用户
- 修改entrypoint.sh文件，将用户映射进去，可以看到，chown -R后面我把环境变量的用户给映射进去了，也就是用宿主机用户来初始化mysql

```
# allow the container to be started with `--user`
if [ "$1" = 'mysqld' -a -z "$wantHelp" -a "$(id -u)" = '0' ]; then
        _check_config "$@"
        DATADIR="$(_get_config 'datadir' "$@")"
        mkdir -p "$DATADIR"
        chown -R ${_USER}:${_USER} "$DATADIR"
        exec gosu ${_USER} "$BASH_SOURCE" "$@"
fi
```

- 重启容器