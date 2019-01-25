﻿---
layout: post                  
title: "Microservice Style"             
date: 2019-01-24               
tag:  微服务
---

## 微服务架构

**传统的系统架构** 是单一架构模式。这种模式就是把应用整体打包部署，具体的样式依赖本身采用的语言，采用Java语言打成war包部署到tomcat，Jetty这样的应用服务器上，如果采用spring boot可以打包成jar包部署。

随着技术的升级以及开源的轻量级技术的不停涌现，催生了新的架构设计风格-微服务架构的出现

**微服务架构** 一个大型的软件应用由一个或多个微服务组成，系统中的各个微服务之间的关联通过暴露api来实现。这些独立的微服务不需要部署在同一个虚拟机，同一个系统和同一个应用服务器中

### 单一架构模式

- 优点
    
    - 在项目初期开发方便，测试方便，部署方便。

- 缺点

    - 开发中后期修复漏洞和实现新功能现得困难和耗时
    - 规模越大，启动时间越长，拖慢开发进度，小功能得修改部署变得复杂耗时
    - 系统的不同模块的需要不同的特定的虚拟机环境，由于是单一的架构模式，只能折中选择
    - 任意模块的漏洞或者错误都会影响这个应用，降低系统的可靠性
    - 不能改变系统采用的技术或者框架或者语言

### 微服务架构特点：

1. 微服务架构中将组件定义为可被独立替换和升级的软件单元，在应用架构设计中将通过整体应用切分为可独立部署及升级的微服务方式进行组件化设计
2. 传统的应用模式是一个团队以项目模式开发完整的应用，开发完成后交给运维维护，而微服务倡导一个团队应该如开发产品般负责一个微服务完整的生命周期，倡导谁开发，谁运营的开发运维一体化方法
3. 去中心化治理：传统的单一架构模式倾向于单一的技术平台，微服务架构鼓励使用合适的工具完成各自的任务，每个微服务可以考虑选用最佳工具完成（不同的编程语言）。微服务架构倡导采用多样性持久化的方法，让每个微服务管理其自有数据库，并允许不同微服务采用不同的数据持久化技术

- 优点

    - 每个服务单独存在且微小，有单独的团队负责开发，就相当于使用单一架构模式在开发，自由选择开发的技术，但是要遵循统一的API约定
    - 每一个微服务都是独立部署，可以进行快速迭代部署，根据各自服务需求选择合适的虚拟机和使用最匹配的服务资源要求的硬件
    - 整体应用程序被分割为可管理的模块和服务，单个的服务可以更快速的开发、更简单的理解和维护
    - 需要实现负载均衡的服务可以部署在多个云虚拟机上，加入NGINX这样的负载均衡服务实现多个实例之间分发请求，这样不需要整个应用都部署负载均衡

- 缺点

    - 微服务应用作为分布式系统带来了复杂性。当应用是整体应用程序时，模块之间的调用都在应用内，即使进行分布式部署，任然可以在应用内调用。微服务是多个独立的服务，进行模块调用的时候，分布式会很麻烦
    - 多个独立数据库，事务的实现更具有挑战性
    - 测试微服务变得复杂，当一个服务依赖另一个服务时，测试时候需要另一个服务支持
    - 部署基于微服务的应用也很复杂，整体应用程序部署只需要部署在同一组相同的服务器上，在这些服务前面加上传统的负载均衡器即可。独立的服务变得复杂，需要更高得自动化形式

Spring Cloud为开发人员提供了快速构建分布式系统中的一些通用模式

1. 配置管理
2. 服务发现
3. 断路器
4. 智能路由
5. 微代理
6. 控制总线
7. 一次性令牌
8. 全局锁
9. 领导选举
10. 分布式会话
11. 群集状态
