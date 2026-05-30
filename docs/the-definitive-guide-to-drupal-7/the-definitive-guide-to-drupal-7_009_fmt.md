# 第六部分：高级站点构建主题

## 第 25 章：Drupal Commerce

Drupal Commerce 概述

关键特性

深入探究 Drupal Commerce

商务

价格

动态定价

产品

订单项

产品引用

构建产品展示节点类型

客户

订单

支付

启用支付方式

结账

购物车

总结主要组件

实现 Drupal Commerce

开发历史

设计理念

开发标准

基于 Drupal 7 构建

核心实体与字段

表单 API 改进

贡献模块依赖

总结

## 第 26 章：Drush

Drush 入门

Drush 命令中的 Drupal 站点选择

Drush 别名文件 (`aliases.drushrc.php`)

使用 Drush Shell

使用 Drush 应用代码更新

安装 Drush 扩展

深入探讨 Drush 配置选项和别名

Drush 上下文

命令特定选项

站点列表

使用远程命令通过 Drush 部署站点

设置 SSH 密钥对

创建远程 Drupal 站点的本地副本

管理转储文件

在远程系统上使用`sql-sync`而无需安装`drush`

使用 Drush 站点上下文控制`sql-sync`选项

使用 Drush 编写脚本

处理脚本命令行参数和选项

运行外部命令

处理调用进程的结果

输出与日志记录

提示用户

日志记录与错误报告

编写 Drush 扩展

Drush 命令钩子

提供命令实现函数

返回数组以将结构化数据传递给其他 Drush 脚本

通过回调项手动指定命令函数

将命令实现放在单独的文件中

Drush 帮助钩子

更改 Drush 命令行为

总结

![images](img/square.jpg) 第 27 章：扩展 Drupal

你需要关心扩展性吗？

缓存

在开发过程中禁用缓存

`memcached`

Varnish

关于数据库

索引

SQL 中的`NULL`

ACID 与 BASE 之间的 CAP 理论

MongoDB

Watchdog、Session 和队列

MongoDB 中的空值

总结

![images](img/square.jpg) 第 28 章：用美味语义为你的内容增添风味

信息过载

我们是如何走到这一步的？

去中心化的数据空间

在全球网络规模上链接数据

你明白我的意思吗？

RDFa，或如何用语义增强 HTML

RDFa、微格式与微数据

Drupal 7 与语义网

理解 RDF 映射的结构

处理 RDF 映射结构

Drupal 7 中的 RDF 词汇表

通过贡献模块在 Drupal 核心之外使用 RDF

总结

![images](img/square.jpg) 第 29 章：菜单系统与进入 Drupal 的路径

通过示例理解 Drupal 的菜单系统

无止境之路

路径的结构

回调函数

加载函数

适配度

修改现有路由项

总结

![images](img/square.jpg) 第 30 章：底层探秘：Drupal 在显示页面时的内部机制

引导启动

第一阶段：初始化配置

第二阶段：尝试提供缓存页面

第三阶段：初始化数据库层

第四阶段：初始化变量系统

第五阶段：初始化会话处理

第六阶段：设置页面头信息

第七阶段：确定页面语言

最终阶段：加载模块并初始化主题

页面回调函数的执行

典型示例

总结

![images](img/square.jpg) 第 31 章：搜索与 Apache Solr 集成

搜索模块管理选项

搜索结果与分面区块

搜索模块 API

创建搜索所需的钩子实现

其他搜索模块钩子

Apache Solr 搜索配置

启用的过滤器

类型偏好与排除

Apache Solr 搜索自定义

用于将数据传入 Solr 的钩子

用于修改查询和结果的钩子

与 Apache Solr 服务器集成

管理 Solr 索引中的数据

搜索与分析

总结

![images](img/square.jpg) 第 32 章：用户体验

模块化

人性化 API

记忆

长期记忆

心智模型

感知

格式塔心理学

相似律

接近律

色彩

色彩和谐

实践

流程

挑战

概念：你到底在构建什么？

线框图

构建：构建 Alpha 版本并让用户验证

优化：观察与新版本