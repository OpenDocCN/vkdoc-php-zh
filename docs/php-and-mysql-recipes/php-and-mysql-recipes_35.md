# 第 15 章 ■ 使用 JSON 进行数据交换

`function addLink($name, $value) {`

```
if (!$this->links) {
    $this->links = new stdClass();
}
$this->links->$name = "http://localhost/$value";
}
```

```
class JsonApi {
    function addResource($data) {
        $this->data = $data;
    }
    function addLink($name, $value) {
        if (!$this->links) {
            $this->links = new stdClass();
        }
        $this->links->$name = "http://localhost/$value";
    }
    function __toString() {
        return json_encode($this, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES);
    }
}
```

```
$user = new JsonResource('user', 123);
$user->addAttribute('name', 'John Smith');
$user->addAttribute('age', 25);

$api = new JsonApi();
$api->addResource($user);
header("Content-Type: application/vnd.api+json");
echo $api;
```

这是一个非常基础的示例，仅涵盖了所支持的少数几个元素，但利用这个简单的结构就能为任何类型的数据生成有效的 API 响应。若要使其完整，还需要一种处理错误、关联及其他类型数据的方法。目前，GitHub 上有许多不同的项目，其完备性都高于此示例。

该脚本的输出如下所示：

```
{
    "data": {
        "type": "user",
        "id": 123,
        "links": {
            "self": "http://localhost/user/123"
        },
        "attributes": {
            "name": "John Smith",
            "age": 25
        }
    }
}
```

关于该标准及更多文档信息，请访问 [`jsonapi.org`](http://jsonapi.org/)。

## 技巧 15-5：调用 API

### 问题

调用 API 时，通常需要传递参数才能返回特定元素的数据。参数可以作为 GET 参数传递（即直接以键值对的形式添加到请求 URI 中），也可以作为 POST 参数传递（即作为请求体中的键值对）。

有些 API 要求使用 POST 请求，仅仅是为了让用户请求数据变得稍微困难一些（无法在浏览器地址栏中输入请求来使用`wget`获取数据）；还有一些 API 则要求设置特殊的请求头。那么，如何使用 PHP 和 JavaScript 来调用这类 API 呢？

### 解决方案

在 PHP 中，我们可以利用流上下文（stream context）来设置请求方法，并添加请求头和请求体数据。在 JavaScript 中，我们可以使用 jQuery 库中的 `$.post()` 函数。当然，使用 `XMLHTTP` 对象也是可行的，但这会需要多写几行代码。出于安全原因，大多数情况下，浏览器会阻止 `XMLHTTP` 向 HTML 文档来源域以外的任何域发送请求。

### 实现原理

第一个示例是一个 PHP 脚本，它构建了一个 HTML 表单，用户可以在其中输入端点 URI 以及若干键值对。该表单由 PHP 提交，用于创建 API 请求并显示请求结果。在处理 API 时，这可以作为一个简单的调试工具来使用。

```php
// 15_6.php
$response = null;

if ($_POST) {
    $params = [];
    foreach($_POST['key'] as $i=>$key) {
        if (!empty($key) && array_key_exists($i, $_POST['value'])) {
            $params = "$key=" . urlencode($_POST['value'][$i]);
        }
    }

    $options = array('http' =>
        array(
            'method' => $_POST['method'],
            'content' => implode("&", $params)
        )
    );

    $context = stream_context_create($options);
    $response = file_get_contents($_POST['url'], false, $context);

    switch ($_POST['response']) {
        case 0 :
            $json = json_decode($response, true);
            $response_text = json_encode($json, JSON_PRETTY_PRINT);
            break;
        case 1:
            $dom = dom_import_simplexml($response)->ownerDocument;
            $dom->formatOutput = true;
            $response_text = $dom->saveXML();
            break;
    }
}

echo <<<HEREDOC
<!DOCTYPE html>
<html>
<head>
<script src="https://code.jquery.com/jquery-1.12.2.min.js"
    integrity="sha256-lZFHibXzMHo3GGeehn1hudTAP3Sc0uKXBXAzHX1sjtk="
    crossorigin="anonymous"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"
    integrity="sha384-0mSbJDEHialfmuBBQP6A4Qrprq5OVfW37PRR3j5ELqxss1yVqOtnepnHVP9aJ7xS"
    crossorigin="anonymous"></script>
<link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"
    rel="stylesheet"
```



```html
integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7"
crossorigin="anonymous">

<script>
function AddParameter() {
    var $params = $('#parameters');
    if ($params) {
        var $div = $('<div>');
        $div.html($params.html());
        $div.insertAfter($params);
    }
}
</script>

echo <<<HEREDOC

<!DOCTYPE html>
<html>
<head>
<script src="https://code.jquery.com/jquery-1.12.2.min.js"
        integrity="sha256-lZFHibXzMHo3GGeehn1hudTAP3Sc0uKXBXAzHX1sjtk="
        crossorigin="anonymous"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"
        integrity="sha384-0mSbJDEHialfmuBBQP6A4Qrprq5OVfW37PRR3j5ELqxss1yVqOtnepnHVP9aJ7xS"
        crossorigin="anonymous"></script>
<link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7"
      crossorigin="anonymous">

<script>
function AddParameter() {
    var $params = $('#parameters');
    if ($params) {
        var $div = $('<div>');
        $div.html($params.html());
        $div.insertAfter($params);
    }
}
</script>

</head>
<body>
<form name="api" method="POST" action="{$_SERVER['PHP_SELF']}">
    <div>
        <lable>API 端点</label>
        <input class="form-control" type="text" name="url" />
    </div>
    <div>
        <lable>方法</label>
        <select class="form-control" name="method">
            <option value="GET">GET</option>
            <option value="POST">POST</option>
        </select>
    </div>
    <div>
        <lable>响应格式</label>
        <select class="form-control" name="response">
            <option value="0">JSON</option>
            <option value="1">XML</option>
        </select>
    </div>
    <div id="parameters">
        <lable>参数</label>
        <div style="width:30%; display: inline-block">
            <input class="form-control" type="text" name="key[]" />
        </div>
        <div style="width:50%; display: inline-block">
            <input class="form-control" type="text" name="value[]" />
        </div>
    </div>
</form>

<button onclick="AddParameter()">添加参数</button>
<button onclick="document.api.submit()">提交</button>

<pre id="response">
$response_text
</pre>

</body>
</html>
HEREDOC;

# 第 15 章 ■ 使用 JSON 进行数据交换

请注意 JavaScript 部分中的美元符号已被转义。美元符号用作 `jQuery()` 函数的简写 `$()`，也出现在某些变量名中，但由于 PHP 会在 `HEREDOC` 块中进行变量替换，这些未在 PHP 作用域中定义的变量将被替换为空字符串。

表单包含了用于输入 URL、方法、响应显示方式以及单个参数的键/值的字段。如果需要更多参数，用户可以点击 `添加参数` 按钮，表单将会扩展。

当表单提交后，PHP 端会根据提交的数据创建一个 HTTP 请求，并在 HTML 文档的预格式化区域显示来自 API 调用的响应。

## 配方 15-6. API 身份验证

### 问题

对 API 的用户进行身份验证的最佳方式是什么？

### 解决方案

首先，在调用 API 时你绝不应该以明文形式发送用户名和密码，并且也绝不应该将任何身份验证参数包含在 URI 中，因为这些参数会被服务器记录。这些信息应始终作为请求头或请求体的一部分发送。

API 提供者通常会为每个用户发放一个应用密钥或其他唯一标识。这通常是某种哈希值，无法关联到特定的用户账户。有些网站还会提供一个密钥代码，用于生成请求的哈希值，但该密钥代码不应随请求一起发送。服务器知道这个密钥，可以在服务器端生成哈希值，并与请求中包含的哈希值进行比较。如果两者匹配，服务器就可以假定请求者就是其所声称的身份。

哈希运算可以使用部分请求参数进行，但也可以包含诸如请求者的公网 IP 或用户代理字符串等信息，只要客户端和服务器使用相同的数据集来创建哈希值即可。
```


某些 API 要求客户端在每个请求上都进行身份验证，但服务器也可以在响应中包含一个会话令牌，来自同一客户端且包含该会话令牌的任何后续请求都将被视为有效。服务器可能会使用请求中的额外参数（如用户代理字符串和 IP 地址）来验证会话令牌。如果这些参数在请求之间发生变化，服务器将使会话失效。

### 工作原理

有多种生成应用密钥和机密（App Keys and Secrets）的方法。应用密钥可以通过对与账户相关的某些数据进行 SHA256 哈希运算来生成。

```php
<?php

$appkey = hash('sha256', $email . $first_name, $date);
```

在这个例子中，拼接电子邮件地址、名字和某个日期表示形式会创建一个字符串，然后用该字符串生成一个哈希值。这个哈希值随后会与账户一起存储。

对账户中用户唯一的不同值进行哈希运算可以生成一个机密，该机密仅由服务器和客户端知晓，并且永远不会作为请求的一部分被包含。

```php
<?php

$secret = hash('sha1', $account_id . $email);
```

在服务器端，应用密钥和机密将作为附加的账户信息保存。一个好的做法是，在生成机密时，只在网页上显示一次，以防止多次显示。如果用户忘记了机密，则应生成一个新的。

客户端和服务器之间必须就如何生成哈希值达成一致。这可以通过将所有参数作为字符串，加上机密，然后对该字符串生成哈希值来实现。该哈希值随后作为附加参数添加到请求中。在服务器端，除了哈希值之外的所有参数将被组合成类似的字符串，并使用应用密钥在数据库中查找机密，从而生成哈希值以验证提供的哈希值。

下一个示例展示了客户端如何使用包含 `$appkey` 和 `$secret` 变量的包含文件。

确保包含文件存储在 Web 根目录之外，以防止从浏览器访问它。

```php
<?php

// 15_7.php

require "api_credentials.inc";

$params = [

"appkey=$appkey",

"article=1"

];

$hash = hash('sha256', implode("", $params) . $secret);

$params['hash'] = $hash;

$options = array('http' =>

array(

'method' => "POST",

'content' => implode("&", $params)

)

);

$context = stream_context_create($options);

$response = file_get_contents("http://example.com/articels", false, $context);
```

如果 API 调用的服务器端由 PHP 脚本处理，其处理请求的方式如下：

```php
<?php

// 16_8.php

$params = [

"appkey={$_POST['appkey']}",

"article={$_POST['article']}"

];

$secret = GetSecret($_POST['appkey']);

$hash = hash('sha256', implode("", $params) . $secret);

if ($hash == $_POST['hash']) {

// Request is allowed

}

else {

// Error – the hash did not match

}
```

`GetSecret()` 函数调用应执行数据库查找，以根据提供的应用密钥获取机密的数值。

## 第 16 章 使用 MySQL 数据库

数据库在 Web 应用程序中扮演着重要角色，这些应用程序的内容不断变化，并根据用户的偏好、位置或其他参数进行提供。数据库也用于以博客评论、创建账户时的个人信息等形式从用户那里收集信息。

PHP 以一组函数的形式支持多种不同类型的数据库，用于连接、执行查询以及获取数据库中表和列的信息。其中一些数据库扩展是过程式的，而另一些则提供了面向对象的接口。名为 MySQL 的开源数据库是与 PHP 一起使用最广泛的数据库，Facebook、Twitter 和 Wikipedia 等大小公司都在使用它。该扩展在大多数发行版中默认提供。

`mysql` 扩展已被弃用，并由 `mysqli` 扩展取代。其中的 `I` 代表 Improved（改进版）。

从 PHP 7 开始，不再提供 `mysql` 扩展。




MySQL 数据库的源代码目前由 Oracle 公司拥有和维护，而 MySQL 原始团队开发了一个名为 MariaDB 的新开源版本。该数据库与 MySQL API 兼容，可作为 MySQL 的即插即用替代品。目前 CentOS 7 提供的标准数据库正是 MariaDB。

另一个优势在于其体积小巧，这使得 MySQL 能够用于硬件资源有限的环境或嵌入式系统。

本章将演示如何使用 `mysqli` 扩展从 PHP 脚本执行常见的数据库操作。

## 食谱 16-1. 连接 MySQL

### 问题

尽管数据库中的 MySQL 数据存储在文件系统中，但无法通过使用 `fopen()` 或 `file_get_contents()` 函数打开并读取/写入文件来访问。要发送命令并接收数据，需要让数据库服务器保持运行，并在客户端与服务器之间建立 TCP/IP 连接。如何在 PHP 中实现这一点呢？

### 解决方案

`mysqli` 扩展包含一个名为 `mysqli_connect()` 的函数。该函数需要主机名、用户 ID、密码、数据库名称、TCP 端口和套接字作为参数。如果连接成功，它将返回一个连接资源；如果连接失败，则返回 `false`。

也可以使用 PHP 数据对象（`PDO`）扩展来连接 MySQL。

© Frank M. Kromann 2016  
F.M. Kromann, *PHP and MySQL Recipes*, DOI 10.1007/978-1-4842-0605-8_16

第 16 章 ■ 使用 MySQL 数据库

所有参数都可以通过在 `php.ini` 中提供的配置选项来设置默认值。这些默认值定义如下：

```
mysqli.default_host = "127.0.0.1"
mysqli.default_user = "db_user"
mysqli.default_pw = "secret"
mysqli.default_port = 3306
mysqli.default_socket = ""
```

至少需要提供主机名、用户名和密码的值。如果指定了数据库，则会尝试连接到该数据库。随时可以使用 `mysqli_select_db()` 函数切换到同一主机下的其他数据库。

如果数据库服务器和 Web 服务器运行在同一台机器上，我们可以使用 `localhost` 或 IP 地址 `127.0.0.1` 作为数据库服务器的主机名。如果两个服务位于不同的服务器上，则需要提供远程主机名或 IP 地址。在高流量环境中，使用 IP 地址可以提供更好的性能。在这种情况下，客户端无需在每次连接请求时都执行 DNS 查询来将主机名解析为 IP 地址。

### 工作原理

要连接数据库，我们需要在服务器上创建一个数据库。可以通过在服务器上使用 `mysql` 命令来创建。

```
mysql –u root –p
```

该命令将使用 `root` 用户连接到本地主机上的 MySQL（或 MariaDB）服务器。

连接将是交互式的，可用于创建新的数据库、表，以及添加、更新或删除数据。

在此示例中，我们将创建一个名为 `music` 的表来存储专辑、曲目和艺术家信息。第一步是创建数据库：

```
create database music;
```

使用命令 `show databases;` 可以获取服务器上所有可用数据库的列表。

```
MariaDB [mysql]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| book               |
| music              |
| mysql              |
| performance_schema |
+--------------------+
5 rows in set (0.00 sec)
```

第 16 章 ■ 使用 MySQL 数据库

还需要创建一个数据库用户，并授予该用户对数据库的访问权限。这是从 PHP 连接数据库时将使用的用户账户。

```
MariaDB [mysql]> CREATE USER 'php'@'localhost' IDENTIFIED BY 'secret';
Query OK, 0 rows affected (0.01 sec)
```

这将创建一个名为 `php`、密码为 `secret` 的用户账户。最后一步是授予 `php` 用户对 `music` 数据库的访问权限。

```
MariaDB [mysql]> GRANT ALL PRIVILEGES ON music . * TO 'php'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

该语句将授予该用户对 `music` 数据库中所有表的访问权限。




退出`mysql`工具，然后使用新帐户重新登录：

```bash
mysql -u php -p
```

输入密码（`secret`），然后输入显示数据库的命令。

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| music              |
+--------------------+
2 rows in set (0.01 sec)
```

这将仅列出用户有权访问的数据库。`information_schema`数据库由 MySQL 用于存储有关表、列、键和索引等数据，这是一个所有用户都可以访问（只读）的数据库。

为了在数据库中存储数据，我们需要创建一个或多个表。每个表定义了多个不同数据类型（如整数、字符、日期时间等）的列。数据以行的形式添加到表中。

每行将包含每个列的一个值。如果模式定义允许，某些列的值可以为空，但所有行将具有相同数量的列。

```
MariaDB [music]> create table album (
    -> id int auto_increment,
    -> title varchar(250) not null,
    -> description varchar(2000),
    -> primary key(id)
    -> );
Query OK, 0 rows affected (0.02 sec)
```

第 16 章 ■ 使用 MySQL 数据库

`show tables;`和`describe album;`命令可用于显示可用表的列表以及`album`表的定义。

```
MariaDB [music]> show tables;
+-----------------+
| Tables_in_music |
+-----------------+
| album           |
+-----------------+
1 row in set (0.00 sec)

MariaDB [music]> describe album;
+-------------+---------------+------+-----+---------+----------------+
| Field       | Type          | Null | Key | Default | Extra          |
+-------------+---------------+------+-----+---------+----------------+
| id          | int(11)       | NO   | PRI | NULL    | auto_increment |
| title       | varchar(250)  | NO   |     | NULL    |                |
| description | varchar(2000) | YES  |     | NULL    |                |
+-------------+---------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)
```

包含在主键中的列不能有`NULL`值，因此无需在任何属于主键的列上指定禁止空值的约束。

`auto_increment`约束用于在`INSERT`语句未提供值时自动为列分配值。

现在我们有了一个包含单个表的数据库，以及一个可用于从 PHP 脚本连接和操作数据库的用户。在过程化设置中，这可能类似于`16_1.php`示例。

```php
<?php
// 16_1.php
$con = mysqli_connect('127.0.0.1', 'php', 'secret', 'music');
if (mysqli_connect_error()) {
    die("Error connecting: " . mysqli_connect_errno() . ' '
        . mysqli_connect_error() . "\n");
}
else {
    echo "Connected to music\n";
}
```

作为一种替代方案，相同的功能可以通过面向对象的方法实现。

```php
<?php
// 16_2.php
$mysqli = new mysqli('127.0.0.1', 'php', 'secret', 'music');
if ($mysqli->connect_error) {
    die("Error connecting: {$mysqli->connect_errno} {$mysqli->connect_error}\n");
}
else {
    echo "Connected to music\n";
}
```

第 16 章 ■ 使用 MySQL 数据库

如果脚本作为来自 Web 服务器的请求调用，则无需断开连接或释放任何资源。当脚本终止时，PHP 会自动处理这些。如果脚本运行时间较长或不再需要数据库连接，最好关闭连接。这可以通过`mysqli_close()`函数或`mysqli::close()`方法完成。

```php
<?php
// 16_3.php
$con = mysqli_connect('127.0.0.1', 'php', 'secret', 'music');
if (mysqli_connect_error()) {
    die("Error connecting: " . mysqli_connect_errno() . ' '
        . mysqli_connect_error() . "\n");
}
else {
    echo "Connected to music\n";
    mysqi_close($con);
}
```

或者使用面向对象的代码：

```php
<?php
// 16_4.php
$mysqli = new mysqli('127.0.0.1', 'php', 'secret', 'music');
if ($mysqli->connect_error) {
    die("Error connecting: {$mysqli->connect_errno} {$mysqli->connect_error}\n");
}
else {
    echo "Connected to music\n";
    $mysqli->close();
}
```

配方 16-2。持久连接

问题



