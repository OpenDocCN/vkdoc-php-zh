# 第 3 节 前置条件

本节概述了后续章节将用到的 Symfony 组件和后端服务，以及它们的安装说明。

本书的代码基于 PHP 8.3 版本和 Symfony 7.3 版本编写。

## Symfony 组件

### Symfony Serializer Pack

Symfony Serializer 组件提供了一种简单高效的数据序列化与反序列化方式。它允许我们将 PHP 数据结构（如数组和对象）转换为易于存储或传输的格式（如 JSON 或 XML）。同样，该组件也允许我们将原始数据（JSON、XML）反序列化为 PHP 类。我们将需要此组件来反序列化 API 输入并序列化 API 输出。

### Symfony Validator

Symfony Validator 组件提供了一种数据验证方式。它允许您为数据定义规则和约束，然后检查数据是否符合这些规则。该组件提供了大量内置验证器，例如对电子邮件地址、URL 和信用卡号的检查，以及更复杂的验证逻辑。

我们将需要此组件来验证执行操作所需的数据，从而避免在执行过程中出错。

### Symfony Security

Symfony Security 组件提供了一套工具，用于处理 Symfony 应用程序中的身份验证、授权和其他安全相关任务。该组件具有用户身份验证、访问控制和防火墙配置等功能，可帮助开发者实施强大的安全措施，以保护其应用程序免受未经授权的访问和潜在安全威胁。

我们将在第 2 章中使用此组件来创建防火墙以保护 API 端点，同时还将使用投票器（voters）根据用户角色等条件授权特定用户执行操作。

### Symfony Messenger

Symfony Messenger 组件简化了在 Symfony 应用程序中发送和接收消息的过程。它使开发者能够通过使用消息总线、处理器和传输机制来异步处理消息，从而实现应用逻辑的解耦。通过利用 Messenger 组件，开发者可以简化应用程序不同部分之间的通信，提高可扩展性和可维护性。

我们将在第 3 章中使用此组件在后台执行操作。此外，我们还需要根据实际使用的传输方式安装相应的 Messenger 传输包。本书使用 Redis 作为路由传输，因此我们还需要安装 `redis-messenger` 包。由于我们需要将失败的操作路由到 Doctrine 传输，因此还需要安装 `doctrine-messenger` 组件。

### Symfony ORM Pack

Symfony Doctrine 组件是一个强大的工具，它将 Doctrine ORM（对象关系映射）库集成到 Symfony 应用程序中。该组件通过将数据库表映射到 PHP 对象来简化数据库交互，使开发者能够像处理普通 PHP 对象一样处理数据库记录。该组件具有实体映射、查询构建和数据库迁移等功能，为在 Symfony 应用程序中管理数据库操作提供了一种强大且高效的方式。

我们需要此组件，因为将在第 2 章创建一个 `User` 实体来存储能够访问我们 API 的用户。在第 3 章中也需要它，因为我们将把失败的操作路由到 Doctrine 传输。

### Symfony Doctrine Fixtures Bundle

Symfony Doctrine Fixtures Bundle 是一个工具，允许开发者轻松管理并将样本数据加载到 Symfony 应用程序的数据库中。它提供了一种创建和执行数据夹具的方式，这些夹具是定义要加载到数据库中的样本数据的 Symfony 服务。我们需要这些服务来向 `User` 实体加载用户数据。

### Symfony String

Symfony String 组件提供了一套用于在 PHP 中处理字符串的有用类和函数。该组件包含用于操作字符串的类（如修剪、填充和分割），以及用于处理 URL、词形变化器和 Slug 生成器的类。

我们将在第 1 章的最后一个部分使用此组件来驼峰化操作名称。

### Symfony Maker Bundle

Symfony Maker Bundle 是一个对开发者友好的工具，它可以自动创建常见的 Symfony 元素，例如控制器、实体、表单等。我们将使用此组件生成 `User` 实体，并在最后一章生成 API 测试。

### Symfony Rate Limiter

Symfony Rate Limiter 组件允许开发者控制向应用程序发出的请求速率。它通过限制用户在特定时间范围内可以发出的请求数量来帮助防止滥用。该组件提供了一种灵活且可定制的方式来实现限流策略（如窗口限流和令牌桶），以确保系统的稳定性和安全性。

要安装此组件，我们还需要同时安装 Symfony Lock 组件。

### Symfony Test-Pack

Symfony Test Pack 是一个集成 PhpUnit 测试工具的组件，提供了一套工具和实用程序来帮助您为基于 Symfony 的应用程序编写和运行测试。它包含测试客户端、分析器以及一系列断言等功能，使测试更加轻松高效。我们需要此组件以便在最后一章编写 API 测试。

## 安装 Composer

在创建 Symfony 项目之前，我们必须安装 Composer。Composer 是最常用的 PHP 包管理器，创建项目和安装 Symfony 组件都需要它。若要下载并安装它，请参考官方文档。

## 创建 Symfony 项目

我们将需要安装 `symfony-cli` 工具来创建新的 Symfony 项目。请按照 Symfony 官方下载文档进行安装。安装完成后，我们就可以创建项目了。为此，需要在您选择的文件夹中执行以下命令。

```
symfony new  --version="7.3.*"
```

上述命令应会创建基本的文件夹结构并安装框架所需的库。

## 安装组件

创建项目后，我们需要安装之前小节中讨论的组件。要安装它们，您必须使用 composer：

```
composer require symfony/messenger
composer require symfony/redis-messenger
composer require symfony/doctrine-messenger
composer require symfony/orm-pack
composer require symfony/security-bundle
composer require --dev symfony/maker-bundle
composer require --dev orm-fixtures
composer require symfony/lock
composer require symfony/rate-limiter
composer require symfony/string
composer require --dev symfony/test-pack
```

安装测试组件后，通过从项目根文件夹运行以下命令来确保 PhpUnit 已安装：`bin/phpunit`。

### 安装数据库

由于我们将在第 2 章创建用户实体，因此需要一个随时可用的数据库。我们将使用 [SQLite](https://www.sqlite.org/index.html) 来管理数据库。为了安装它，我们将使用 [docker](https://www.docker.com/) 和 [docker-compose](https://docs.docker.com/compose/)。

您可以自由选择您想要的数据库（MySQL、Postgres、MariaDB……），并使用您认为合适的方式安装。就我个人而言，我喜欢使用 docker，因为它可以轻松创建新容器，而无需在计算机上安装服务，并且 docker hub 中有大量预构建的镜像可供使用。

让我们在项目根目录下创建 `docker-compose.yaml` 文件，并添加 SQLite 服务：

```
version: '3'
services:
  sqlite3:
    image: keinos/sqlite3:latest
    volumes:
      - './db:/var/db'
```

如您所见，我们将使用最新的 SQLite3 镜像版本（您可以选择任何其他镜像），并且我们正在创建一个新的卷，将容器 `db` 文件夹链接到我们的 `var/db` 文件夹。要启动容器，请使用以下命令：

```
docker compose up -d
```

然后，我们只需将 `DATABASE_URL` 环境变量添加到位于项目根目录的 `.env` 文件中。

```
DATABASE_URL="sqlite:///%kernel.project_dir%/var/db/data.db"
```

此变量包含 SQLite 数据库文件的路径。该路径与 compose 文件中定义的卷路径相匹配。请确保您已创建 `var/db` 目录。

### 安装 Redis

[Redis](https://redis.io/) 是一个开源的内存数据库，可用作缓存或消息代理等。它支持多种数据结构，例如字符串、哈希、列表、集合和流。它以其高性能和处理数据的通用性而闻名。我们将在第 3 章中使用 Redis 作为信使传输。就像我们对 SQLite 所做的那样，我们将使用 docker 来安装它。

正如我之前所说，您可以自由选择您的方式安装 Redis。

让我们将 Redis 服务添加到我们的 compose 文件中：

```
version: '3'
services:
  sqlite3:
    image: sqlite:latest
    volumes:
      - './db:/var/db'
  redis:
    image: redis
    ports:
      - '6401:6379'
```

如您所见，我们正在使用 Redis 镜像，并将容器 Redis 端口 `6379` 映射到我们计算机的端口 `6401`。让我们再次执行 `docker compose up` 命令来启动 Redis 容器：

```
docker compose up -d
```

这将为刚刚添加的 Redis 服务创建容器。现在，就像我们之前对 `DATABASE_URL` 环境变量所做的那样，让我们添加 `MESSENGER_TRANSPORT_DSN` 变量，该变量将包含消息将路由到的 Redis DSN。

```
MESSENGER_TRANSPORT_DSN=redis://localhost:6401/operations
```

`operations` 路径指明了消息将排队等候的 Redis 流名称。

`redis-messenger` 组件使用 [Redis 流](https://redis.io/docs/latest/develop/data-types/streams/) 来排队消息。您可以通过最后一个链接了解更多信息，但您不必担心它们是如何工作的，因为 Symfony 信使会在内部完成所有路由工作。