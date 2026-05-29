# 16. 基础 Symfony 应用

上一章你安装并探索了 Symfony PHP 框架。本章将重点开发一个基础 Symfony 应用。

本章包含以下小节：

*   创建新 Symfony 项目

*   数据库设置与配置

*   数据库迁移

*   控制器路由

*   注册视图表单

*   在数据库中存储用户数据

## 创建新 Symfony 项目

由于上一章已安装 Symfony，现在我们创建一个名为 `blog-app` 的新项目并执行以下命令：

```bash
composer create-project symfony/skeleton:"6.1.*" blog-app
cd blog-app
composer require webapp
```

安装应用特定包时，系统可能会询问是否需要 Docker 特定配置设置，以便在 Docker 化环境中运行 Symfony 应用。更多信息请参考 Symfony 设置 Docker 安装网页：[`https://symfony.com/doc/current/setup/docker.xhtml`](https://symfony.com/doc/current/setup/docker.xhtml)。暂时忽略此项并选择否。所有关于 Symfony 目录结构的信息已在第 15 章提供。

Symfony CLI 提供了一个非常便捷的命令来检查所有已安装包的安全性，确保它们安全可靠：

```bash
symfony check:security
```

显示任何 Symfony 包是否存在已知漏洞的输出如图 16-1 所示。

![](img/514068_1_En_16_Fig1_HTML.jpg)

**图 16-1** 无已知漏洞

项目创建完成后，使用以下命令启动：

```bash
cd blog-app
symfony server:start
```

启动 Symfony Web 服务器的输出如图 16-2 所示。

![](img/514068_1_En_16_Fig2_HTML.jpg)

**图 16-2** Symfony Web 服务器已启动

现在你可以通过 `http://localhost:8000` 访问该应用，如图 16-3 所示。

![](img/514068_1_En_16_Fig3_HTML.png)

**图 16-3** Symfony 仪表盘网页

这应该能确认 Symfony 的安装和设置有效。

开发工具栏如图 16-4 所示。

![](img/514068_1_En_16_Fig4_HTML.jpg)

**图 16-4** Symfony 开发工具栏

该工具栏显示汇总统计信息。更多信息请阅读第 15 章。

## 数据库设置与配置

在构建博客应用的用户注册子功能时，你将了解 Symfony 的各种组件。在此过程中，你将看到 Symfony 如何让你轻松构建此类应用。任何应用的核心都是数据，你需要一个数据库来存储与用户相关的数据。

随着不同步骤的推进，你将逐步创建表。在前面几章中，你通过 `phpMyAdmin` 界面手动创建了数据库和表。这通常适用于演示项目，但在生产项目中，通常建议通过迁移（migrations）来维护数据库和表，这些迁移存储在文件中，可以提交到像 `git` 这样的源代码管理系统。

这有助于拥有可重复的数据结构，供其他团队成员快速上手并设置项目，也有助于创建不同的环境来运行项目，如 `dev`、`staging` 和 `production`。这样做还有一个额外的好处，即可以对数据库和表进行版本控制的模式，以便理解和维护变更历史，用于审计等目的。

在开始设置 Symfony 的迁移功能之前，需要配置数据库配置。第一步是安装 `Doctrine` ORM 包，这是一组主要专注于提供持久化服务和功能的 PHP 库。

```bash
$ composer require symfony/orm-pack
```

安装 Doctrine 包的输出如图 16-5 所示。

![](img/514068_1_En_16_Fig5_HTML.jpg)

**图 16-5** 安装 Doctrine 包

让我们在你的应用中安装并启用此 bundle，如图 16-6 所示。

![](img/514068_1_En_16_Fig6_HTML.jpg)

**图 16-6** 在应用中安装并启用此 bundle

```bash
$ composer require --dev symfony/maker-bundle
```

现在更新 `.env` 文件中与数据库相关的值。注释掉 PostgreSQL URL 并取消注释其上方的 MySQL URL 行。然后将：

```env
DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=8&charset=utf8mb4"
```

更新为：

```env
DATABASE_URL="mysql://root:password@127.0.0.1:3306/blog"
```

现在通过运行以下命令创建数据库：

```bash
$ php bin/console doctrine:database:create
```

数据库创建的输出如图 16-7 所示。

![](img/514068_1_En_16_Fig7_HTML.jpg)

**图 16-7** 数据库创建

你可以通过 `phpMyAdmin` 进行确认。见图 16-8。

![](img/514068_1_En_16_Fig8_HTML.png)

**图 16-8** 使用 `phpMyAdmin` 工具查看并确认更改

你需要一个实体来表示用户对象。通过运行以下命令创建它：

```bash
$ php bin/console make:entity
```

这将询问实体的名称，在你的例子中是 `User`，以及要定义的任何字段及其类型。

此命令的输出如图 16-9 所示。

![](img/514068_1_En_16_Fig9_HTML.jpg)

**图 16-9** 创建 User 实体

同样，添加 `email` 和 `password` 字段。完成后，直接按回车而不输入任何值。做出这些更改后，你将看到一条成功消息，如图 16-10 所示。

![](img/514068_1_En_16_Fig10_HTML.jpg)

**图 16-10** User 实体已成功创建

让我们验证在 `src/Entity/User.php` 创建的文件内容：

```php
public function getName(): ?string
{
    return $this->name;
}
public function setName(string $name): self
{
    $this->name = $name;
    return $this;
}
public function getEmail(): ?string
{
    return $this->email;
}
public function setEmail(string $email): self
{
    $this->email = $email;
    return $this;
}
public function getPassword(): ?string
{
    return $this->password;
}
public function setPassword(string $password): self
{
    $this->password = $password;
    return $this;
}
```

您已将所有定义的列添加到 `User` 属性的属性中。现在可以使用此功能创建迁移，以便在数据库中创建表。运行以下命令：

```bash
php bin/console make:migration
```

该命令的输出如图 16-11 所示。

![Figure 16-11](img/514068_1_En_16_Fig11_HTML.jpg)

**图 16-11** 迁移已成功创建

让我们检查一下创建的迁移文件（在我们的案例中，它位于 `migrations/Version20220821091914.php`）：

```php
$this->addSql('CREATE TABLE user (id INT AUTO_INCREMENT NOT NULL, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL, password VARCHAR(255) NOT NULL, PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');
$this->addSql('CREATE TABLE messenger_messages (id BIGINT AUTO_INCREMENT NOT NULL, body LONGTEXT NOT NULL, headers LONGTEXT NOT NULL, queue_name VARCHAR(190) NOT NULL, created_at DATETIME NOT NULL, available_at DATETIME NOT NULL, delivered_at DATETIME DEFAULT NULL, INDEX IDX_75EA56E0FB7336F0 (queue_name), INDEX IDX_75EA56E0E3BD61CE (available_at), INDEX IDX_75EA56E016BA31DB (delivered_at), PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');
}
public function down(Schema $schema): void
{
    // 此 down() 迁移是自动生成的，请根据您的需求进行修改
    $this->addSql('DROP TABLE user');
    $this->addSql('DROP TABLE messenger_messages');
}
```

该文件包含用于迁移和回滚的 `up` 和 `down` 方法。它包含了创建表的 SQL 语句。这非常方便，因为您无需自己编写 SQL 语句。运行此迁移：

```
php bin/console doctrine:migrations:migrate
```

该命令的输出如图 16-12 所示。

![Figure 16-12](img/514068_1_En_16_Fig12_HTML.jpg)

**图 16-12** 数据库迁移

按回车键继续，如图 16-13 所示。

![Figure 16-13](img/514068_1_En_16_Fig13_HTML.jpg)

**图 16-13** 数据库迁移完成

让我们检查 phpMyAdmin 以查看 `user` 表和 `migrations` 表的创建情况，如图 16-14 和 16-15 所示。

![Figure 16-15](img/514068_1_En_16_Fig15_HTML.jpg)

**图 16-15** 用于检查创建的 phpMyAdmin 工具

![Figure 16-14](img/514068_1_En_16_Fig14_HTML.jpg)

**图 16-14** 用于查看用户迁移表的 phpMyAdmin 工具

`doctrine_migration_versions` 表是 Symfony 特有的内部表，用于跟踪迁移更改。图 16-16 快速展示了 `migrations` 表的架构，其中指明了您刚才执行的一次迁移。

![Figure 16-16](img/514068_1_En_16_Fig16_HTML.jpg)

**图 16-16** 迁移表架构

通过这些更改，如果您现在启动 Symfony 服务器，将会看到一些与连接数据库以及执行系统级查询相关的日志。

现在，既然 `user` 表已设置好，您就可以创建用户注册功能了。从功能上讲，您将构建三个子功能：

1.  用于加载注册视图表单并接受表单提交请求的控制器路由

2.  注册视图表单

3.  将用户数据存储到数据库中

#### 控制器路由

现在让我们看看如何使用控制器路由来加载注册视图表单并接受表单提交请求。

当您访问 `http://localhost:8000/register` URL 时，会得到如图 16-17 所示的页面，返回 404 未找到代码，因为您的控制器中没有此 URL 的路由。

![Figure 16-17](img/514068_1_En_16_Fig17_HTML.png)

**图 16-17** 注册网页

如果您看到消息 "The metadata storage is not up to date, please run the sync-metadata-storage command to fix this issue"，请运行以下命令修复：

```bash
$ php bin/console doctrine:migrations:sync-metadata-storage
```

让我们创建一个控制器来处理此路由：

```bash
$ php bin/console make:controller UserController
```

此命令的输出如图 16-18 所示。

![Figure 16-18](img/514068_1_En_16_Fig18_HTML.jpg)

**图 16-18** `UserController` 已创建

它创建了两个文件：

```
src/Controller/UserController.php
templates/user/index.xhtml.twig
```

让我们查看 `src/Controller/UserController.php` 的内容：

```php
render('user/index.xhtml.twig', [
'controller_name' => 'UserController',
]);
}
}
```

将路由从 `/user` 修改为 `/register`，并在浏览器中重新加载 `http://localhost:8000/register`：

```php
class UserController extends AbstractController
{
#[Route('/register', name: 'app_user')]
```

代码的输出如图 16-19 所示。

![Figure 16-19](img/514068_1_En_16_Fig19_HTML.png)

**图 16-19** `UserController` 代码输出

检查上述代码，如果未提及 `GET` 或 `POST` 方法，则默认为 `GET` 请求，`/register` 是 `GET` 请求的相对 URI 路径。当控制器拦截到对 `/register` 路径的 `GET` 请求时，它会通过调用 `render` 函数返回 `user/index.xhtml.twig` 页面。稍后您将开发注册表单。现在，让我们为注册成功页面创建一个示例视图页面，并为接受注册提交的表单提交请求创建另一个路由。请参阅 `templates/user/registration_success.xhtml.twig` 中的注册成功页面：您已成功注册！

使用以下代码更新 `src/Controller/UserController.php` 中的 `GET` 路由，以添加 POST 请求处理程序：

```php
....
use Symfony\Component\HttpFoundation\Request;
....
#[Route('/register', name: 'app_user', methods: ['GET', 'POST'])]
public function index(Request $request): Response
{
if ($request->isMethod('POST') {
return $this->render('user/registration_success.xhtml.twig', [
'controller_name' => 'UserController',
]);
}
...
// Previous code for GET response
}
```

上述路由现在处理对 `/register` 路由的 `GET` 和 `POST` 请求，当向 `/register` 路径发送 `POST` 请求时，它返回注册成功页面。您已省略数据处理，稍后将完成。

要测试此更改，请使用`curl`命令打开终端，或使用 Postman。Postman 是一个用于运行 API 请求的 UI 界面。有关安装和使用的更多文档，请访问[`https://learning.postman.com/docs/getting-started/introduction/`](https://learning.postman.com/docs/getting-started/introduction/)。

`curl`请求命令如下：

```bash
curl --location --request POST 'http://localhost:8000/register'
```

此命令的输出如图 16-20 所示。

![Figure 16-20](img/514068_1_En_16_Fig20_HTML.jpg)

**图 16-20** `Register`命令输出

Postman 请求如图 16-21 所示。

![Figure 16-21](img/514068_1_En_16_Fig21_HTML.png)

**图 16-21** Postman 工具中的`Register`命令输出

### 注册视图表单

Symfony 的 Twig 模板引擎提供了许多内置功能，可根据需要加载、循环、解析数据和使用内置函数。您将学习如何创建一个接受几个字段输入并将其提交到`/register POST`路由的表单。

要添加表单功能，请通过运行以下命令安装包含此功能的包：

```bash
composer require symfony/form
```

Symfony 允许您通过`FormBuilder`方法初始化表单，并将字段与实体关联，而无需显式管理大量验证。

第一个更改是在`index`方法内部创建 User 实体的实例：

```php
...
use App\Entity\User;
...
public function register(Request $request, ManagerRegistry $doctrine): Response
{
$user = new User();
...
```

现在，您将使用表单构建器方法构建表单，并将其绑定到`$user`实例：

```php
$form = $this->createFormBuilder($user) // 将 $user 绑定到 $form 实例
// 将实体名称字段关联到 $form 并显示为文本类型字段
->add('name', TextType::class)
// 将实体电子邮件字段关联到 $form 并显示为电子邮件类型字段
->add('email', TextType::class)
// 将实体密码字段关联到 $form 并显示为密码类型字段
->add('password', PasswordType::class)
// 最后添加一个提交按钮
->add('save', SubmitType::class, ['label' => 'Register'])
->getForm();
```

一旦表单构建器实例创建完成，您需要将`$form`关联并传递到视图层，即`user/index.xhtml.twig`。通过使用以下内容更新`index`渲染调用来完成此操作：

```php
return $this->renderForm('user/index.xhtml.twig', [
'form' => $form,
]);
```

开始对`user/index.xhtml.twig`中的视图文件进行更改，将所有内容替换为以下代码以使用创建的表单：

```twig
{% extends 'base.xhtml.twig' %}
{% block title %}Registration Page!{% endblock %}
{% block body %}
{{ form(form) }}
{% endblock %}
```

在此代码中，您已将标题和块主体替换为对`form(form)`的简单调用，该调用会加载传入的表单。这将创建一个包含三个字段和一个用于 XSS 防护的 csrf 令牌的基本表单 HTML 元素，如开发工具检查图 16-22 所示。

![Figure 16-22](img/514068_1_En_16_Fig22_HTML.png)

**图 16-22** 基本注册视图表单示例

最终的控制器代码应如下所示：

```php
createFormBuilder($user)
->add('name', TextType::class)
->add('email', EmailType::class)
->add('password', PasswordType::class)
->add('save', SubmitType::class, ['label' => 'Register'])
->getForm();
if ($request->isMethod('POST')) {
return $this->render('user/registration_success.xhtml.twig', [
'controller_name' => 'UserController',
]);
}
return $this->renderForm('user/index.xhtml.twig', [
'form' => $form,
]);
}
}
```

尝试使用一些示例值填写字段详细信息，然后单击 Register 按钮，以查看表单的`POST`提交请求的实际效果。

提交后，它将显示如图 16-23 所示的输出。

![Figure 16-23](img/514068_1_En_16_Fig23_HTML.jpg)

**图 16-23** 基本注册视图表单网页输出

## 在数据库中存储用户数据

您已经设置了视图和控制器，但尚未对数据进行任何操作。其中一个重要部分是解析`POST`请求提交的数据，进行验证，然后保存到数据库。

现在您已经能够将数据发布到路由端点，您可以解析不同的值。这些值作为 Request 对象的一部分可用，并可用于与表单实体关联。让我们在实践中看看。在`$form`实例化之后，对`index`方法进行以下更改：

```php
...
$form->handleRequest($request);
...
```

现在有了上述更改，已与您的 User 实体关联的`$form`将拥有填充到各自字段的提交值，并可用于获取设置为这些值的 User 实体：

```php
$user = $form->getData();
```

在获取表单数据之前，你需要验证一切是否正常，并且需要将`isMethod()`函数调用检查替换为表单提供的一个便捷方法，用来检查表单是否已提交，以确认这是一个 POST 请求：

```php
if ($form->isSubmitted() && $form->isValid()) {
$user = $form->getData();
```

现在，通过 Doctrine EntityManager 将数据保存到数据库。在保存之前，出于安全考虑，你需要对密码进行加密。添加`ManagerRegistry`和`UserPasswordHasherInterface`的命名空间，并将它们作为参数传递给`index()`，这样 Symfony 就能通过依赖注入来实例化它们并传递给该函数。

```php
use Doctrine\Persistence\ManagerRegistry;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;
...
public function index(
Request $request,
ManagerRegistry $doctrine,
UserPasswordHasherInterface $passwordHasher,
): Response
{
...
$user->setPassword(
$passwordHasher->hashPassword($user, $user->getPassword())
);
$entityManager = $doctrine->getManager();
$entityManager->persist($user);
$entityManager->flush();
```

这将把用户详细信息保存到`user`表中，并向用户返回注册成功模板。请注意，这里使用了哈希助手对密码进行单向哈希处理，以便密码以加密格式存储在数据库中，确保安全性。

现在，整个代码应该如下所示：

```
现在让我们也检查一下数据库，确认用户详细信息已存储为加密密码，如图 16-24 所示。

![](img/514068_1_En_16_Fig24_HTML.png)

图 16-24：检查包含加密密码的数据库

## 总结

在本章中，你了解了开始使用 Symfony 的基本要素。你设置并配置了一个数据库，并添加了数据。你还学习了 Symfony 的一些主要特性。不过，在 Composer 资源库中还有更多实用的特性和大量软件包等待你去发现。这是本书的最后一章。你已经掌握了 PHP 8 的基础知识！
```