# 排版后的 Markdown 文档

尽管长期以来，各种命令行应用都能执行本节演示的网络任务，但通过 Web 来执行这些任务显然也有其便利之处。例如，我们在公司内网部署了一系列基于 Web 的应用，供 IT 支持部门的员工在排查网络问题时使用，即便他们手头没有 SSH 客户端。此外，现代无线 PDA 上的 Web 浏览器也能访问这些应用。最后，虽然命令行工具的功能更加强大和灵活，但通过 Web 查看这些信息有时反而更加方便。无论出于何种原因，你很可能都会用到本节中的某些应用。

[www.it-ebooks.info](http://www.it-ebooks.info/)

---

**394**

## 第 16 章 ■ 网络功能

> **注意** 本节中的几个示例使用了 `system()` 函数。该函数已在第 10 章介绍过。

#### Ping 服务器

验证服务器的连通性是一项常见的系统管理任务。以下示例展示了如何使用 PHP 实现：

```php
<?php

// 要 ping 哪台服务器？
$server = "www.example.com";

// 要 ping 服务器几次？
$count = 3;

// 执行任务
echo "<pre>";
system("/bin/ping -c $count $server");
echo "</pre>";

// 终止任务
system("killall -q ping");

?>
```

上述代码应该相当直观，唯一可能需要解释的是对 `killall` 的系统调用。这是必要的，因为如果用户提前终止了进程，由 `system` 调用执行的命令仍会继续运行。由于在浏览器中终止脚本执行并不能实际停止服务器上的进程，因此你需要手动终止。

示例输出如下：

```
PING www.example.com (192.0.34.166) from 123.456.7.8 : 56(84) bytes of data.
64 bytes from www.example.com (192.0.34.166): icmp_seq=0 ttl=255 time=158 usec
64 bytes from www.example.com (192.0.34.166): icmp_seq=1 ttl=255 time=57 usec
64 bytes from www.example.com (192.0.34.166): icmp_seq=2 ttl=255 time=58 usec

--- www.example.com ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/mdev = 0.048/0.078/0.158/0.041 ms
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

---

**395**

PHP 的程序执行函数非常强大，因为它们让你能够利用服务器上安装的任何程序。在本节后续内容中，我们将多次回到这些函数。

#### 端口扫描器

本章前面在介绍 `fsockopen()` 时，曾演示了如何创建一个端口扫描器。然而，正如本节介绍的许多任务一样，使用 PHP 的程序执行函数可以更轻松地实现。下面的示例使用了 PHP 的 `system()` 函数和 Nmap（网络映射器）工具：

```php
<?php

$target = "www.example.com";

echo "<pre>";
system("/usr/bin/nmap $target");
echo "</pre>";

// 终止任务
system("killall -q nmap");

?>
```

示例输出片段如下：

```
Starting nmap V. 2.54BETA31 ( www.insecure.org/nmap/ )
Interesting ports on (209.51.142.155):
(The 1500 ports scanned but not shown below are in state: closed)
Port       State       Service
22/tcp     open        ssh
80/tcp     open        http
110/tcp    open        pop-3
111/tcp    filtered    sunrpc
```

#### 子网转换器

你可能曾经挠头苦思，试图解决某个晦涩的网络配置问题。最常见的情况是，罪魁祸首往往是网线故障或未插好。而第二常见的问题，大概就是计算必要的基本网络要素（IP 地址、子网掩码、广播地址、网络地址等）时出错。为解决这个问题，可以利用几个 PHP 函数和位运算来帮你进行计算。清单 16-2 中的示例演示了如何在给定 IP 地址和位掩码后，计算出这些组件。



[www.it-ebooks.info](http://www.it-ebooks.info/)

**396**

