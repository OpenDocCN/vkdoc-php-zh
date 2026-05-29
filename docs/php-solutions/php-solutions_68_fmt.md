# PHP 解决方案 17-2：通过数据库验证用户凭证

本 PHP 解决方案演示了如何通过查询数据库来验证用户存储的凭证，即先查找用户名的加密密码，然后将其与用户提交的密码一起作为参数传递给 `password_verify()`。如果 `password_verify()` 返回 `true`，则将用户重定向到受限页面。

将 `ch17` 文件夹中的 `login_db.php`、`menu_db.php` 和 `secretpage_db.php` 复制到 `authenticate` 文件夹。同时将 `ch17` 文件夹中的 `logout_db.php` 和 `session_timeout_db.php` 复制到 `includes` 文件夹。

这样就搭建了与第 9 章相同的基础测试平台。唯一的区别是链接已修改为重定向到 `authenticate` 文件夹。

在 `login_db.php` 的 `DOCTYPE` 声明上方的 PHP 代码块中添加以下代码：

```php
$error = '';

if (isset($_POST['login'])) {

session_start();

$username = trim($_POST['username']);

$password = trim($_POST['pwd']);

// 成功登录后重定向的位置

$redirect = 'http://localhost/phpsols/authenticate/menu_db.php';

require_once '../includes/authenticate_mysqli.php';

}
```

这段代码遵循了与第 9 章登录表单中相似的模式。首先将 `$error` 初始化为空字符串。条件语句在表单提交后启动会话。从用户输入字段中去除空白字符，并将成功登录后用户重定向的页面位置存储在变量中。最后引入身份验证脚本，该脚本将在下一步中构建。

如果使用 PDO，请使用 `authenticate_pdo.php` 作为处理脚本。

创建一个名为 `authenticate_mysqli.php` 或 `authenticate_pdo.php` 的新文件，并将其保存在 `includes` 文件夹中。该文件仅包含 PHP 脚本，因此请去除任何 HTML 标记。引入数据库连接文件，使用只读账户创建数据库连接，并使用预处理语句获取用户详细信息。

对于 MySQLi，请使用以下代码：

```php
<?php

require_once 'connection.php';

$conn = dbConnect('read');

// 从数据库中获取用户名的加密密码

$sql = 'SELECT pwd FROM users WHERE username = ?';

// 初始化并准备语句

$stmt = $conn->stmt_init();

$stmt->prepare($sql);

// 绑定输入参数

$stmt->bind_param('s', $username);

$stmt->execute();

// 绑定结果，使用新变量存储密码

$stmt->bind_result($storedPwd);

$stmt->fetch();
```

这是一个非常直接的 `SELECT` 查询，因此在将其传递给 MySQLi 的 `prepare()` 方法时，我没有使用条件语句。用户名是一个字符串，因此 `bind_param()` 的第一个参数是 `'s'`。如果找到匹配项，结果将绑定到 `$storedPwd`。你需要使用一个新变量来存储密码，以避免覆盖用户提交的密码。

语句执行后，`fetch()` 方法获取结果。

对于 PDO，请改用以下代码：

```php
<?php

require_once 'connection.php';

$conn = dbConnect('read', 'pdo');

// 从数据库中获取用户名的加密密码

$sql = 'SELECT pwd FROM users WHERE username = ?';

// 准备语句

$stmt = $conn->prepare($sql);

// 将输入参数作为单元素数组传递

$stmt->execute([$username]);

$storedPwd = $stmt->fetchColumn();
```

这段代码与 MySQLi 版本的功能相同，但使用了 PDO 语法。用户名作为单元素数组传递给 `execute()` 方法。由于结果中只有一列，`fetchColumn()` 返回值并将其赋值给 `$storedPwd`。

获取用户名的密码后，只需将提交的密码和存储的密码传递给 `password_verify()`。如果 `password_verify()` 返回 `true`，则创建会话变量以指示成功登录和会话开始时间，重新生成会话 ID，并重定向到受限页面。否则，在 `$error` 中存储错误消息。

在上一步输入的代码之后插入以下代码。该代码对于 MySQLi 和 PDO 相同。

```php
// 检查提交的密码是否与存储的密码匹配

if (password_verify($password, $storedPwd)) {

$_SESSION['authenticated'] = 'Jethro Tull';

// 获取会话开始时间

$_SESSION['start'] = time();

session_regenerate_id();

header("Location: $redirect");

exit;

} else {

// 如果未验证，准备错误消息

$error = 'Invalid username or password';

}
```

与第 9 章一样，`$_SESSION['authenticated']` 的值实际上并不重要。

保存 `authenticate_mysqli.php` 或 `authenticate_pdo.php`，然后使用在 PHP 解决方案 17-1 末尾注册的用户名和密码登录，测试 `login_db.php`。登录过程应与第 9 章完全相同。区别在于所有详细信息都更安全地存储在数据库中。

如有必要，你可以对照 `login_mysqli.php` 和 `authenticate_mysqli.php`，或 `login_pdo.php` 和 `authenticate_pdo.php` 检查代码，这些文件均位于 `ch17` 文件夹中。如果遇到问题，最常见的错误是在数据库中为加密密码创建的列太窄。它必须至少 60 个字符宽，建议使其能够存储多达 255 个字符，以备将来加密方法生成更长的字符串。

尽管将加密密码存储在数据库中比使用文本文件更安全，但密码是从用户浏览器以纯文本（未加密）形式发送到服务器的。这对于大多数网站来说是足够的，但如果需要更高的安全级别，则登录和后续页面的访问应通过安全套接层（SSL）连接进行。

## 使用双向加密

设置用户注册和双向加密身份验证的主要区别在于，密码需要以 `BLOB` 数据类型（有关更多信息，请参见第 10 章中的“存储二进制数据”）作为二进制对象存储在数据库中，并且密码验证在 SQL 查询中执行，而不是在 PHP 脚本中。

### 创建存储用户详细信息的表

在 phpMyAdmin 中，在 `phpsols` 数据库中创建一个名为 `users_2way` 的新表。它需要三列，设置如表 17-2 所示。

**表 17-2. `users_2way` 表的设置**

| 名称 | 类型 | 长度/值 | 属性 | 空值 | 索引 | A_I |
| --- | --- | --- | --- | --- | --- | --- |
| `user_id` | `INT` | | `UNSIGNED` | 取消选择 | `PRIMARY` | 选择 |
| `username` | `VARCHAR` | `15` | | 取消选择 | `UNIQUE` | |
| `pwd` | `BLOB` | | | 取消选择 | | |

### 注册新用户

`AES_ENCRYPT()`函数接受两个参数：要加密的值和加密密钥。加密密钥可以是您选择的任意字符串。在本示例中，我选择了`takeThisWith@PinchOfSalt`，但使用随机字母数字字符和符号组合会更安全。

单向加密和双向加密的基本注册脚本是相同的。唯一区别在于将用户数据插入数据库的部分。

> **提示**
>
> 以下脚本直接将加密密钥嵌入页面中。如果您在服务器根目录之外有一个私有文件夹，建议在包含文件中定义密钥并将其存储在私有文件夹中。

MySQLi 的代码如下（完整代码见 `ch17` 文件夹中的 `register_2way_mysqli.php`）：

### 使用双向加密进行用户认证

创建带有双向加密的登录页面非常简单。连接数据库后，将用户名、密钥和未加密的密码整合在 `SELECT` 查询的 `WHERE` 子句中。如果查询找到匹配项，则允许用户进入网站的受限部分；如果没有匹配项，则拒绝登录。代码与 PHP 解决方案 17-2 基本相同，唯一下面部分不同。

对于 MySQLi，代码如下（见 `authenticate_2way_mysqli.php`）：

```php
<?php
require_once 'connection.php';
$conn = dbConnect('read');

// 创建密钥
$key = 'takeThisWith@PinchOfSalt';

$sql = 'SELECT username FROM users_2way
WHERE username = ? AND pwd = AES_ENCRYPT(?, ?)';

// 初始化并准备语句
$stmt = $conn->stmt_init();
$stmt->prepare($sql);

// 绑定输入参数
$stmt->bind_param('sss', $username, $password, $key);
$stmt->execute();

// 要获取匹配数量，必须存储结果
$stmt->store_result();

// 如果找到匹配项，num_rows 为 1，视为 true
if ($stmt->num_rows) {
    $_SESSION['authenticated'] = 'Jethro Tull';
    // 获取会话开始时间
    $_SESSION['start'] = time();
    session_regenerate_id();
    header("Location: $redirect"); exit;
} else {
    // 如果未验证，准备错误消息
    $error = '用户名或密码无效';
}
```

请注意，在访问 `num_rows` 属性之前，必须先存储预处理语句的结果。如果未能执行此操作，`num_rows` 将始终为 `0`，即使用户名和密码正确，登录也会失败。

PDO 的修订代码如下（见 `authenticate_2way_pdo.inc.php`）：

```php
<?php
require_once 'connection.php';
$conn = dbConnect('read', 'pdo');

// 创建密钥
$key = 'takeThisWith@PinchOfSalt';

$sql = 'SELECT username FROM users_2way
WHERE username = ? AND pwd = AES_ENCRYPT(?, ?)';

// 准备语句
$stmt = $conn->prepare($sql);

// 通过在执行语句时以数组形式传递来绑定变量
$stmt->execute([$username, $password, $key]);

// 如果找到匹配项，rowCount() 返回 1，视为 true
if ($stmt->rowCount()) {
    $_SESSION['authenticated'] = 'Jethro Tull';
    // 获取会话开始时间
    $_SESSION['start'] = time();
    session_regenerate_id();
    header("Location: $redirect"); exit;
} else {
    // 如果未验证，准备错误消息
    $error = '用户名或密码无效';
}
```

要测试这些脚本，请用它们替换 `authenticate_mysqli.php` 和 `authenticate_pdo.php`。

### 解密密码

解密使用双向加密的密码，只需在预处理语句中将密钥作为第二个参数传递给 `AES_DECRYPT()`，如下所示：

```php
$key = 'takeThisWith@PinchOfSalt';
$sql = "SELECT AES_DECRYPT(pwd, '$key') AS pwd FROM users_2way
WHERE username = ?";
```

密钥必须与最初用于加密密码的密钥完全相同。如果丢失密钥，密码将像使用单向加密存储的一样无法访问。

通常，只有在用户请求密码提醒时才需要解密密码。为此类提醒制定适当的安全策略在很大程度上取决于您运营的网站类型。然而，不言而喻，您不应该在屏幕上显示解密后的密码。您需要设置一系列安全检查，例如询问用户的出生日期或母亲的婚前姓名，或者提出一个只有用户本人可能知道答案的问题。即使用户回答正确，您也应该通过电子邮件将密码发送到用户的注册地址。

如果您已成功阅读本书至此，那么您应该能够轻松掌握所有必要知识。

## 更新用户详细信息

我没有为用户注册页面包含任何更新表单。这是一个现阶段您应该能够自行完成的任务。更新用户注册详细信息最重要的一点是，不应在更新表单中显示用户现有的密码。如果使用单向加密，您无论如何也无法显示它。