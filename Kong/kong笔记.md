#### 服务网关

- 抽象 group project 概念进行管理
- 基于路径做路由 /group/project/service-path
- 默认ip限制， 可选的upstream hmac 鉴权
- 服务发现依赖 Consul



#### Kong的配套体系

- Yapi 解决api文档管理，mock数据，自动化测试功能
- Consul 服务发现 配置中心



#### 核心

动态的生成 nginx的配置文件