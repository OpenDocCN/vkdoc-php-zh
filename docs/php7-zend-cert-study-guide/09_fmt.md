# 九、Web 功能

PHP 是一种为网络而创造的语言。它最初的目的是让制作网页变得更容易，现在它仍然非常关注服务器端脚本。本章着眼于使它成为世界上最流行的服务器端 web 编程语言之一的一些语言特性。

## 请求类型

HTTP 有几种不同的请求方法 <sup>1</sup> ，它们通常被称为 HTTP 动词。HTTP 规范 <sup>2</sup> 相当详细地列出了每个动词的用途。您的应用应该遵守这个规范，以便与使用它的客户端兼容。

| 动词 | 习惯 |
| --- | --- |
| `GET` | 检索指定资源的表示形式 |
| `HEAD` | 与`GET`相同，但没有任何响应体 |
| `POST` | 向服务器提交一个条目，通常会导致诸如添加新资源之类的更改 |
| `PUT` | 用请求负载中的资源替换指定的资源 |
| `PATCH` | 对指定的资源应用部分修改 |
| `DELETE` | 删除指定的资源 |
| `CONNECT` | 发起一个 HTTP 隧道 <sup>3</sup> |
| 选择 | 描述目标资源的通信选项 |
| `TRACE` | 对目标资源执行消息环回测试 |

## 请求日期

在典型的生产 web 环境中，PHP 接受 web 服务器传递给它的请求。它运行并处理请求，然后终止并等待下一个请求。web 服务器可以随请求传递数据，这些数据构成了 PHP 运行的上下文 <sup>4</sup> 的一部分。

HTTP 请求由三部分组成:URL、头部和主体。数据可以包含在请求的任何部分中，并且可以用于 PHP 应用，如下所示:

| 来源 | 进来了 | 可用于 |
| --- | --- | --- |
| `GET` | 请求 URL 中的参数 | `$_GET` |
| `POST` | 请求的正文 | `$_POST` |
| `PUT` | 请求的正文 | 用`php://input`阅读 |
| `PATCH` | 请求的正文 | 用`php://input`阅读 |
| 饼干 | “cookie”标题 | `$_COOKIE` |
| 上传的文件 | 请求的正文 | `$_FILES` |

如果 PHP 正在从命令行处理一个请求，那么`$_SERVER['argv']`包含一个传递的参数数组，而`$_SERVER['argc']`包含传递的参数个数。

除了 HTTP 请求中包含的数据之外，PHP 还可以接受来自其运行环境的数据。例如，您可以在 docker 容器中运行 PHP，并设置一个包含数据库服务器地址的环境变量。您可以使用`$_ENV`超全局变量在 PHP 脚本中引用它。 <sup>5</sup>

### 请求超全局

`$_REQUEST`超全局是一个关联数组，默认包含`$_GET`、`$_POST`和`$_COOKIE`的内容。

`php.ini`设置`variables_order`决定了`ENV`、`GET`、`POST`和`COOKIE`变量中的哪一个出现在`$_REQUEST`数组中以及顺序。 <sup>6</sup>

如果同一变量存在于多个请求类型中，它将采用该设置值序列中最后一个的值。

例如，假设配置被设置为`EGPCS`，表示`POST`在`GET`之后。那么如果`$_GET['action']`和`$_POST['action']`都被设置，那么`$_REQUEST['action']`将包含`$_POST['action']`的值。

因为您不能确定`$_REQUEST`中的数据到底来自哪里，所以您应该小心使用这个数组。在代码中引入不确定性会使测试变得复杂，并可能影响安全性。

### 邮政

按照惯例，`POST`请求用于向网站发送数据，并指示它创建一个新的实体。在 CRUD 范例中，这是一个写操作。

#### 接收过帐数据

在`POST`请求中发送的变量包含在主体中。与在 URL 中传递变量的`GET`请求形成对比。

如果用户提交一个表单，那么浏览器会将这些值编码到请求的正文中并发送给你。类似地，指向 API 端点的应用需要将变量编码到请求体中。PHP 将在`$_POST`变量中提供它们。

例如，下面是一个将`POST`作为 name 变量的值发送给网站的例子。这个请求将被用来添加一个叫做`Ron`的人到粉丝俱乐部。

```php
POST  HTTP/1.1
Host: bieberfanclub.com
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

name=Ronald
```

如果运行`bieberfanclub.com`的应用正在运行 PHP，那么`$_POST`数组将是一个数组，包含一个名为`name`的元素，其值为`Ronald`。

用`POST`发送变量有三个好处:

*   `POST`数据可以用特定的字符集编码，而`GET`则不是这种情况。

*   因为变量是在消息体中发送的，所以可以发送的数据量不受 URL 长度的限制。

*   `POST`允许您上传文件，但`GET`不允许。

这两种方法在安全性上没有区别。

HTTP 协议中对 URL 的长度没有限制，但是对浏览器和其他客户端有限制。一般来说，不要创建超过 2000 个字符的 URL。

#### 发送帖子数据

当您想向另一个应用发出一个`POST`请求时，您需要负责将变量编码到主体中。最简单的方法是使用`curl`扩展。 <sup>7</sup> Curl 支持多种协议，可以轻松地按照您的需要设置您的请求。

使用`curl`包括以下过程:

1.  初始化一个`curl`会话。

2.  为会话设置选项。

3.  执行会话(打电话)。

4.  关闭会话并释放资源。

让我们看看如何使用`curl`来设置你之前看到的请求，其中你`POST`将包含值`Ronald`的变量`name`添加到比伯粉丝俱乐部。

```php
<?php
// We specify the url when we initialize the curl resource
$curlResource = curl_init("https://requestb.in/13fkcqj1");
// This array contains the variables we want to POST
$postData = ['name' => 'Ron'];
// Tell curl to do a application/x-www-form-urlencoded POST
curl_setopt($curlResource, CURLOPT_POST, true);
// We specify the values to POST
curl_setopt($curlResource,CURLOPT_POSTFIELDS, $postData);
// Execute the request and store the response
$response = curl_exec($curlResource);
// If there is an error it will be stored in $err
$err = curl_error($curlResource);
// Close the handle
curl_close($curlResource);
```

如果您运行这段代码，您将能够在 [`https://requestb.in/13fkcqj1?inspect`](https://requestb.in/13fkcqj1?inspect) 看到结果。

Tip

可以传递一个你想要设置的所有选项的数组，而不是多次调用它。

### 得到

请求通常用于从服务器获取单个实体或一组实体。你可以把它想象成从服务器读取数据。

#### 接收获取数据

在`GET`请求中发送的变量被编码到 URL 中。以下是如何将变量编码到 URL 中的示例:

[`http://bieberfanclub.com/topfan.php?name=Ron&rank=cheerleader`](http://bieberfanclub.com/topfan.php?name=Ron&rank=cheerleader)

变量以一个问号开始，并用和符号分隔。每个变量都是一个键值对，等号表示值。

PHP 会自动使 URL 中传递的变量在`$_GET` superglobal 中可用。

可以使用如下语法通过`GET`传递数组:

[`http://example.com/users.php?sort[col]=name&sort[order]=desc`](http://example.com/users.php?sort%5bcol%5d=name&sort%5border%5d=desc)

您可以像这样访问这些变量:

```php
<?php
echo $_GET['sort']['col'];
echo $_GET['sort']['order'];
```

#### 发送获取数据

PHP 包含了一个函数，可以非常容易地构建 URL 字符串来传递您的`GET`数据。

The request was rejected because it was considered high risk

| `tmp_name` | 文件在其临时位置的名称 |
| `error` | 错误代码，如果上传成功，则为`UPLOAD_ERR_OK` |

> **Note**
> 第六章有更多关于处理文件上传的信息。

## 形式

表单允许用户向 PHP 脚本提交数据。

当用 HTML 声明一个表单时，需要指定它用来向服务器发送信息的方法。虽然您可以选择`GET`或`POST`，但是您应该确保您选择的请求方法与您想要做的相匹配。

PHP 自动使表单数据在两个超级全局变量中的一个中对您的脚本可用，这两个超级全局变量是`$_GET`或`$_POST`，这取决于表单用来发出请求的方法。

### 表单元素

这些超全局变量很容易被客户端编辑，所以应该小心过滤，不要相信它们。

表单域名中的点和空格被转换为下划线。例如，考虑 HTML 输入标签:

```php
<input name="email.address" type="text">
```

它包含的值将根据 forms 方法放在`$_GET['email_address']`或`$_POST['email_address']`中。

### HTML 表单中的数组

可以使用 HTML 中的语法将表单数据转换为数组:

```php
<form action="formhandler.php" method="POST">
<input type="text" name="name[first]">
        <input type="text" name="name[last]">
        <input type="submit">
</form>
```

这将导致`$_POST`或`$_GET`成为如下所示的数组:

```php
array(
  'name' => array(
    'first' => '',
    'last' => ''
  )
)
```

数组帮助的最有用的方式之一是将输入分组在一起。

考虑一个可以有多个值的复选框:

```php
<h1>What pets do you want in your home?</h1>
<form action="formhandler.php" method="POST">
<input type="checkbox" name="pets[]" value="cats" id="lotsacats">
<label for="lotsacats">Lots of Cats</label>
<input type="checkbox" name="pets[]" value="dog" id="adog">
<label for="adog">Just a dog</label>
<input type="submit">
</form>
```

如果该人在提交表单前勾选了两个框，那么`$_GET`或`$_POST`数组将包含以下内容:

```php
array(
  'pets' => array('cats', 'dog')
)
```

这使得复选框更加整洁和易于使用。你可以在 PHP 手册中了解更多。 <sup>9</sup>

### 从列表中选择多个项目

最后，如果您希望用户能够从一个`select`列表中选择多个项目，您将需要使用一个数组:

```php
<select name="var[]" multiple="yes">
```

注意，`select`的名称是一个数组，所以用户选择的每个值都将被添加到超级全局数组中的`"var"`数组中。

## 饼干

Cookies 允许您在客户端设备上存储少量(4 到 6 KB)数据。客户端将读取它们，并在每次请求时发送它们。

您可以在 cookie 中存储任何类型的信息，但是它们通常与会话相关联。PHP 可以将其会话标识符存储在 cookie 中。会话信息存储在服务器上，并通过 cookie 中的标识符与客户端匹配。当您开始会话时，PHP 会为您做这件事。默认情况下，PHP 会话 cookies 在用户关闭浏览器之前一直有效。

您无法控制客户端设备上的 cookies。客户可以随时编辑或删除它们。这意味着你既不应该相信与它们一起发送的信息，也不应该依赖它们的存在。您也不应该将敏感信息存储在 cookies 中。

如果您想删除 cookie，您可以设置一个过去的到期日期。这将让客户端知道不再需要 cookie，可以将其删除。你不能保证客户会尊重这一点。

服务器将使用`Set-Cookie`响应头设置一个 cookie。客户端将使用`Cookie`请求头将其包含在未来的请求中。

### 设置 Cookies

`setcookie()`函数用于设置一个 cookie。PHP 手册中解释了这些参数，并按照下表的顺序给出:

| 参数 | 用于 |
| --- | --- |
| `value` | 在 cookie 中存储标量值。 |
| `expire` | cookie 过期时的 Unix 纪元时间戳。在 cookie 过期之前，你不能依赖它，因为人们删除他们的 cookie 是很常见的。 |
| `path` | cookie 将在其上可用的域的基本路径。如果设置为`/`，那么它在所有路径上都可用；否则，它将在该路径及其所有子路径上可用。 |
| `domain` | 该 cookie 将在此域及其下的所有子域中可用。您只能设置与提供 cookie 的域相匹配的 cookie。 |
| `secure` | 告诉客户端，如果 cookie 是通过 HTTPS 加密连接发送的，它应该只发送 cookie。 |
| `httponly` | 告诉客户端应该只使用 HTTP 发送 cookie，而不要让 JavaScript 等脚本语言使用它。在一定程度上，这有助于减少对支持 XSS 和会话固定的客户端的攻击。 |

Cookies 只能存储标量值。但是，您可以使用如下所示的语法:

```php
<?php
setcookie("user[name]", "Alice");
setcookie("user[email]", "alice@example.com");
```

下一次用户向站点发出请求时，`$_COOKIE`变量将包含如下内容:

```php
Array
 (
        [PHPSESSID] => jlm5od9ngqi3krmu6fkjcebcb4
        [user] => Array
     (
         [name] => Alice
         [email] => alice@example.com
     )

)
```

注意`user`是一个数组，cookie 值也包含 PHP 会话标识符。

### 正在检索 Cookies

您可以使用`$_COOKIE`超级全局变量来访问 cookie 信息。

记住，这个数组是用客户机发送的 cookie 中的信息填充的。这意味着，如果您使用`setcookie()`来创建或更改 cookie，那么当客户端发出新请求时，`$_COOKIE`数组将只包含新信息。

## HTTP 头

HTTP 头随客户端的请求和服务器的响应一起发送。它们用于传递关于 HTTP 消息的信息，比如提供了什么类型的信息，以及作为回报将接受什么。

HTTP 头采用明文字符串中的名称-值对的形式。每个标题后面都有一个回车符和换行符。标准中没有限制，但是大多数服务器和客户机对一个请求/响应中可以发送的报头长度和报头总数有限制。

PHP 将自动为您发出有效的头，但是在一些情况下，您可能想要发送自己的头。

### 发送邮件头

PHP 函数`header()`让你发送一个头到客户端。您只能在任何正常内容发送到客户端之前发送邮件头。在包含的 PHP 文件中省略结束标签`?>`的原因之一是为了避免在标签后出现换行符。该字符将作为 HTML 内容发送，并阻止您发送标题。

发送到`header()`的参数如下:

| 参数 | 描述 |
| --- | --- |
| 标题字符串 | 包含要设置的标头的字符串。比如`"Cache-Control: no-cache, must-revalidate"`。 |
| 替换 | 布尔值，指示此标头是否必须替换以前发送的同名标头。 |
| 响应代码 | 与标头一起发送的 HTTP 响应代码。 |

头有两种特殊情况。第一种是以字符串`"HTTP/"`开头的头。这些可以用来显式设置 HTTP 响应代码，如 PHP 手册 <sup>10</sup> 中的这个例子:

```php
<?php
header("HTTP/1.0 404 Not Found");
```

第二种特殊情况是使用`"Location"`标题。这个头向客户端表明他们正在寻找的文档在您指定的位置。

> **Note**
> 如果您使用这个头，PHP 将自动设置一个 302 HTTP 状态码，除非您已经设置了一个 2xx 或 3xx 头。

这里有一个例子:

```php
<?php
header("Location: http://www.example.com/");
exit;
```

在此示例中，服务器将使用状态代码 302 进行响应，客户端将被重定向到示例域。

注意发送重定向头后出口语言结构的用法。您的代码在发送标头后会继续运行，除非您停止它。这取决于客户端是否尊重您的重定向头。如果他们决定不尊重它，您的代码将继续输出，他们将看到它生成的任何输出。确保在发送重定向头后显式终止程序。

### 跟踪标题

`headers_list()`函数将返回一个准备发送或已经发送到客户端的头数组。您可以通过调用`headers_sent()`来确定报头是否已经发送。

如果您想阻止发送特定的标题，您可以使用`headers_remove()`功能从要发送的列表中取消设置标题。

## HTTP 认证

PHP 可以向客户端发送一个头，让它弹出一个需要认证的对话框。当用户在对话框中输入用户名和密码时，PHP 脚本的 URL 会被再次调用。

在第二次调用时，PHP 将在`$_SERVER`数组中有三个预定义的变量。它们是`PHP_AUTH_USER`、`PHP_AUTH_PW`和`AUTH_TYPE`，分别被设置为用户名、密码和认证类型。

然后，您应该使用您认为合适的任何方法对用户进行身份验证，比如对照数据库检查用户和密码。

PHP 手册页 <sup>11</sup> 中给出了 HTTP 认证的例子:

```php
<?php
 if (!isset($_SERVER['PHP_AUTH_USER'])) {
     header('WWW-Authenticate: Basic realm="My Realm"');
     header('HTTP/1.0 401 Unauthorized');
     echo 'Text to send if user hits Cancel button';
     exit;
 } else {
     echo "<p>Hello {$_SERVER['PHP_AUTH_USER']}.</p>";
     echo "<p>You entered {$_SERVER['PHP_AUTH_PW']} as your password.</p>";
}
```

在这个例子中，我们只是输出了`$_SERVER`数组中变量的内容，但是在现实生活中，我们会执行某种形式的认证。

客户端发送的密码是 base64 编码的，以标准化字符集，但没有执行哈希或加密。这是一种非常脆弱的保护网站的方式，除非你使用 HTTPS，否则在你的客户端和服务器之间的任何人都可以读取密码。

## HTTP 状态代码

HTTP 状态代码与响应一起发送，并遵循互联网工程任务组制定的标准以及行业内使用的实际标准。

HTTP 状态代码被分成 100 个代码的范围。该范围内的所有状态代码将具有类似的含义，如下表所示。

| 范围 | 一般含义 |
| --- | --- |
| `200` | 请求成功 |
| `300` | 请求需要被重定向 |
| `400` | 客户端有一个错误 |
| `500` | 服务器端有一个错误 |

你不需要为你的考试记住所有不同的状态代码。当你在现实生活中编写 API 时，你将能够访问维基百科 <sup>12</sup> 并为你的响应选择正确的代码。

对于你的考试，你只需要知道最重要的几个:

| 密码 | 状态 |
| --- | --- |
| `200` | OK；请求成功。 |
| `201` | 已创建；该请求导致创建新资源。 |
| `301` | 永久移动；资源将总是在指定的位置找到。 |
| `400` | 错误的请求；请求中的某些内容格式不正确或妨碍了它的执行。 |
| `401` | 未经授权；客户端没有被授权进行此请求。 |
| `403` | 禁止；(经过身份验证的)客户端不允许发出此请求。 |
| `418` | 我是茶壶；客户端试图向服务器发送咖啡制作协议，服务器实际上是一个茶壶。好吧，这不是一个重要的身份代码，但了解它很有趣。 |
| `500` | 内部服务器错误；服务器无法完成请求，无法做出更恰当的响应。通常与崩溃或错误配置相关联。 |

使用 API 时，HTTP 状态代码非常重要。如果你正在编写一个 API，你应该确保你为错误发送了正确的状态码。

例如，如果请求失败，您在正文中发送了一条错误消息，您应该确保 HTTP 状态代码是 400 而不是 200。

当您使用 PHP 时，您会对状态代码更加熟悉，但是如果有疑问，您应该查找一个列表，并确保您发送的是一个适当的响应。

## 输出控制功能

不是立即从脚本中输出，而是将输出存储在一个缓冲区中，然后立即刷新整个缓冲区，这通常非常有用。这对于在将输出发送给用户之前对输出进行转义，或者使用 PHP 内置的压缩例程来压缩发送给兼容浏览器的响应非常有用。

`ob_start()`功能用于启动输出缓冲。您的脚本通常会在响应正文中输出的任何内容都将被存储到缓冲区中。该函数采用一个可选参数，该参数是可调用的，当输出或丢弃缓冲区时将调用该参数。

下面是一个设置回调函数的例子。我将脚本的输出作为注释包含在内。

```php
<?php
function escapeOutput(string $buffer): string {
    return htmlentities($buffer);
}

ob_start("escapeOutput");

// &lt;script&gt;alert(&quot;xss&quot;);&lt;/script&gt;
echo '<script>alert("xss");</script>';
```

在本例中，当脚本结束时，缓冲区被隐式刷新。您可以使用`ob_flush()`函数 <sup>14</sup> 显式刷新缓冲区并输出其内容。当您正在编写 CLI 脚本并希望能够看到进度时，这可能是有用的，而另一个(可能是不好的)用例可能是为长轮询 JavaScript 客户机编写服务器。

```php
<?php
ob_start();
// this is a cli script
for ($i=1; $i<5; $i++) {
    echo $i;
    // each character is output one by one
    ob_flush();
    sleep(1);
}
```

`ob_flush`函数将输出缓冲区，并允许进一步缓冲输出。将其与`ob_end_flush,` <sup>15</sup> 进行比较，后者将输出缓冲并禁用任何进一步的缓冲。

Note

如果您的 web 服务器正在缓冲输出，并且您想尝试覆盖这种行为并直接向浏览器发送内容，那么可以使用`flush()`函数。然而，这并不总是有效的。

PHP 有一个内置的方法来帮助你在通过网络发送数据之前压缩数据。如果您启用它，那么 PHP 将检测浏览器是否能够支持压缩数据，如果是，使用 GZIP 算法来减少响应的大小。要设置它，您需要指定压缩函数作为对`ob_start()`的回调。更多例子请看 PHP 手册 <sup>16</sup> ，这里有一个简单的例子:

```php
<?php
ob_start("ob_gzhandler");
?>
{"string": "my json api output is compressed now"}
```

## Chapter 9 Quiz

Q1:假设`variables_order`被设置为 PHP 的默认值。对于下面的 HTTP 请求，`$_REQUEST['biggestfan']`的值是多少？

| `Ron` |
| `Ronald` |
| 别的东西 |
| 以上都不是 |

```php
POST  HTTP/1.1
Host: thebeebfanclub.com?biggestfan=Ron
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

biggestfan=Ronald
```

Q2:这些输入的每一个都有一个合适的构造形式，适合你的网站。将输入框中输入的名字放入一个变量的正确方法是什么，你可以像这样访问这个变量`$_POST['justin]['numberonefan']`？

| `<input type="text" name="justin.numberonefan">` |
| `<input type="text" name="justin(numberonefan)">` |
| `<input type="text" name="justin[numberonefan]">` |
| `<input type="text" name="justin_numberonefan">` |
| 以上都不是 |

Q3:cookie 是一种存储信息的可靠方式，不会浪费服务器资源。选择最正确的选项。

| 是的，在客户端存储信息可以节省服务器磁盘空间 |
| 不，信息的副本仍保留在会话数据中 |
| 不，他们不可靠 |
| 是的，将所有会话数据存储在 cookies 中是很常见的 |

Q4:这段代码会输出什么？选择所有适用的选项

| 一个通知，因为`$` a 未定义 |
| 警告，因为`$a`未定义 |
| 警告，因为您无法启动会话 |


| 包含`session_id`的随机字符串 |

```php
<?php
echo $a;
session_start();
echo session_id();
```

q5:cookie 的`domain`属性用来做以下哪一项？

| 限制 cookie 对网站的哪一部分有效 |

| 指定您的网站的名称 |

| 阻止浏览器将此 cookie 发送到其他网站 |

| 以上都不是 |

Footnotes 1

[`https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

2

[`https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9`](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9)

3

[`https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT)

4

[`en.wikipedia.org/wiki/Context_(computing)`](https://en.wikipedia.org/wiki/Context_(computing)

5

[`https://php.net/manual/en/reserved.variables.environment.php`](https://php.net/manual/en/reserved.variables.environment.php)

6

[`https://php.net/manual/en/ini.core.php#ini.variables-order`](https://php.net/manual/en/ini.core.php#ini.variables-order)

7

[`https://php.net/manual/en/book.curl.php`](https://php.net/manual/en/book.curl.php)

8

[`https://en.wikipedia.org/wiki/Percent-encoding`](https://en.wikipedia.org/wiki/Percent-encoding)

9

[`https://secure.php.net/manual/en/faq.html.php#faq.html.arrays`](https://secure.php.net/manual/en/faq.html.php#faq.html.arrays)

10

[`https://php.net/manual/en/function.header.php`](https://php.net/manual/en/function.header.php)

11

[`https://php.net/manual/en/features.http-auth.php`](https://php.net/manual/en/features.http-auth.php)

12

[`https://en.wikipedia.org/wiki/List_of_HTTP_status_codes`](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

13

[`https://en.wikipedia.org/wiki/Hyper_Text_Coffee_Pot_Control_Protocol`](https://en.wikipedia.org/wiki/Hyper_Text_Coffee_Pot_Control_Protocol)

14

[`https://php.net/manual/en/function.ob-flush.php`](https://php.net/manual/en/function.ob-flush.php)

15

[`https://php.net/manual/en/function.ob-end-flush.php`](https://php.net/manual/en/function.ob-end-flush.php)

16

[`https://secure.php.net/manual/en/function.ob-gzhandler.php`](https://secure.php.net/manual/en/function.ob-gzhandler.php)
