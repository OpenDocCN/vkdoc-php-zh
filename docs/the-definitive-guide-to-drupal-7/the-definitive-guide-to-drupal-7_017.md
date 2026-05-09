# 第 34 章：Drupal 发行版与安装配置文件

## 站点模板

## 全功能服务

## 开发配置文件

## 示例发行版：Drune

## 创建安装配置文件

### 安装配置文件的结构



处理配置：功能特性

将安装配置文件与功能特性作为开发工具使用

打包你的代码

Drush Makefiles 文件

托管在 drupal.org 上

打包

发行版的未来

总结

第七部分：Drupal 社区

![images](img/square.jpg) 第 35 章：Drupal 的故事：一连串意外事件

最初的意外

Drupal 站稳脚跟

噩梦般延长的周末

如有问题，请在提问前先搜索

故事仍在继续

![images](img/square.jpg) 第 36 章：现在你入行了：用 Drupal 谋生

构建 Drupal 站点：新技术的新规则

“我讨厌 Drupal”：可能出错的地方

理解 Drupal

基于 Drupal 进行构建

确保你的成功

打造你的 Drupal 职业生涯

找到你的位置

让自己走出去

独自创业：建立 Drupal 业务

打造 Drupal 职业生涯

构建 Drupal：以贡献者身份谋生

“回馈”的益处

可持续性至关重要！

潜在的商业模式

设定期望

持续进步

![images](img/square.jpg) 第 37 章：维护一个项目

什么是 Drupal 项目？

设置你的 Drupal.org 账户以进行贡献

创建一个沙盒项目

状态

项目信息

深入使用 Git

管理 SSH

在你的项目上进行开发

从沙盒小镇到项目大都

关于 Drupal.org 上的分支和标签

为你的应用程序准备一个分支

准备你的项目以供审查

申请访问权限

获得访问权限

总结

![images](img/square.jpg) 第 38 章：为社区做贡献

为什么要贡献？

没有贡献，就没有 Drupal

迈出第一步

贡献的方式


1. 提供非技术性支持
2. 分享一切
3. 在论坛、群组、邮件列表、聚会和 IRC 上回答问题
4. 为 Drupal.org 撰写文档
5. 贡献补丁
6. 贡献代码与设计
7. 管理问题队列
8. 审核他人的贡献
9. 让 Drupal.org 变得更好
10. 主持和组织聚会、训练营、峰会等
11. 资金
12. 打造包容的 Drupal 社区

推动这场运动

## 第八部分：附录

![图片](img/square.jpg) 附录 A：将 Drupal 网站从 6.x 升级到 7.x

- 评估现状
- 内容概览
- 第三方模块
- 制定计划
- 运行升级（反复执行）
- 准备工作
- 为升级涉及的所有站点配置 Drush 别名
- 中间方案
- 在更新钩子中捕获额外的升级步骤
- 可选：从网站粘合代码模块的 Drupal 7 版本开始编写自定义升级函数
- 创建升级模块
- 在代码中启用模块
- 在代码中禁用模块
- 自动化字段升级
- 重新运行升级
- 创建功能模块
- 考虑创建基础功能模块
- 构建功能模块
- 将功能模块添加到自动升级中
- 数据迁移
- 管理流程
- 了解遗留数据
- 特定难点
- 初步分析
- 迭代
- 展示
- 审核
- 时间安排
- 上线日
- 总结

![图片](img/square.jpg) 附录 B：Drupal 性能分析与优化

- 用户感知的性能
- 什么导致网站变慢？
- 真正的性能
- 页面与区块级别缓存
- Drupal 性能分析入门
- 慢速数据库查询
- 总结

![图片](img/square.jpg) 附录 C：页面渲染与修改

- 第 1 步：路由项



第二步：页面回调触发

第三步：交付回调函数

第四步：drupal_render_page()

第五步：hook_page_alter()

第六步：drupal_render()

![图像](img/square.jpg) 附录 D：Drupal 可视化设计

为什么设计师应该使用 Drupal

为 Drupal 做设计：这意味着什么

Drupal 页面的结构

从内容出发进行设计

让你的 Drupal 设计师工作更轻松

记住——设计的目的是沟通

理解网站架构和内容策略

明智地选择字体

清晰审查特殊功能的需求并概述其预期功能

为完整的用户体验而设计

Drupal 中的 HTML5

你如何参与其中

![图像](img/square.jpg) 附录 E：无障碍访问

近期增强

标准是什么？

谁受益？

这是法律规定

九种让网站具有无障碍性的方法

无障碍模块

主题化你的网站

对比度与颜色

自动化测试

模拟

引入 WAI-ARIA

维护至关重要

定期对新旧页面进行审核

获取专家反馈

![图像](img/square.jpg) 附录 F：Windows 开发环境

从 LAMP 到 WISP

Visual Studio

WAMP 集成环境

Drupal 基础知识

VS.Php

phpMyAdmin 和 MySQL Connector

Drush

为 Windows 安装 Drush

运行 Drush

总结

附录 G：在 Ubuntu 上安装 Drupal

在 Windows 或 Mac OS X 上运行 Ubuntu

使用 Drubuntu 为 Drupal 开发自定义 Ubuntu

安装 Drupal

附录 H：Mac OSX 安装

下载 Drupal 核心文件

附录 I：使用 Acquia Dev Desktop 设置 Drupal 环境

安装

更进一步

索引

## 前言

市面上有许多 Drupal 书籍，它们都在争相获取你辛苦挣来的钱。从网站构建到主题开发再到模块开发，各种书籍专注于你可能感兴趣的 Drupal 各个领域。

而这本书则是独一无二的。它的目标是向你展示 Drupal 的方方面面，而且在许多情况下，这些内容正是来自那些亲手创建它们的专家。无论是 Drupal 的经验还是兴趣，本书都有适合所有层次读者的内容。

本书从入门材料开始，教你如何快速搭建并运行一个简单的网站，以及如何利用 Drupal 一些最受欢迎的贡献模块来扩展它。书中还有章节介绍如何通过初级和高级主题化以及 jQuery 进行前端开发，让你的 Drupal 网站看起来不再像一个典型的 Drupal 网站。你将学习如何通过模块开发来定制 Drupal，以满足那些超越庞大贡献项目库所能提供的用例，以及如何将 Drupal 6 模块移植到 Drupal 7，并添加自动化测试来确保你的代码持续正常工作。对于极客们来说，书中还提供了关于如何从命令行使用 Drupal、将其与 Git 配合使用、以及管理服务器部署和性能的信息。此外，甚至还有关于更广泛主题的内容，这些主题远远超出了 Drupal 本身，例如项目管理、为你的网站创建文档以及用户体验。

全书贯穿了来自一线的实践建议和经验教训，这些内容来自 Drupal 社区中那些最杰出、最聪明、最具创新精神的全明星阵容。作为 Drupal 7 的共同维护者，看到这庞大的工作得以整合，我感到无比激动。向 Benjamin 和整个编写团队致敬！

Angela Byron (webchick)
*Drupal 7 维护者*



## 关于作者

![images](img/square.jpg) **本杰明·梅朗松**是 Agaric（[`agaric.com`](http://agaric.com)）的联合创始人兼负责人，帮助人们创建和使用强大的网站。他与 Agaric 希望与那些重视开放与自由、并热衷于创建可扩展的协作网络的公司和组织合作。为了追求赋予所有人对其生活最大可能的掌控权（作为正义与自由的工作定义），本杰明致力于连接思想、资源和人。

![images](img/square.jpg) **阿尔伯特·阿尔巴拉**在完成语言学与计算机科学的大学本科学位后，于 2006 年开始涉足 Drupal。在 Joomla 工作两年后，他联合创立了`Mediatribe.net`，一家提供 Drupal 咨询服务的合伙企业。2009 年，他加入了蒙特利尔的非营利性 Drupal 组织 Koumbit。作为更大团队的一员，他得以专注于 Simpletest 和 Features 这两个领域，它们随着平台的成熟不断强化 Drupal 网站的质量。阿尔伯特的其他活动包括通过“青年大地”组织开展小规模的国际发展项目。

![images](img/square.jpg) **格雷格·安德森**是 Drupal 命令行工具`drush`的联合维护者之一。他还在理光公司负责美洲区的开发者技术支持小组。格雷格在工作中和个人生活中都使用 Drupal。他为“伟大狄更斯集市”网站进行了 Drupal 改造，在那里他与其他表演者一起将圣诞精神带入生活。他还协助为“创造性无政府状态协会”（一个历史重建协会，他和妻子在那里负责儿童项目）进行另一个大型网站迁移项目。他与一个社区学区支持小组合作，并与妻子共同撰写了一个环境博客。

![images](img/square.jpg) **萨姆·博耶**主导了 Drupal 项目从 CVS 到 Git 的迁移，并持续领导`Drupal.org` Git 团队在`Drupal.org`上扩展 Git 相关功能的工作。为此，他也是 d.o 基础设施团队的成员。

在 D8 之前，萨姆对核心的贡献不算多，但他现在投入了大量精力处理 Drupal 底层的关键路径系统。在贡献模块方面，他与 merlinofchaos 共同维护 Panels 和 CTools，并主导了版本控制 API 模块系列的管理。

![images](img/square.jpg) **埃德·卡勒维尔**是麻省理工学院的一名长期网络开发者，专注于能源与可持续发展领域（`mitenergyclub.org, sustainability.mit.edu`）。作为初生的麻省理工学院 Drupal 小组的创始人，他在麻省理工学院举办了多次 Drupal 活动，包括 Dries Buytaert 的 Drupal 现状演讲（麻省理工学院世界频道，2009 年）、2009 年和 2010 年的波士顿 Design4Drupal Camp，以及由 Moshe Weitzman 领导的波士顿 Drupal 小组月度会议。

![images](img/square.jpg) **乔治·卡西**从 Drupal 4.7 版本开始，就使用 Drupal 构建了多种网站和工具。他目前在 Acquia 担任客户顾问，专注于 Drupal Gardens。

![images](img/square.jpg) **纳撒尼尔·卡奇波尔**自 Drupal 4.5 版本起开始使用 Drupal，并从 2006 年起成为 Drupal 核心的定期贡献者。他为 Drupal 7 版本贡献了超过 400 个补丁，同时进行了广泛的代码性能分析，并维护了实体缓存和性能优化贡献模块。

![images](img/square.jpg) **斯特凡·科洛斯基**拥有爱尔兰数字企业研究所（DERI）的语义网硕士学位。他目前在麻省总医院神经退行性疾病研究所（MIND）担任软件工程师，专注于科学协作框架——一个基于 Drupal、用于构建生物医学研究者在线社区的分发版。

斯特凡曾为 Drupal 6 做出贡献，并且是 Drupal 7 核心的前 30 名贡献者之一。他维护 Drupal 7 中的 RDF 模块，并且是 Drupal 安全团队的成员。自 2005 年加入社区以来，他曾在多次 DrupalCon 和 DrupalCamp 上发表演讲，主题多围绕 RDF 和 Drupal。他合著的论文“使用 Drupal 生产和消费关联数据！”荣获了 ISWC 2009 最佳语义网应用论文奖。

斯特凡与他挚爱的妻子迪利尼、刚出生的儿子基兰以及极度活跃的狗狗玛雅居住在波士顿地区。

![images](img/square.jpg) **凯西·奎恩·多林**是一位人类学家，在研究（和组织）社区及非营利项目的成长与生存方面拥有丰富经验。她在弗吉尼亚联邦大学获得了人类学学士学位，辅修拉丁美洲研究（专注于巴西）和国际研究。她已发表的作品包括《坎东布雷语：牙买加拉斯塔法里运动中的声音与力量》和《约鲁巴宗教在巴西坎东布雷中的生存》。

![images](img/square.jpg) **罗伯特·道格拉斯**自 2004 年起全职从事 Drupal 相关工作。他撰写了第一本关于 Drupal 的书籍（《使用 Drupal、phpBB 和 WordPress 构建在线社区》；Apress，2005 年），并担任了《Pro Drupal Development》（Apress，2007 年）所有三个版本的技术编辑。通过教学和个人影响，他帮助数百人成功转型为 Drupal 开发者和服务提供商。2005 年，他领导了 Drupal 参与首届谷歌编程之夏项目，并从此一直活跃在领导者和导师的岗位上。

罗伯特自 2006 年起成为 Drupal 协会全体大会成员，并积极参与许多协会活动，包括组织 DrupalCon 和 DrupalCamp。2008 年，他联合创立了 Drupal 倡议组织，这是德国旨在推广 Drupal 的非营利组织。在该组织担任了两年的副主席后，他协助协调了新董事会的选举，并于 2010 年移交了管理权。

2007 年，罗伯特创立并构建了`goPHP5.org`，这是一项旨在联合开源 PHP 项目和网络主机以加速升级到 PHP5 的倡议。其成果之一是 Drupal 7 放弃了 PHP4 兼容性，使核心团队能够实现诸如添加 PDO 数据库驱动程序等功能。超过 100 个软件项目和 200 多个网络主机加入了这一运动，并获得了大量媒体报道。

罗伯特对 Drupal 最大的代码贡献体现在 Apache Solr 模块和 Memcache 模块上，这两个模块都始于 2007 年，他至今仍在维护。除了担任 Acquia 的全职顾问和咨询师外，罗伯特还担任 Commerce Guys 和 ICanLocalize 的顾问委员会成员，协助其业务、产品和市场决策。

![images](img/square.jpg) **斯特凡·弗罗伊登贝格**是一位后端开发人员，在 Linux 系统管理方面有一定经验。他于 2008 年底开始使用 Drupal 开发网站并加入社区，此后将大部分商业和志愿活动都投入其中。他的调试和性能分析技能使他在团队中广受欢迎，而他最喜欢的是论证更简单的架构和遵循标准，但这并不总是能被很好地接受。斯特凡是 Agaric（[`agaric.com`](http://agaric.com)）的负责人之一。

![images](img/square.jpg) **德米特里·加斯金**是一位 Drupal 贡献者，或许以 dmitrig01 的网名更为人所知。他 8 岁开始编程，11 岁开始接触 Drupal。从那时起，德米特里就对 Drupal、PHP、JavaScript 和 jQuery 非常熟悉。他维护着多个模块（包括 Drush Make），但主要工作集中在 Drupal 核心补丁上。德米特里曾在 Drupalcon、Badcamp 和谷歌发表过演讲。当德米特里不编程时，他就在创作音乐、听音乐，或者上十年级的课。



![images](img/square.jpg) **迈克·吉福德**于 1999 年创立了 OpenConcept Consulting Inc.，他长期积极开发和扩展开源内容管理系统，旨在帮助用户更好地掌控自己的网站。迈克一直致力于为加拿大和美国的进步组织及政治人物搭建在线活动平台。

自 2005 年起，OpenConcept (OC) 专攻 Drupal 开发。OC 团队为 Drupal 社区贡献了多个模块，并积极推动 Drupal 在政府和非营利组织中的应用。

迈克自 20 世纪 90 年代初便投身无障碍议题，是标准驱动设计的坚定倡导者。自 2009 年起，他为 Drupal 社区采纳的无障碍增强功能做出了贡献，包括 Drupal 7 核心的改进。他曾在多伦多和蒙特利尔的 DrupalCamp 上发表演讲，最近一次是关于 OC 在无障碍增强方面的工作。

![images](img/square.jpg) **丹·哈基姆扎德**是 Agaric 的创始合伙人之一。丹将时间和精力投入到构建这个俗称“互联网”的神秘现象中。他信奉自由开源软件的原则，主要使用 Drupal 内容管理框架进行开发。

![images](img/square.jpg) **米歇尔·劳尔**（在[`drupal.org`](http://drupal.org)上又名 miche）于 2006 年开启了她的 Drupal 冒险之旅，并迅速以其对细节的极致把握和把握全局的能力而闻名。这些天赋使她专攻网站架构和分阶段部署。除了自定义模块开发和主题制作，她还负责制定复杂内容架构的策略，涵盖从最终用户体验到网站管理员的可管理性。米歇尔的履历包括在 DrupalCon 巴黎、DrupalCon 旧金山和 DrupalCamp 蒙特利尔进行演讲，并担任 DrupalCamp NH 培训日的协调员和课程作者。在[`bymiche.com`](http://bymiche.com)上了解更多关于米歇尔的信息，并在 Twitter 上关注她 @bymiche。

![images](img/square.jpg) **弗洛里安·洛雷坦**于 2005 年开始使用 Drupal，这份激情在两年后转化为全职工作。作为一名 Drupal 开发者，他参与了许多大型社交网络项目，并为多家知名网络开发机构提供咨询服务。他的贡献包括模块、主题以及众多核心补丁。弗洛里安来自瑞士，同时也是全球多个社区的活跃成员。他是首届 Drupal Dev Days 的核心组织者，DrupalCon 哥本哈根的专题主席，并参与组织了许多其他活动。他曾在 DrupalCons、DrupalCamps 以及从日内瓦到圣地亚哥的各个地方社区进行演讲。弗洛里安是 Wunderkraut 的联合创始人，该公司为欧洲市场提供 Drupal 培训和咨询服务。他偶尔会在个人博客[`happypixels.com`](http://happypixels.com)上撰写文章。

![images](img/square.jpg) **杰辛·路易斯**是一名前端开发者，专精于 Drupal 主题开发。自 2004 年起从事网站开发，自 2007 年起主要使用 Drupal。她将大量业余时间投入到 Drupal 核心的标记和 CSS 相关问题中，同时也参与 Skinr 模块和 Sky 主题等贡献项目。她目前居住在纽约州瑞布鲁克。

![images](img/square.jpg) **福雷斯特·马尔斯**是一位超媒体架构师，自 2005 年起使用 Drupal——主要应用于媒体和业务集成领域——并且是 Drupal 社区的极度活跃分子。作为 DrupalCon 巴黎 2009 团队的成员，他搭建了著名的“Druplicon Road Trip”网站。他近期的一些项目包括为世界上最大的电视网络搭建视频分发平台，以及为纽约市构建首个公民参与平台。在业余时间里，他会进行关于 Drupal API 的演讲，例如“Bongo for Mongo”和“Drupal 的可怕真相”。他住在纽约，没有养他的两只猫。

![images](img/square.jpg) **艾莉·米卡**自 2001 年起便是一名开源开发者和倡导者。她曾参与多个开源应用程序的开发，包括在 Drupal 项目中进行了三年的深入开发和参与。通过她的托管和服务工作，她为当地社区提供了积极的赞助和教育支持，目前她在 Advantage Labs 从事教育、基础设施和开源开发工作。在此之前，艾莉曾是一家大型在线经纪公司的网站开发团队经理，这段经历教会了她如何将企业业务战略应用于可持续的社区参与。

![images](img/square.jpg) **罗宾·蒙克斯**（[`robinmonks.com`](http://robinmonks.com)）是一位热忱的开源贡献者，在 Drupal 社区拥有超过七年的经验，并且是 Podhurl Inc.的创始人。他目前致力于开发工具和服务，以促进“开放网络”的发展。

![images](img/square.jpg) **卡洛伊·涅杰希**在 1990 年代曾是匈牙利当时最大的计算机月刊《Chip Magazine》的专栏作家和编辑。之后，他转向了网络编程。他的生活和 Drupal 在 2004 年彻底交织在一起。此后，他成为最核心的贡献者之一，并曾短暂担任安全团队的首任负责人。如今，他是[`Examiner.com`](http://Examiner.com) 的高级软件架构师，这是最大的基于 Drupal 的网站之一。他对认知科学非常着迷，并将奥森·斯科特·卡德的《安德的游戏》奉为宝典。

![images](img/square.jpg) **丹尼·诺丁**（[`tzk-design.com`](http://tzk-design.com)）是一位用户体验设计师，于 2008 年从 Wordpress 转向 Drupal。自那以后，她一直是“Drupal 设计”社区的活跃声音，并且花费了大量时间寻找让 Drupal 设计更高效、更有效的方法。她定期在波士顿的 Drupal 活动中演讲，包括波士顿的“Drupal Camp 设计”活动。丹尼目前正在撰写《设计师的 Drupal 指南》，计划于 2011 年出版。你通常可以在 Twitter 上找到她，她的账号是@danigrrl。

![images](img/square.jpg) **迈克·瑞安**自 2003 年首次贡献以来，为 Drupal 社区付出了很多。他是 Migrate 和 Table Wizard 模块的幕后推手，并专长于作为 Cyrve 公司的一部分将数据迁移到 Drupal。他还是 Pathauto 模块的原作者，该模块是一个极为流行的 Drupal 模块，用于自动为页面创建友好的 URL，从而极大地提升搜索引擎优化（SEO）。

![images](img/square.jpg) **克劳迪娜·萨拉赫**的职业生涯始于俄勒冈州波特兰的 Pop Art 和 The New Group 公司，担任前端开发者。2007 年，她离开西海岸在纽约创业，开始在《赫芬顿邮报》工作。随后，她成为联合国儿童基金会创新部门的创始成员之一。正是在联合国儿童基金会，她对开源技术策略产生了兴趣。之后她在 Method 公司进一步发展了这一兴趣，为 Charlie Rose、PBS、Scholastic 和 Count Me In 等客户领导互动开发。她还为 Vestify 领导了策略和初始产品开发，这是一个为企业家提供众筹服务的网络应用，为 MassChallenge 全球创业大赛提供支持。



