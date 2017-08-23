尊敬的甘学长: 您好, 我是七月初即将入职的吴凯, 以下是我的 6/12 -- 6/17 学习进度周报:

- 学习了 `docker machine` 的用法, 利用 `digitalocean` 部署了一个基于 `swarm` 编排的巨丑的 kv 数据查询器应用, 请[猛戳这里](http://45.55.216.55:5000)调戏, 建立了三个主机: `consul` 用来作服务发现, app 部署在 master 上, 基于 `redis` 的存储服务部署在 slave 上.

- 在以上基础上, 为本项目的 GitHub [代码](https://github.com/quietcoolwu/dockerapp) 增加了 `CircleCI` 的 webhook 触发CI流程, 初步实现了代码 push 后自动 build 并反馈, 包括运行成功后自动 push 到 `docker hub(docker registry)` 的流程.

- 进一步了解 `docker-compose` 的配置语法, 比如利用 `extends` 关键字实现生产环境和开发环境公共部分配置的复用.

- Dashboard 选型调查中, 阅读 `ElementUI` 的文档, 参考一个现有[项目](https://github.com/PanJiaChen/vue-element-admin) 和[CoPilot](https://github.com/misterGF/CoPilot), 还有[vue-admin](https://admin.vuebulma.com), 想以后进一步比较下.

- 阅读了 `click` 的[文档](http://click.pocoo.org/5/), 准备基于这个开发命令行工具, 主要是因为 `click` 本身也是新版 `Flask` 依赖.

- 复习了 `Flask` 中 [Blueprint](http://www.pythondoc.com/exploreflask/blueprints.html) 的用法.

- 初步了解了 `docker swarm` 的过滤器和调度原理与配置方法.

- 结合之前代码复习 `Celery` 配置, 初步了解使用 `Flower` 对 `Celery` 进行网页端监控, 参考 [Mastering Flask](https://www.packtpub.com/web-development/mastering-flask).
