# 基本介绍

- 多个服务器上，运行着多个容器，容器的统一管理不方便
- 大规模的容器编排技术
- [官方文档](https://kubernetes.io/)

# 架构

- N master node + N worker node， N>=1
- 主节点不干活，工作节点干活，投票机制