### EDGE -- 边缘节点通用计算平台
#### 吴凯
#### 导师: 甘建勋
#### 2017.08.28



### Topics
- 起源
- 需求分析
- 开发选型
- 约束与对应措施
- 开发流程简述
- 性能测试
- 展望
- Q & A



### 起源
- CDN 节点需要一个外部探测工具查看网络情况(like [17ce](www.17ce.com))
- DNS 需要一个外部探测工具
Note: DNS 探测是要取得什么数据?



### 需求分析与开发选型
- QA 购买的外部机房资源不便完全开放来运行定制任务
- 边缘节点的计算资源需要复用(自建CDN的资源，UU的资源, etc.)
  - 可参考方案: [Amazon Lambda@Edge](http://www.infoq.com/cn/news/2017/07/aws-lambda-at-edge)

- 开发与线上部署: Dockerized.

Note: 会在靠近最终用户的AWS站点上自动运行和伸缩代码，实现高可用性 Docker 化开发原因: 统一依赖, 资源隔离




- 容器集群管理与调度: Docker Swarm
  - Docker Swarm Mode: Out of the box
  - 资源限制 + 服务平滑升级 + 负载均衡
Note: swarm mode 开箱即用, 方便部署, 横向扩展, 添加节点, swarm join



- WebApp framework: flask; UI: vue.js
  - 前后端完全分离, 可分开部署
  - 用户可使用自定义 UI, 直接调用相关 API 即可
- DB: MySQL + Mongo
- 消息队列: RabbitMQ + celery
- 容器编排: Docker-Compose
Note: MySQL: Cluster + Job + Instance 配置, Mongo 储存结果和输出



### 系统设计图示
![edge_user_chart](img/edge_user_chart.png)
<!-- .element height="100%" width="100%" -->



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
    - 直接和宿主机共享同一个 Network Namespace
    - 对主机网络有完全访问权
    - 不同容器之间使用相同端口会产生冲突



- 创建 scope 为 swarm 的 bridge 网络
  - `scope=swarm`: 广播 swarm manager 节点间网络配置
  - 依然是 bridge network, 相对安全
  - 配合策略路由在 rt_tables 添加路由表即可, 比较简单
- 最终选择: Bridge Network + 策略路由



### 开发流程
- Past: 完全基于轮询 DB 创建任务及其实例.
  - 每个实例对应一个 swarm service, 大量的dead container -> 资源浪费
  - 一次实例对应一次 SSH Login
  - 需要对 service 及其容器定期 gc
  - TOO SLOW!



- Now: 基于 MQ 的服务创建与实例执行
  - 一个服务对应一个 queue
  - 服务一次性创建, 启动后只监听 queue, 服务数与任务无关
  - 灵活添加新服务或更改原服务
  - celery 设置多任务并发
  - service 副本数可调
  - speed boosted!



### DEMO



#### 性能测试 Part1: 响应时间测试
- 机器资源: staging server x 3
  - Xeon Intel Xeon E312xx, 2.4 GHz x 2; 4G RAM

- 主服务: WebApp + Rabbitmq + MySQL
  - gunicorn + gevent: worker * 5



- 响应时间测试
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
  - Connection reset by peer
- 原因分析: 单点DB查询压力较大, 响应时间基本和 QPS 成正比



### 展望
- 服务的资源限制优化:
  - Now: CPU + mem: limit(运行时资源阈值)
  - Planning:
    - 增加 reserve 检查(运行前资源需求下限检查)
    - 增加更多限制维度: net bandwidth, disk io params, etc.
    - 获取节点的基础配置和资源使用情况: Galaxy + Gpostd

- 服务创建的自动化流程打通:
  - 自动创建代码与 Docker 仓库, 更加方便用户定制并创建服务

- 监听未完成任务的速度优化



### Q & A
