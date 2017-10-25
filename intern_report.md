### 毕业论文结构简介
#### 吴凯
#### 2017.10.25



### 拟定章节
- 绪论
  - 课题研究的目的和意义
  - 预期创新点
  - 论文结构
- 工业远程监控系统简介
  - 基于工业以太网的远程监控系统
  - PLC 监控系统方案
  - 小结


- 现场数据采集结构设计
  - 硬件结构
  - 可靠性保障
  - 程序设计
  - 小结

  
- 监控数据处理与展示
  - 数据序列化与可视化原理
  - 序列回归算法简介
- 展望



### 绪论
- CDN 节点需要一个外部探测工具查看网络情况(like [17ce](http://www.17ce.com))
- DNS 需要一个外部探测工具(解析时间, 劫持检查等)



### 需求分析与开发选型
- QA 购买的外部机房资源不便完全开放运行定制任务
- 边缘节点的计算资源需要复用(自建CDN的资源，UU的资源, etc.)
  - 可参考方案: [Amazon Lambda@Edge](http://www.infoq.com/cn/news/2017/07/aws-lambda-at-edge)

- 开发与线上部署: Dockerized

Note: 会在靠近最终用户的AWS站点上自动运行和伸缩服务，实现高可用性, 由 cloudfront 事件触发; Docker 化开发原因: 统一依赖, 资源隔离




- 容器集群管理与调度: Docker Swarm
  - Docker Swarm Mode: Out of the box
  - 资源限制 + 服务升级与伸缩 + 负载均衡
Note: swarm mode 开箱即用, 方便部署, 横向扩展, 添加节点使用 swarm join 就好



- WebApp framework: flask; UI: vue.js
  - 前后端完全分离, 可分开部署
  - 用户可开发自定义 UI, 直接调用相关 API 即可
- DB: MySQL + MongoDB
- 消息队列: RabbitMQ + Celery
- 容器编排: Docker-Compose
Note: MySQL存储集群 + 任务 + 实例配置, Mongo 储存结果和输出



### 系统设计图示
![edge_user_chart](img/edge_user_chart.png)
<!-- .element height="100%" width="100%" -->
Note: 用户如何选择运行节点, 多次重复执行功能, 每轮聚合一次, 数据管道, 将数据打到其他地方



### 环境约束
- 服务创建与添加: 边缘节点的端口限制较严, 直接使用 Remote API 需对额外端口备案
  - 使用 SSH Login 创建服务

- 边缘节点资源限制 + 运行任务实际需求资源变化
  - 服务创建时的资源限制
  - 合理使用节点资源 -> 服务伸缩
  - 服务运行时的副本数调整



- 多线机房, 需要将服务创建后使用的网络和实际运行线路关联
- 备选方案:
  - 启动容器时指定使用 host network, 让容器具有跟宿主机一样的网络环境
    - 跳过 libnetwork, 直接和宿主机共享同一个 network namespace
    - 对主机网络有完全访问权
    - 不同容器之间, 容器宿主机之间, 会产生资源竞争和冲突
Note: veth pair, 一端放在新的 namespace, 另一端放在宿主机的 namespace 连接物理网络设备




- 创建 scope 为 swarm 的 bridge 网络
  - `scope=swarm`: 广播 swarm manager 节点间网络配置
  - 依然是 bridge network, 相对安全
  - 配合策略路由在 rt_tables 添加路由表即可, 比较简单
- 最终选择: Bridge Network + 策略路由



#### 配置示例:
``` shell
# 创建 scope 为 swarm 的 bridge network
docker network create --config-only --subnet 192.168.50.0/24 \
--gateway 192.168.50.1 tel_net_50
docker network create -d bridge --scope swarm \
--config-from tel_net_50 edge_tel

# 添加路由表
vim /etc/iproute2/rt_tables
# Add
150 docker_tel

# 为路由表添加规则
ip route add via 219.135.99.193 dev eth2 table 150
# 对于来自此子网的数据使用 150 路由表的规则
ip rule add from 192.168.50.0/24 table 150
```




### 开发流程
- Past: 完全基于轮询 DB 创建任务及其实例.
  - 每个实例对应一个 swarm service, 大量的dead container -> 资源浪费
  - 一次实例对应一次 SSH Login
  - 需要对 service 及其容器定期 gc
  - TOO SLOW!



- Now: 基于 MQ 的服务创建与实例执行
  - 一个服务对应一个队列
  - 服务一次性创建, 启动后只监听相应队列, 在线服务数与任务多少无关
  - 灵活添加新服务或升级原服务
  - celery 设置多任务并发
  - service 副本数可调
  - speed boosted!



### DEMO



#### 性能测试: 响应时间测试
- 机器资源: staging server x 3
  - Xeon Intel Xeon E312xx, 2.4 GHz x 2; 4G RAM

- 主服务: WebApp + Rabbitmq + MySQL
  - gunicorn + gevent: worker * 5

  - 300 users in 30s
  - min_wait: 1000ms; max_wait: 5000ms




主服务器情况:
<img src="img/0826/webapp_htop.png" />
<img src="img/0826/webapp_iftop.png" />
边缘节点情况:
<img src="img/0826/edge_exec_htop.png" />



Locust Chart:

<img src="img/0826/total_2.png" width="700" height="500" />



- Failure Rate:
  - 同时在线用户超过 150 个时, 错误率超过 1%
  - Connection reset by peer为主
- 可能原因分析: 单点 DB 频繁查询压力较大, 响应时间基本和 QPS 成正比



### 展望
- 服务的资源限制优化:
  - Now: CPU + mem: limit(运行时资源阈值)
  - Planning:
    - 增加 reserve 检查(运行前资源需求下限检查)
    - 增加更多限制维度: net bandwidth, disk io params, etc.
    - 获取节点的硬件配置(资产信息)和资源使用情况: Galaxy + Gpostd



- 服务创建的自动化流程打通:
  - 自动创建代码与 Docker 仓库, 更加方便用户定制并创建服务

- 监听未完成任务的速度优化



### Q & A
Note: 轮询优化, celery_beat_schedule 导致间隔时间无法灵活根据任务要求调整;
