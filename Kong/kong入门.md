## 前沿

最近有在学习和了解`kong`，今天就和大家来分享下`kong`和`konga`吧



## 介绍

`kong` 是一款基于Nginx_lua 模块写的高可用网关API，在客户端和服务间转发API通信的网关，可以通过插件扩展功能。

简单一句话：**kong是动态增强版的nginx**

看看几个名词

- `Nginx` 是模块化设计的反向代理软件，C语言开发
- `OpenResty` 是以`Nginx` 为核心的web 开发平台，可以解析执行Lua 脚本
- `kong` 是一个`openResty`应用，一个api gateway



## kong安装

主要以`docker` 方式来部署。当前`kong`版本`2.3.0`

1. **创建docker 虚拟网络**

```
docker network create kong-net
```

2. **运行postgresql 的数据库**

```
docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               -e "POSTGRES_PASSWORD=kong" \
               postgres:9.6
```

3. **初始化数据库（迁移数据）**

```
docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     kong:latest kong migrations bootstrap
```

4. **运行kong**

```
docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 8000:8000 \
     -p 8443:8443 \
     -p 127.0.0.1:8001:8001 \
     -p 127.0.0.1:8444:8444 \
     kong:latest
```

5. **验证是否安装成功**

```
//在宿主机上运行
curl -i http://localhost:8001/
```

返回下面内容

```
HTTP/1.1 200 OK
Date: Mon, 05 Apr 2021 08:37:24 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 10452
X-Kong-Admin-Latency: 20215
Server: kong/2.3.0

............
```

表示安装正确，可以正常使用kong



## kong的运用

kong 可以动态的生成 `nginx`的配置文件，下面我们来演示一下

**一个典型的nginx配置文件**

```
upstream demoupstream {
		server localhost:3000 weight=100;
}
server {
		listen 8000;
		server_name  demo.com;
		location /demo {
				proxy_pass http://demoupstream;
		}
}
```

上面的简单`Nginx`配置，可以转为如下**kong 配置**

1. **配置 Upstream** 

```
curl -X POST http://localhost:8001/upstreams --data "name=demoupstream"
```

2. **配置 target**

```
curl -X POST http://localhost:8001/upstreams/demoupstream/targets --data "target=localhost:3000" --data "weight=100"
```

3. **配置 service**

```
curl -X POST http://localhost:8001/services --data "name=demo" --data "host=demoupstream" --data "url=http://demo.com/"
```

4. **配置 route**

```
curl -X POST http://localhost:8001/services/demo/routes --data "paths[]=/demo" --data "hosts[]=demo.com"
```

当然啦，你可以吧8000端口改为熟悉的80端口，只要在启动kong的时候 将映射端口修改下就可以了。

5. **验证**

```
curl -i http://localhost:8000/demo --header host:demo.com
```

⚠️注意：**上面的target端口要本地可访问的，域名hosts记得配置**



**概念术语介绍**

- **upstream** : 是对上游服务器的抽象
- **target**：代表了一个物理服务，是 IP + port 的抽象
- **service**：是抽象层面的服务，他可以直接映射到一个物理服务（host 指向 ip + port）,也可以指向一个 `upstream` 来做到负载均衡
- **route**：是路由的抽象，他负责将实际的 `request` 映射到 `service`

**Kong 默认监听的端口**

- **8000** ： 监听来自客户端的`HTTP` 流量，转发到你的 `upstream` 服务上
- **8443** ： 监听`HTTPS`的流量，功能跟 `8000`一样，可以通过配置文件禁止。
- **8001** ： `kong`的`HTTP`监听的 admin api 管理接口
- **8444** ： `kong`的`HTTPS`监听的 admin api 管理接口

## 

## konga 

上面都是命令行的，有没有什么直观的图形界面呢？

有的，比较火的就是 konga 了，官方是收费，我们可以用社区贡献者的

[konga地址](https://github.com/pantsel/konga)

我们以 docker 来安装

1. **Konga 也需要创建数据库，这里我们创建和迁移数据一起完成**

```
docker run --rm pantsel/konga:latest -c prepare -a postgres -u postgresql://kong:kong@192.168.1.102:5432/konga
```

解释下参数 `kong:kong@192.168.1.102:5432/konga`  `用户名：密码@数据库ip:端口/数据库名称`

2. **启动konga**

```
docker run -p 1337:1337 \
        --network kong-net \
        --name konga \
        -e "NODE_ENV=production"  \
        -e "DB_ADAPTER=postgres" \
        -e "DB_URI=postgresql://kong:kong@192.168.1.102:5432/konga" \
        pantsel/konga
```

这边的参数就比较简单了，具体可以看上面官方链接

3. **验证**

```
浏览器访问 http://localhost:1337/
```

注册进入就可以看到如下界面了

![](../image/konga.jpg)

图中的Name 可以随意起， kong Admin URl 就是访问kong的admin api 里面填 `http://kong:8001`

激活之后就可以看到你前面创建的 上游，服务，路由这些啦。

大家玩耍起来吧，还希望看更多的可以访问 kong 官方网站接着学习

## 致谢

感谢你看完这篇文章，有什么不对的地方欢迎指出，谢谢🙏