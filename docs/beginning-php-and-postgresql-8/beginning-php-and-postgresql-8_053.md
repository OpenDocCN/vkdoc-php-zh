# 网络

## DNS 功能

##### `dns_get_record()`

来看一个示例。假设你想进一步了解 `example.com` 域名：

```php
<?php

$result = dns_get_record("example.com");

print_r($result);

?>
```

返回的抽样信息如下：

```
Array (
    [0] => Array (
        [host] => example.com
        [type] => NS
        [target] => a.iana-servers.net
        [class] => IN
        [ttl] => 110275
    )
    [1] => Array (
        [host] => example.com
        [type] => A
        [ip] => 192.0.34.166
        [class] => IN
        [ttl] => 88674
    )
)
```

如果你只对名称服务器记录感兴趣，可以执行以下代码：

```php
<?php

$result = dns_get_record("example.com","DNS_CNAME"); print_r($result);

?>
```

这会返回以下内容：

```
Array ( [0] => Array ( [host] => example.com [type] => NS
[target] => a.iana-servers.net [class] => IN [ttl] => 21564 )
[1] => Array ( [host] => example.com [type] => NS
[target] => b.iana-servers.net [class] => IN [ttl] => 21564 ) )
```

##### `getmxrr()`

```
int getmxrr (string *hostname*, array *&mxhosts* [, array *&weight*])
```

`getmxrr()` 函数检索由 `hostname` 指定的主机的 MX 记录。MX 记录会被添加到 `mxhosts` 指定的数组中。如果提供了可选输入参数 `weight`，相应的权重值将被放置在其中，这些权重值指向每个由记录标识的服务器所分配的命中率。示例如下：

```php
<?php

getmxrr("wjgilmore.com",$mxhosts);

print_r($mxhosts);

?>
```

这会返回以下内容：

```
Array ( [0] => mail.wjgilmore.com)
```

#### 服务

尽管我们通常泛化地使用“互联网”这个词，谈论着使用互联网聊天、阅读或下载某款游戏的最新版本，但实际上，我们指的是共同定义这个通信平台的一个或多个互联网服务。这些服务的例子包括 HTTP、FTP、POP3、IMAP 和 SSH。出于各种原因（其解释超出了本书范围），每个服务通常运行在一个特定的通信端口上。例如，HTTP 的默认端口是 80，SSH 的默认端口是 22。如今，网络各层级对防火墙的广泛需求使得了解这些事项变得相当重要。PHP 有两个函数，`getservbyname()` 和 `getservbyport()`，可用于了解更多关于服务及其对应端口号的信息。

##### `getservbyname()`

```
int getservbyname (string *service*, string *protocol*)
```

`getservbyname()` 函数返回与 `/etc/services` 文件中 `service` 对应的服务端口号。`protocol` 参数指定你引用的是该服务的 tcp 还是 udp 组件。来看一个示例：

```php
<?php

echo "HTTP 的默认端口号是：".getservbyname("http", "tcp");

?>
```

这会返回以下内容：

```
HTTP 的默认端口号是：80
```

##### `getservbyport()`

```
string getservbyport (int *port*, string *protocol*)
```

`getservbyport()` 函数返回与 `/etc/services` 文件中提供的端口号对应的服务名称。`protocol` 参数指定你引用的是该服务的 tcp 还是 udp 组件。来看一个示例：

```php
<?php

echo "端口 80 的默认服务是：".getservbyport(80, "tcp");

?>
```

这会返回以下内容：

```
端口 80 的默认服务是：http
```

### 建立套接字连接

在当今的网络环境中，你常常需要查询本地或远程的服务。这通常通过与该服务建立套接字连接来完成。本节将演示如何使用 `fsockopen()` 函数实现这一点。

#### `fsockopen()`

```
resource fsockopen (string *target*, int *port* [, int *errno* [, string *errstring*
[, float *timeout*]]])
```

`fsockopen()` 函数在端口 `port` 上与 `target` 指定的资源建立连接，并将错误信息返回给可选参数 `errno` 和 `errstring`。可选参数 `timeout` 设置一个时间限制（以秒为单位），指定函数在失败之前尝试建立连接的时间长度。

第一个示例展示了如何使用 `fsockopen()` 与 `www.example.com` 建立端口 80 连接，并输出其首页：

```php
<?php

// 与 www.example.com 建立端口 80 连接

$http = fsockopen("www.example.com",80);

// 向服务器发送请求

$req = "GET / HTTP/1.1\r\n";

$req .= "Host: www.example.com\r\n";

$req .= "Connection: Close\r\n\r\n";

fputs($http, $req);

// 输出请求结果

while(!feof($http))

{

echo fgets($http, 1024);

}

// 关闭连接

fclose($http);

?>
```

这会返回以下内容：

```
HTTP/1.1 200 OK Date: Mon, 05 Jan 2006 02:17:54 GMT Server: Apache/1.3.27 (Unix) (Red-Hat/Linux) Last-Modified: Wed, 08 Jan 2006 23:11:55 GMT ETag:
"3f80f-1b6-3e1cb03b" Accept-Ranges: bytes Content-Length: 438
Connection: close Content-Type: text/html

You have reached this web page by typing "example.com", "example.net", or
"example.org" into your web browser.

These domain names are reserved for use in documentation and are not available for registration. See RFC 2606, Section 3.
```

第二个示例，如清单 16-1 所示，演示了如何使用 `fsockopen()` 构建一个基本的端口扫描器。

**清单 16-1.** *使用 fsockopen() 创建端口扫描器*

```php
<?php

// 为脚本提供足够的时间完成此任务

ini_set("max_execution_time", 120);

// 定义扫描范围

$rangeStart = 0;

$rangeStop = 1024;

// 要扫描哪个服务器？

$target = "www.example.com";

// 构建端口值的数组

$range =range($rangeStart, $rangeStop);

echo "<p>$target 的扫描结果</p>";

// 执行扫描

foreach ($range as $port) {

$result = @fsockopen($target, $port,$errno,$errstr,1);

if ($result) echo "<p>端口 $port 处套接字已打开</p>";

}

?>
```

扫描 `www.example.com` 网站，返回以下输出：

```
www.example.com 的扫描结果：

端口 22 处套接字已打开

端口 80 处套接字已打开

端口 443 处套接字已打开
```

实现相同目标的一种更省事的方法是使用程序执行命令（如 `system()`）和出色的免费软件包 Nmap (http://www.insecure.org/nmap/)。本章的结束部分“常见网络任务”中演示了这种方法。

#### `pfsockopen()`

```
int pfsockopen (string *host*, int *port* [, int *errno* [, string *errstring*
[, int *timeout*]]])
```

`pfsockopen()` 函数，即“持久化 `fsockopen()`”，在操作上与 `fsockopen()` 相同，区别在于脚本执行完成后连接不会关闭。

### 邮件

PHP 这个强大且易于实现的功能非常有用，并且在众多 Web 应用程序中都是必需的，以至于本节很可能是本章（如果不是本书的话）中最受欢迎的部分之一。在本节中，你将学习如何使用 PHP 流行的 `mail()` 函数发送电子邮件，包括如何控制头部、包含附件以及执行其他常见的任务。此外，还将介绍 PHP 的 IMAP 扩展，并演示通过这个出色的库提供的众多功能。

本节将介绍相关的配置指令，描述 PHP 的 `mail()` 函数，并以几个突出展示该函数多种用法的示例作为结尾。

### 配置指令

有五个与 PHP 的 `mail()` 函数相关的配置指令。请密切关注这些描述，因为每条指令都是特定于平台的。

- `SMTP`
  - 作用域：`PHP_INI_ALL`
  - 默认值：`localhost`



