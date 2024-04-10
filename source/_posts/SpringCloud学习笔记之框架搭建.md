---
title: SpringCloud学习笔记之框架搭建
date: 2021-04-02 12:59:23
categories:
    - 后端
    - SpringCloud
tags:
    - 框架搭建
    - SpringCloud
---
# 创建目录层级

## idea 创建多个空的 Maven 工程如下层级目录:
```
light-cloud     
├── ruoyi-gateway         // 网关模块 [7499]
├── ruoyi-auth            // 认证中心 [7399]
├── light-dependencies    // 通用模块
│       └── light-api-dependencies             // feign接口通用依赖
│       └── light-base-dependencies            // 核心业务通用依赖
│       └── light-entity-dependencies          // 实体模块通用依赖
│       └── light-gen                          // 代码生成 [6999]
├── light-modules         // 业务模块
│       └── light-user                // 业务模块-用户
│           └── light-user-api              // 用户模块feign接口
│           └── light-user-biz              // 用户模块核心业务 [7299]
│           └── light-user-entity           // 用户模块实体
│       └── ...                       // 其他业务模块
│       └── light-job                 // 定时任务 [7199]
│       └── light-file                // 文件服务 [7599]
├── ruoyi-visual          // 监控中心 [9100]
├──pom.xml                // 公共依赖

```
其中:
- 主模块为 light-cloud,主要用来管理整个服务框架的第三方包;
- light-gateway 为网关模块,负责路由转发、负载均衡、熔断、鉴权、路径重写等;
- 采用 nacos 作为服务注册发现组件,项目创建暂不涉及;
- user 为业务模块-用户管理;
- 每个业务模块下分三个子模块 biz/api/entity,分被对应业务处理(controller,service)/业务调用(feign)/业务实体(bean) ;
- light-base-dependencies 为 biz 共用基础模块;
- light-api-dependencies 为 api 共用基础模块;
- light-entity-dependencies 为 entity 共用基础模块;
- light-job为定时任务模块;
- light-file为文件服务模块;

项目结构及各模块 pom 源码 [Gitee](https://gitee.com/wangafa/light-cloud);

initial分支为初始代码;