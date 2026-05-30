# Drupal 7 权威指南

版权所有 © 2011，作者：Benjamin Melançon、Jacine Luisi、Károly Négyesi、Greg Anderson、Bojhan Somers、Stéphane Corlosquet、Stefan Freudenberg、Michelle Lauer、Ed Carlevale、Florian Lorétan、Dani Nordin、Ryan Szrama、Susan Stewart、Jake Strawn、Brian Travis、Dan Hakimzadeh、Amye Scavarda、Albert Albala、Allie Micka、Robert Douglass、Robin Monks、Roy Scholten、Peter Wolanin、Kay VanValkenburgh、Greg Stout、Kasey Qynn Dolin、Mike Gifford、Claudina Sarahe、Sam Boyer 和 Forest Mars，以及 George Cassie、Mike Ryan、Nathaniel Catchpole 和 Dmitri Gaskin 的贡献。

保留所有权利。未经版权所有者及出版人事先书面许可，不得以任何形式或任何方式（电子或机械，包括影印、录制或任何信息存储检索系统）复制或传播本著作的任何部分。

ISBN-13 (平装): 978-1-4302-3135-6

ISBN-13 (电子): 978-1-4302-3136-3

本书中可能出现商标名称、标志和图像。我们仅在编辑风格下使用这些名称、标志和图像，以维护商标所有者的权益，并无意侵犯商标，而非在每个出现商标名称、标志或图像的地方都使用商标符号。

本出版物中使用的商品名称、商标、服务标志及类似术语，即使未被明确标识，也不应视为对其是否受专有权利保护的表达。

总裁与出版人：Paul Manning

首席编辑：Ben Renow-Clarke 和 Matthew Moodie

技术审稿人：Richard Carter

编辑委员会：Steve Anglin、Mark Beckner、Ewan Buckingham、Gary Cornell、Jonathan Gennick、Jonathan Hassell、Michelle Lowman、James Markham、Matthew Moodie、Jeff Olson、Jeffrey Pepper、Frank Pohlmann、Douglas Pundick、Ben Renow-Clarke、Dominic Shakeshaft、Matt Wade、Tom Welsh

协调编辑：Debra Kelly

文字编辑：Mary Behr

排版：MacPS, LLC

索引编制：BIM Indexing & Proofreading Services

插图制作：April Milne

封面设计：Anna Ishchenko

本书在全球范围内通过 Springer Science+Business Media, LLC. 发行，地址：233 Spring Street, 6th Floor, New York, NY 10013。电话：1-800-SPRINGER，传真：(201) 348-4505，电子邮件：`orders-ny@springer-sbm.com`，或访问 `www.springeronline.com`。

如需翻译信息，请发送电子邮件至 `rights@apress.com`，或访问 `www.apress.com`。

Apress 及 friends of ED 的书籍可批量购买以用于学术、企业或促销用途。大多数图书也提供电子版版本和许可证。更多信息请参考我们的特殊批量销售——电子版许可网页：`www.apress.com/bulk-sales`。

本书中的信息按“原样”提供，不提供任何担保。尽管在准备本著作时已采取一切预防措施，但作者和 Apress 对因本书所含信息直接或间接引起的任何损失或损害，不承担任何责任。

本书中示例和项目所需的源代码可在 `definitivedrupal.org` 和 `apress.com` 上获取。成功下载代码前，您需要回答与本书相关的问题。

*谨献给 Drupal 社区。*

## 内容概览

目录

序言

关于作者

关于技术审稿人

致谢

前言：为什么选择 Drupal？

Drupal 7 的新特性

如何使用本书

Drupal 的工作原理

第一部分：入门

第 1 章：构建一个 Drupal 7 站点

第 2 章：基本工具：Drush 与 Git

第二部分：开始使用

第 3 章：使用 Views 构建动态页面

第 4 章：总有一个合适的模块

第 5 章：使用 Organic Groups 创建社区网站

第 6 章：Drupal 的安全性

第 7 章：更新 Drupal

第 8 章：扩展你的站点

第三部分：让生活更简单

第 9 章：Drupal 社区：获取帮助与参与其中

第 10 章：规划与管理 Drupal 项目

第 11 章：为最终用户和生产团队编写文档

第 12 章：开发环境

第 13 章：上线部署与新功能发布

第 14 章：以人为中心的开发思维

第四部分：前端开发

第 15 章：主题制作

第 16 章：高级主题制作

第 17 章：jQuery

第五部分：后端开发

第 18 章：模块开发入门

第 19 章：在模块中使用 Drupal 的 API

第 20 章：优化你的模块

第 21 章：将模块移植到 Drupal 7

第 22 章：编写项目特定代码

第 23 章：使用 Simpletest 进行功能测试入门

第 24 章：编写一个主要模块

第六部分：高级站点构建主题

第 25 章：Drupal Commerce

第 26 章：Drush

第 27 章：扩展 Drupal

第 28 章：用语义化丰富你的内容

第 29 章：菜单系统与 Drupal 路径解析

第 30 章：揭秘：Drupal 显示页面时的内部机制

第 31 章：搜索与 Apache Solr 集成

第 32 章：用户体验

第 33 章：完成站点：剩下的 90%