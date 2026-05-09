# 连接到 Web 服务

大多数服务提供商（包括 Amazon.com）都使用 `SOAP` 或 `REST`（或两者兼用）向互联网客户端程序暴露 Web 服务。你可以选择使用 `REST` 或 `SOAP` 来发起 Web 服务请求，这两种方式都能获得完全相同的结果。在本章中，你将学习如何使用 `REST` 和 `SOAP` 来访问 ECS 4.0。

`REST`（表述性状态转移）使用精心构造的 URL 以及特定的名称-值对，来调用服务器上的特定方法。你可以在 http://www.xml.com/pub/a/2004/08/11/rest.html 和 http://www.onlamp.com/pub/a/php/2003/10/30/amazon_rest.html 找到两篇关于 `REST` 的有用文章。

`REST` 被认为是与暴露此接口的 Web 服务进行通信的最简单方式。非官方消息称，85% 的 ECS 客户端采用了 `REST` 方式。

使用 `REST` 时，你只需执行一个标准的 HTTP GET 请求即可在 Amazon 上执行搜索，并会收到 XML 格式的响应。

`SOAP`（简单对象访问协议）是一种基于 XML 的标准，用于对 Web 服务请求或响应中传输的信息进行编码。`SOAP` 由多个组织共同推广，其中包括微软、IBM 和 Sun 等实力雄厚的公司。

在访问 ECS 时，你可以通过 `REST` 或发送 `SOAP` 消息来发起请求。Web 服务会返回包含你所请求数据的 XML 响应。

通过操作 ECS，你将更深入地了解 `REST` 和 `SOAP`。

**注意** 你需要明白，本章只涉及 Amazon ECS 提供的一小部分功能。对该主题进行深入讨论可能需要一本独立的书籍，但本章内容足以让你走上正轨。此外，请注意本章集成了 Amazon.com 的功能，但使用同一个 Amazon ECS 账户，你还可以访问 Amazon.fr、Amazon.ca、Amazon.de、Amazon.co.jp 和 Amazon.co.uk 的服务。

## 创建你的亚马逊电子商务服务账户

ECS 官方网站位于 http://www.amazon.com/webservices。你可以在 http://developer.amazonwebservices.com/connect/ 找到最新版本的文档——请务必收藏此网址，因为它会非常有用。

在继续之前，你需要先创建一个 Amazon ECS 账户。要访问 ECS，你需要一个 `Access Key ID`，它用于在 ECS 系统中标识你的账户。如果你还没有，请立即在 http://www.amazon.com/gp/aws/registration/registration-form.html 申请。`Access Key ID` 是一个 20 位的字母数字字符串。

**注意** 在 2005 年 10 月 11 日之前，Amazon 提供的是称为 `Subscription ID` 的东西，而不是 `Access Key ID`。两者的用途类似，如果你已有 `Subscription ID`，可以继续使用它。对于任何新的应用程序，Amazon 鼓励你使用 `Access Key ID`。

[www.it-ebooks.info](http://www.it-ebooks.info/)

如 图 17-2 所示，`Access Key ID` 使你可以访问更多 Amazon Web Services 和 Alexa Web Services（Alexa 是 Amazon 旗下的一项服务）。要访问其中一些服务，你还需要一个 `Secret Access Key`，该密钥在注册时也会获得，但使用 ECS 时不需要 `Secret Access Key`。

**图 17-2.** *Amazon Web Services*

## 获取亚马逊联盟推广 ID

你之前创建的 `Access Key ID` 是通过 Amazon ECS 检索数据的钥匙。这些数据允许你构建你在图 17-1 中看到的 Amazon Super Hats 部门。



