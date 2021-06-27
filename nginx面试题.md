##### nginx 惊群现象

在说nginx，先来看看什么是“惊群”？ 简单说来，多进程/多线程 等待同一个socket事件，当这个事件发生时，这些线程/进程 被同时唤醒，就是惊群。 可以想象，效率很低下，许多进程被内核重新调度唤醒，同时去相应这一个事件，当然只有一个进程能处理事件成功，其他进程在处理该事件失败后重新休眠（也有其他选择）。这种性能浪费现象就是惊群。

简单了说，就是同一时刻，只允许一个 nginx worker 在自己的epoll中处理监听句柄。它的负载均衡 也很简单，当达到最大connection 的7/8时，本woker不会去试图拿accept锁，也不会去处理新连接，这样其他nginx worker进程就更有机会去处理监听句柄，建立新连接了。而且，由于timeout的设定，使得没有拿到锁的worker进程，去拿锁的频繁更高。

##### Nginx 与 fpm的方式，主要是两种工作方式

Tcp socket: tcp socket通信方式，需要在nginx配置文件中填写php-fpm运行的ip地址和端口号

```
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME 			$document_root$fastcgi_script_name;;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
}
```

Unix socket: unix socket通信方式，需要在nginx配置文件中填写php-fpm运行的pid文件地址。

```
//service php-fpm start生成.sock文件
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
}
```



##### nginx上传文件大小的限制，修改命令

`client_max_body_size`  10M 设置上传文件大小限制



##### php+nginx 默认配置，突然收到大量请求，服务器开始报错，哪个错误比较多？502 503 504

- 502原因：php-fpm配置文件错误，或者是接口请求有问题
- 503原因：机器维护或者暂时不可用
- 504原因：一般是nginx配置有问题



##### fpm的平滑重启过程

通过观察，可以分析出大致的平滑重启过程为：

1. Master 使用新配置 fork出n-1个worker及新master
2. 新worker处理新请求，旧worker执行完退出
3. master重新加载配置，期间使用新master 接管服务
4. master加载配置完毕，新master 切换为worker 工作模式 平滑重启完，master 进程号并不会发生变化

##### Nginx的错误码

502:一般是fpm配置有问题。比如请求超时。max_children 和 request_terminate_timeout。max_children 最大子进程数 超过了php-fpm的最大响应数，就会出现502 .netstat 可以查看连接数。一个php-cgi 消耗的内存在20M左右，php-cgi所占用的内存为 20M*max_request数量。为单个请求的超时时间（request_terminate_timeout）.当数据库连接超时，或者大量请求超时的时候，会出现502.

504一般是NGINX配置有问题。Gateway Time-out。fastcgi_connect_timeout、fastcgi_send_timeout、fastcgi_read_timeout、fastcgi_buffer_size、fastcgi_buffers、fastcgi_busy_buffers_size、fastcgi_temp_file_write_size、fastcgi_intercept_errors。fgi缓冲区太小，会导致504. 或者是程序无故退出， 或者是程序执行太慢。 proxy_read_timeout 60 proxy_send_timeout 60

##### 如果包含正在处理的进程，会报什么错误 【百度面试】

```
<?php
	exec("sleep 5");
	echo 'done';
```

会报502 process_control_timeout 设置子进程接受主进程复用信号的超时时间，可以解决这个问题

##### Nginx的 max_requests

Max_requests意味着，子进程处理多少请求之后，就会关闭。因为子进程差不多都能同时打满，同时关闭，所以会出现502的问题。 默认max_request为500个。可以修改这个参数。或者增加机器的内存，修改参数。

##### Nginx的负载均衡

Ip hash 根据ip进行hash可以解决session问题

```
upstream backserver {
		ip_hash;
		server 192.168.0.14:88;
		server 192.168.0.15:80;
}
```

轮询（默认的方式）

```
upstream backserver {
		server 192.168.0.14;
		server 192.168.0.15;
}
```

权重

```
upstream backserver {
		server 192.168.0.14 weight=3;
		server 192.168.0.15 weight=7;
}
```

Fair 根据响应时间来进行分配， 响应时间短的优先分配

```
upstream backserver {
		server server1;
		server server2;
		fair;
}
```



##### nginx与fpm通信（php-fpm进程master进程和worker进程分别的责任是什么）

动态程序，将请求发送给php-fpm，fpm的master启动worker去执行phpcgi

然后将编译结果返回fcgi执行原理

1. Web server 启动时载入fastcgi进程管理器（IIS ISAPI 或 Apache Module）
2. FastCGI 进程管理器自身初始化，启动多个CGI解释器进程（可见多个php-cgi）并等待来自 web server的连接
3. 当客户端请求到达 web server时，FastCGI进程管理器选择连接到一个CGI解释器。web server 将CGI 环境变量和标准输入发送到 FastCGI子进程php-cgi
4. Fastcgi 子进程完成处理后 将标准输出和错误信息从同一连接返回 web server。当fastcgi子进程关闭连接时，请求便告处理完成。fastcgi子进程接着等待并处理来自 fastCGI进程管理器（运行在web server中）的下一个连接。而在CGI模式中，php-cgi在此便退出了。在上述情况中，你可以想象CGI通常有多慢。每一个web 请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI,所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接可以工作

Fastcgi 与传统的cgi模式的区别之一则是web 服务器不是直接执行cgi程序了，而是通过socket与fastcgi响应器（fastcgi进程管理器）进行交互，也正是由于fastcgi进程管理器基于socket通信的，所以也是分布式的，web服务器可以和cgi响应器服务器分开部署，web服务器需要将数据cgi/1.1的规则封装在遵循fastcgi协议包中发送给fastcgi响应器程序。

##### cgi协议流程

##### 

##### fpm

fpm的master通过【共享内存】与worker进行通信，同时监听worker的状态，已处理请求数。要杀死一个worker通过发送信号的方式来实现。fpm的master进程与worker进程之间不会直接进行通信，master通过共享内存获取worker进程信息，比如worker进程当前状态，已处理请求数等，当master进程要杀掉一个worker进程时则通过发送信号的方式通知worker进程

##### fpm进程管理

介绍下三种不同的进程管理方式：

1. Static: 静态模式 这种方式比较简单，在启动时master按照pm.max_children配置fork出相应数量的worker进程，即worker进程数是固定不变的
2. Dynamic: 动态进程管理, 首先在fpm启动时按照pm.start_servers初始化一定数量的worker，运行期间如果master发现空间worker数低于pm.min_spare_servers配置数（表示请求比较多，worker处理不过来了）则会fork worker 进程，但总的worker数不能超过pm.max_children如果master发现空闲worker数超过了pm.max_spare_servers(表示闲着的worker太多了)则会杀掉一些 worker ,避免占用过多资源，master通过这4个值来控制worker数
3. ondemand: 这种方式一般很少用，在启动时不分配worker进程，等到有请求了后再通知master进程fork worker进程，总的worker数不超过pm.max_children，处理完成后worker进程不会立即退出，当空闲时间超过pm.process_idle_timout 后再退出

worker会将自己的状态更新到fpm_scoreboard_proc_s->request_stage

Master就是通过这个字段来判断worker是否是空闲

##### PHP 的四种工作模式

- Cgi 通用网关接口 （common gateway interface）
- Fast-cgi 常驻（long-live）型的CGI
- Cli 命令行运行 （command line interface）
- Mod_php模式（apache等web服务器运行的模块模式）

##### 遇到过一个问题，开多少进程合适？

看硬件资源来决定

##### Lvs 和 nginx的区别，都是四层的一个负载均衡

Lvs : 重量级的四层负载软件，应该就是直接进行转发

Nginx:  轻量级的四层负载软件，带缓存功能，正则表达式较为灵活。可以进行限流，重定向。



##### Restart 和 reload的区别

Reload ----重新加载，reload 会重新加载配置文件，Nginx服务不会中断，而且reload时会测试conf语法等，如果出错会rollback用上一次正确配置文件保持正常运行。

Restart----重启（先stop后start）会重启Nginx服务。这个重启会造成服务一瞬间的中断，如果配置文件出错会导致服务启动失败，那就是更长时间的服务中断了。所以，如果是线上的服务，修改的配置文件一定要备份。为了保证线上服务高可用，最好使用reload

##### 



















