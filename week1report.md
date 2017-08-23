各位学长学姐:
 我是07.04入职的SA实习生吴凯, 以下是我的 07.04 -- 07.07 工作周报:

- 参与了 07.05 的组会, 其中对接手项目进行了讨论, 初步明确了项目需要实现的功能细节, 技术重点难点等要素, 并开始参与技术调研.
- 在调研过程中发现自己对 docker network 与宿主机网络的关系了解不足, 在 mentor 指导下了解了 `iptables` 和 `ip` 命令配置转发与路由规则, 初步掌握了配置方法, 参考了[文档1](http://blog.csdn.net/reyleon/article/details/12976341)和[文档2](https://wsgzao.github.io/post/static-routes/#rt-tables).
- 对 docker network 下的几种 network driver 有了初步了解, 参考了[文档3](https://blog.docker.com/2016/12/understanding-docker-networking-drivers-use-cases/).
- 了解了策略路由配置的概念及其与静态路由配置的异同, 初步了解如何通过 `ip` 命令配置策略路由.
- 在 mentor 指导下对 docker swarm mode 下不同于 bare metal 的网络有了初步了解, 参考了[文档4](https://docs.docker.com/engine/swarm/networking/)
- 根据对需求的了解开始写第一版 [web app](https://git-sa.nie.netease.com/gzwukai/edge_node_models_demo), 从 db 建构开始, 但由于对业务场景细节还不够熟悉, 想必还有非常多需要更改的地方.

第一周小结: 自己在网络基础知识上有所缺漏, 没有在项目选型初期输出应有的帮助, 后期会尽快补缺.
