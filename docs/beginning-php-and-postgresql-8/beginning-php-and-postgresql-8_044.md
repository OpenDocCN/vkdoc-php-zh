# 将用户身份验证集成到 Web 应用中

将用户身份验证直接集成到 Web 应用程序逻辑中既方便又灵活；方便之处在于它整合了原本需要一定程度进程间通信的操作，而灵活之处在于集成的身份验证为与应用程序的其他组件（例如内容个性化和用户权限指定）集成提供了一种更简单的方法。在本章的剩余部分，我们将研究 PHP 内置的身份验证功能，并演示几种你可以立即开始集成到应用程序中的身份验证方法。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 14 章：身份验证

#### 身份验证变量

PHP 使用两个预定义变量来验证用户身份：`$_SERVER['PHP_AUTH_USER']` 和 `$_SERVER['PHP_AUTH_PW']`。这些变量分别保存身份验证所需的两个组件，即用户名和密码。它们的用法将在以下示例中变得清晰。然而，目前在使用这些预定义变量时，有两个重要的注意事项需要牢记：

- 每个受限页面开始时都必须验证这两个变量。你可以通过包装每个受限页面轻松实现这一点，这意味着将身份验证代码放在一个单独的文件中，然后使用 `REQUIRE()` 函数将该文件包含到受限页面中。
- 这些变量在 PHP 的 CGI 版本中无法正常工作，在 Microsoft IIS 上也无法正常工作。请参阅关于 PHP 身份验证和 IIS 的补充说明。

##### PHP 身份验证和 IIS

如果你正在将 IIS 与 PHP 的 ISAPI 模块结合使用，并且希望使用 PHP 的 HTTP 身份验证功能，则需要对本章提供的示例进行小幅修改。在使用 IIS 时，用户名和密码变量仍然对 PHP 可用，但不是通过 `$_SERVER['PHP_AUTH_USER']` 和 `$_SERVER['PHP_AUTH_PW']`。相反，这些值必须从另一个服务器全局变量 `$_SERVER['HTTP_AUTHORIZATION']` 中解析出来。因此，例如，你需要像这样解析这些变量：

```
list($user, $pswd) = explode(':', base64_decode(substr($_SERVER['HTTP_AUTHORIZATION'], 6)));
```

#### 有用的函数

通过 PHP 处理身份验证时，通常使用两个标准函数：`header()` 和 `isset()`。本节将介绍这两个函数。

##### `header()`

`void header(string *string* [, boolean *replace* [, int *http_response_code*]])`

`header()` 函数向浏览器发送原始的 HTTP 标头。`string` 参数指定发送给浏览器的标头信息。可选的 `replace` 参数决定此信息是替换还是附加到先前发送的标头。最后，可选的 `http_response_code` 参数定义了一个伴随标头信息的特定响应代码。请注意，你可以将此代码包含在 `string` 中，稍后将演示这一点。应用于用户身份验证时，此函数对于向浏览器发送 WWW 身份验证标头非常有用，从而显示弹出式身份验证提示。如果提交了不正确的身份验证凭据，它也可用于向用户发送 401 标头消息。示例如下：

```php
<?php
header('WWW-Authenticate: Basic Realm="Book Projects"');
header("HTTP/1.1 401 Unauthorized");
...
?>
```

请注意，除非启用了输出缓冲，否则这些命令必须在返回任何输出之前执行。忽略此规则将导致服务器错误，因为这违反了 HTTP 规范。

##### `isset()`

`boolean isset(mixed *var* [, mixed *var* [,...]])`

`isset()` 函数判断一个变量是否已被赋值。如果变量包含值，则返回 `TRUE`；否则返回 `FALSE`。应用于用户身份验证时，`isset()` 函数可用于判断 `$_SERVER['PHP_AUTH_USER']` 和 `$_SERVER['PHP_AUTH_PW']` 变量是否已正确设置。清单 14-1 提供了一个使用示例。

**清单 14-1.** *使用 `isset()` 验证变量是否包含值*

```php
<?php
if (isset($_SERVER['PHP_AUTH_USER']) and isset($_SERVER['PHP_AUTH_PW'])) {
    // 执行额外的身份验证任务
} else {
    echo "<p>请输入用户名和密码！</p>";
}
?>
```

#### 身份验证方法

有多种方法可以通过 PHP 脚本实现身份验证。当需要调用此类功能时，你应该考虑每种方法的范围和复杂性。具体来说，本节讨论了将登录信息对硬编码到脚本中、使用基于文件的身份验证、使用基于 IP 的身份验证、使用 PEAR 的 HTTP 身份验证功能以及使用基于数据库的身份验证。

##### 硬编码身份验证

限制资源访问的最简单方法是将用户名和密码直接硬编码到脚本中。清单 14-2 提供了一个如何实现此操作的示例。

**清单 14-2.** *针对硬编码登录信息对进行身份验证*

```php
if (($_SERVER['PHP_AUTH_USER'] != 'specialuser') ||
    ($_SERVER['PHP_AUTH_PW'] != 'secretpassword')) {
    header('WWW-Authenticate: Basic Realm="Secret Stash"');
    header('HTTP/1.0 401 Unauthorized');
    print('你必须提供正确的凭据！');
    exit;
}
```

此示例中的逻辑非常简单。如果 `$_SERVER['PHP_AUTH_USER']` 和 `$_SERVER['PHP_AUTH_PW']` 分别设置为 "specialuser" 和 "secretpassword"，则代码块不会执行，并且该块之后的任何内容都会执行。否则，系统会提示用户输入用户名和密码，直到提供正确的信息或由于多次身份验证失败而显示 401 Unauthorized 消息。

尽管使用硬编码的身份验证信息对配置起来非常快速且简单，但它有几个缺点。首先，就目前的代码而言，所有需要访问该资源的用户都必须使用相同的身份验证信息对。通常，在现实场景中，每个用户都必须被唯一标识，以便可以提供用户特定的偏好或资源。尽管你可以通过添加额外的逻辑来允许多个登录信息对，但生成的代码会非常笨拙。其次，更改用户名或密码只能通过进入代码并手动修改来完成。接下来的两种方法可以满足这一需求。

##### 基于文件的身份验证

通常你需要为每个用户提供一个唯一的登录信息对，从而可以记录用户特定的登录时间、移动和操作。你可以使用文本文件轻松实现这一点，该文件非常类似于通常用于存储 Unix 用户信息的文件（`/etc/passwd`）。清单 14-3 提供了这样一个文件。每行包含一个用户名和一个加密的密码对，这两个元素用冒号（`:`）分隔。

**清单 14-3.** *包含加密密码的 `authenticationFile.txt` 文件*

```
jason:60d99e58d66a5e0f4f89ec3ddd1d9a80
donald:d5fc4b0e45c8f9a333c0056492c191cf
mickey:bc180dbc583491c00f8a1cd134f7517b
```

关于 `authenticationFile.txt` 的一个关键安全考虑因素是，此文件应存储在服务器文档根目录之外。如果不是，攻击者可能通过暴力猜测发现该文件，从而泄露一半的登录组合。此外，尽管你可以选择跳过密码加密并以纯文本格式存储，但强烈不鼓励这种做法，因为如果文件权限配置不正确，有权访问服务器的用户可能能够查看登录信息。



## 第 14 章：身份验证

用于解析此文件并根据给定的登录对验证用户身份的 PHP 脚本，仅比用于根据硬编码验证对进行身份验证的脚本稍微复杂一些。区别在于，该脚本还必须将文本文件读入一个数组，然后遍历该数组以查找匹配项。这涉及到使用几个函数，包括以下内容：

- `file(string filename)`：`file()` 函数将文件读入一个数组，数组的每个元素对应文件中的一行。
- `explode(string separator, string string [, int limit])`：`explode()` 函数将一个字符串分割成一系列子字符串，每个字符串的边界由特定的分隔符确定。
- `md5(string str)`：`md5()` 函数使用 RSA Data Security Inc. 的 MD5 消息摘要算法（http://www.rsa.com）计算字符串的 MD5 哈希值。

**注意：** 尽管它们在功能上相似，但你应该使用 `explode()` 而不是 `split()`，因为 `split()` 调用了 PHP 的正则表达式解析引擎，速度稍慢。

清单 14-4 展示了一个 PHP 脚本，它能够解析 `authenticationFile.txt`，并可能将用户的输入与登录对匹配。

**清单 14-4.** *针对平面文件登录存储库验证用户*

```php
<?php

// 预设身份验证状态为 false

$authorized = FALSE;

if (isset($_SERVER['PHP_AUTH_USER']) && isset($_SERVER['PHP_AUTH_PW'])) {

    // 将身份验证文件读入一个数组

    $authFile = file("/usr/local/lib/php/site/authenticate.txt");

    // 遍历文件中的每一行，搜索身份验证匹配项
    foreach ($authFile as $login) {

        list($username, $password) = explode(":", $login);

        // 从密码中移除换行符

        $password = trim($password);

        if (($username == $_SERVER['PHP_AUTH_USER']) &&
            ($password == md5($_SERVER['PHP_AUTH_PW']))) {

            $authorized = TRUE;

            break;

        }

    }

}

// 如果未授权，则显示身份验证提示或 401 错误

if (! $authorized) {

    header('WWW-Authenticate: Basic Realm="Secret Stash"');
    header('HTTP/1.0 401 Unauthorized');

    print('你必须提供正确的凭据！');

    exit;

}

// 受限制的内容放在这里...

?>
```

尽管基于文件的身份验证系统对于相对较小且静态的身份验证列表效果很好，但当你处理大量用户、用户被频繁添加、删除和修改，或者需要将身份验证方案集成到更大的信息基础设施中时（例如，集成到预先存在的用户表中），这种策略可能变得有些不方便。这些需求通过实现基于数据库的解决方案可以更好地满足。下一节将演示这样一个解决方案，使用 PostgreSQL 数据库来存储身份验证对。

##### 基于数据库的身份验证

在本章讨论的所有各种身份验证方法中，实现基于数据库的解决方案是最强大的方法，因为它不仅增强了管理便利性和可伸缩性，而且可以集成到更大的数据库基础设施中。出于本示例的目的，我们将数据存储限制为四个字段——一个主键、用户的全名、一个用户名和一个密码。这些列被放入一个我们称之为 `userauth` 的表中，如清单 14-5 所示。

**注意：** 如果你不熟悉 PostgreSQL 服务器，并对以下示例中的语法感到困惑，请考虑复习第 30 章中的内容。

**清单 14-5.** *一个用户身份验证表*

```sql
create table userauth (

    rowid serial,

    commonname varchar(35) not null,

    username varchar(8) not null,

    pswd varchar(32) not null,

    CONSTRAINT userauth_id PRIMARY KEY(rowid)

);
```

清单 14-6 展示了用于根据存储在 `userauth` 表中的信息验证用户提供的用户名和密码的代码。

**清单 14-6.** *针对 PostgreSQL 表验证用户*

```php
<?php

/* 因为身份验证提示需要被调用两次，

   所以将其嵌入到一个函数中。

*/

function authenticate_user() {

    header('WWW-Authenticate: Basic realm="Secret Stash"');
    header("HTTP/1.0 401 Unauthorized");

    exit;

}

/* 如果 $_SERVER['PHP_AUTH_USER'] 为空，则用户尚未被提示输入身份验证信息。

*/

if (! isset($_SERVER['PHP_AUTH_USER'])) {

    authenticate_user();

} else {

    // 连接到 PostgreSQL 数据库

    $conn = pg_connect("host=localhost dbname=corporate

                        user=authentication password=secret")
                        or die(pg_last_error($conn));

    // 创建并执行选择查询

    $query = "SELECT username, pswd FROM userauth
              WHERE username='$_SERVER[PHP_AUTH_USER]' AND
              pswd=md5('$_SERVER[PHP_AUTH_PW]')";

    $result = pg_query($conn, $query);

    // 如果没有找到任何内容，则重新提示用户输入登录信息
    if (pg_num_rows($result) == 0) {

        authenticate_user();

    } else {

        echo "欢迎访问秘密档案库！";

    }

}

?>
```

尽管 PostgreSQL 身份验证比前两种方法更强大，但实现起来其实非常简单。只需使用输入的用户名和密码作为查询条件，对 `userauth` 表执行一个选择查询即可。当然，这种解决方案并不依赖于特定使用 PostgreSQL 数据库；任何关系型数据库都可以替代它。

##### 基于 IP 的身份验证

有时你需要更高级别的访问限制来确保用户的有效性。当然，用户名/密码组合并非万无一失；这些信息可以透露给其他人，或者从用户那里窃取。它也可能通过推断或暴力破解被猜到，尤其是在用户选择了较差的登录组合的情况下，这种情况仍然相当普遍。

为了解决这个问题，进一步加强身份验证有效性的一种有效方法是，不仅要求有效的用户名/密码登录对，还要求特定的 IP 地址。为此，你只需稍微修改上一节中使用的 `userauth` 表，并对清单 14-6 中使用的查询进行微小修改。首先是表，如清单 14-7 所示。

**清单 14-7.** *重新审视 userauth 表*

```sql
create table userauth (

    rowid serial,

    commonname varchar(35) not null,

    username varchar(8) not null,

    pswd varchar(32) not null,

    ipaddress varchar(15) not null,

    CONSTRAINT userauth_id PRIMARY KEY(rowid)

);
```

用于验证用户名/密码和 IP 地址的代码如清单 14-8 所示。

**清单 14-8.** *使用登录对和 IP 地址进行身份验证*

```php
<?php

function authenticate_user() {

    header('WWW-Authenticate: Basic realm="Secret Stash"');
    header("HTTP/1.0 401 Unauthorized");

    exit;

}

if(! isset($_SERVER['PHP_AUTH_USER'])) {

    authenticate_user();

} else {

    // 连接到 PostgreSQL 数据库

    $conn = pg_connect("host=localhost dbname=corporate

                        user=authentication password=secret")
                        or die(pg_last_error($conn));

    // 创建并执行选择查询

    $query = "SELECT username, pswd FROM userauth
              WHERE username='$_SERVER[PHP_AUTH_USER]' AND
              pswd=MD5('$_SERVER[PHP_AUTH_PW]') AND
              ipaddress='$_SERVER[REMOTE_ADDR]'";

    $result = pg_query($conn, $query);

    // 如果没有找到任何内容，则重新提示用户输入登录信息
    if (pg_num_rows($result) == 0) {

        authenticate_user();

    } else {

        echo "欢迎访问秘密档案库！";

    }

}

?>
```



尽管这一额外的安全层效果不错，但你需要明白它并非万无一失。IP 欺骗（即诱骗网络，使其误以为流量来自某个特定 IP 地址）这一手段，长期以来一直是精明攻击者的惯用工具。

因此，如果攻击者获取了用户的用户名和密码，他们就有可能绕过你基于 IP 的安全障碍。

#### 利用 `PEAR: Auth_HTTP`

虽然前面讨论的认证方法已经足够好用，但将一些实现细节封装在类中总是更令人愉悦。PEAR 类 `Auth_HTTP` 很好地满足了这一需求，它利用 Apache 的认证机制和提示（见图 14-1）生成相同的提示，但使用 PHP 来管理认证信息。`Auth_HTTP` 封装了用户认证中许多琐碎的方面，通过一个便捷的接口将我们所需的信息和功能暴露出来。此外，由于它继承自 `Auth` 类，`Auth_HTTP` 还提供了广泛的认证存储机制，其中包括 DB 数据库抽象包、LDAP、POP3、IMAP、RADIUS 和 SAMBA。在本节中，我们将演示如何利用 `Auth_HTTP` 将用户认证信息存储在平面文件中。

##### 安装 `Auth_HTTP`

要利用 `Auth_HTTP` 的功能，你需要从 PEAR 安装它。因此，启动 PEAR 并传入以下参数：

```
%>pear install -o auth_http
```

由于 `auth_http` 依赖于另一个包（`Auth`），你至少应传入 `-o` 选项，该选项将安装这个必需的包。执行此命令后，你将看到类似下面的输出：

```
downloading Auth_HTTP-2.1.6.tgz ...
Starting to download Auth_HTTP-2.1.6.tgz (9,327 bytes)
.....done: 9,327 bytes
downloading Auth-1.2.3.tgz ...
Starting to download Auth-1.2.3.tgz (24,040 bytes)
...done: 24,040 bytes
```

```
skipping Package 'auth' optional dependency 'File_Passwd'
skipping Package 'auth' optional dependency 'Net_POP3'
skipping Package 'auth' optional dependency 'DB'
skipping Package 'auth' optional dependency 'MDB'
skipping Package 'auth' optional dependency 'Auth_RADIUS'
skipping Package 'auth' optional dependency 'File_SMBPasswd'
Optional dependencies:
package 'File_Passwd' version >= 0.9.5 is recommended to utilize some features.
package 'Net_POP3' version >= 1.3 is recommended to utilize some features.
package 'MDB' is recommended to utilize some features.
package 'Auth_RADIUS' is recommended to utilize some features.
package 'File_SMBPasswd' is recommended to utilize some features.
install ok: Auth 1.2.3
install ok: Auth_HTTP 2.1.6
%>
```

安装完成后，你就可以开始利用 `Auth_HTTP` 的功能了。为了演示，我们将考虑如何针对 PostgreSQL 数据库进行认证。

##### 针对 PostgreSQL 数据库进行认证

由于 `Auth_HTTP` 是 `Auth` 包的子类，它继承了 `Auth` 的所有功能。而 `Auth` 是 `DB` 包的子类，因此 `Auth_HTTP` 可以利用这个流行的数据库抽象层将认证信息存储在数据库表中。为了存储信息，我们将使用一个与本章前面使用的完全相同的表：

```sql
create table userauth (
rowid serial,
commonname varchar(35) not null,
username varchar(8) not null,
pswd varchar(32) not null,
CONSTRAINT userauth_id PRIMARY KEY(rowid)
);
```

接下来，我们需要创建一个脚本，调用 `Auth_HTTP` 并告知它引用一个 PostgreSQL 数据库。该脚本如清单 14-9 所示。

**清单 14-9.** *使用 Auth_HTTP 验证用户凭据*

```php
<?php
require_once("Auth/HTTP.php");

// 指定认证凭据、表名、
// 用户名和密码列、密码加密类型、
// 以及用于检索其他字段的查询参数
$dblogin = array (
'dsn' => "pgsql://corpweb:secret@localhost/corporate",
'table' => "userauth",
'usernamecol' => "username",
'passwordcol' => "pswd",
'cryptType' => "md5"
'db_fields' => "*"
);

// 实例化 Auth_HTTP
$auth = new Auth_HTTP("DB", $dblogin) or die("blah");

// 开始认证过程
$auth->start();

// 认证失败时提供的消息
$auth->setCancelText('认证凭据不被接受！');

// 检查凭据。如果不存在，则提示输入
if($auth->getAuth())
{
echo "欢迎, $auth->commonname<br />";
}
?>
```

执行清单 14-9 中的代码，并传入与 `userauth` 表中信息匹配的数据，用户就能进入受限区域。否则，用户将收到 `setCancelText()` 中提供的错误消息。

代码中的注释应该足以引导你理解代码含义，也许除了 `$dblogin` 数组之外。这个数组与数据源类型的声明一起被传入 `Auth_HTTP` 的构造方法。有关可接受的数据源类型列表，请参阅 `http://pear.php.net/package/Auth_HTTP` 上的 `Auth_HTTP` 文档。该数组的第一个元素 `dsn` 代表数据源名称（DSN）。DSN 必须采用以下格式：

```
数据源标题:用户名:密码@主机名/数据库
```

因此，我们使用以下 DSN 登录 PostgreSQL 数据库：`pgsql://corpweb:secret@localhost/corporate`

如果是 MySQL 数据库，且其他条件相同，则 `数据源标题` 应设置为 `mysql`。有关可接受的 `数据源标题` 值的完整列表，请参阅 `http://pear.php.net/package/DB` 上的 `DB` 文档。

接下来的三个元素，即 `table`、`usernamecol` 和 `passwordcol`，分别代表存储认证信息的表、存储用户名的列名和存储密码的列名。

`cryptType` 元素指定密码是以明文形式还是以 MD5 哈希值的形式存储在数据库中。如果是明文存储，`cryptType` 应设置为 `none`；而如果是 MD5 哈希值存储，则应设置为 `md5`。

最后，`db_fields` 元素提供了用于检索其他表信息（例如 `commonname` 字段）的查询参数。

`Auth_HTTP`、其父类 `Auth` 以及 `DB` 数据库抽象类为用户提供了一组强大的功能，能够完成原本繁琐的任务。务必花时间访问 PEAR 网站，了解更多关于这些包的信息。

#### 用户登录管理

当你在应用程序中集成用户登录时，提供可靠的认证机制只是整个图景的一部分。如何确保用户选择了一个可靠的密码，其复杂度足以让攻击者无法将其作为可能的攻击途径？此外，你如何处理用户忘记密码这一不可避免的事件？本节将详细讨论这两个主题。

##### 密码设定

密码通常是在某种用户注册过程中设定的，通常是在用户注册成为网站会员时。除了提供各种信息（例如用户的姓名和电子邮件地址）外，用户通常还会被提示设定一个用户名和密码，以便以后登录网站。你将创建一个此类注册过程的工作示例，并使用以下表来存储用户数据：

```sql
create table userauth (
rowid serial,
commonname varchar(35) not null,
email varchar(55) not null,
username varchar(8) not null,
pswd varchar(32) not null,
CONSTRAINT userauth_id PRIMARY KEY(rowid)
);
```



