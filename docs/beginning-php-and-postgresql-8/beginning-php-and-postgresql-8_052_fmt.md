# 第 16 章 ■ 网络

**361**

- **SOA:** 起始授权记录。为主机设置全局参数。

- **SRV:** 服务记录。用于表示所提供域的各种服务的位置。

来看一个例子。假设你想验证域名 `example.com` 是否已被注册：

```php
<?php

$recordexists = checkdnsrr("example.com", "ANY"); if ($recordexists) echo "该域名已被注册，抱歉！"; else echo "该域名可用！";

?>
```

这将返回以下内容：

该域名已被注册，抱歉！

你可以使用此函数来验证指定邮件地址的域名是否存在：

```php
<?php

$email = "ceo@example.com";

$domain = explode("@",$email);

$valid = checkdnsrr($domain[1], "ANY");

if($valid) echo "该域名存在 MX 记录！";

else echo "找不到$domain[1]的 MX 记录！";

?>
```

这将返回：

该域名存在 MX 记录！

请注意，这并不是请求验证 MX 记录是否存在。有时网络管理员会采用其他配置方法来实现邮件解析，而不使用 MX 记录（因为 MX 记录并非强制要求）。为稳妥起见，只需检查域名是否存在，而无需专门验证 MX 记录是否存在。

##### `dns_get_record()`

`array dns_get_record (string *hostname* [, int *type* [, array *&authns*, array *&addtl*]])`

[www.it-ebooks.info](http://www.it-ebooks.info/)

**362** 第 16 章 ■ 网络

`dns_get_record()` 函数返回一个数组，其中包含与 `hostname` 指定的域名相关的各种 DNS 资源记录。虽然默认情况下 `dns_get_record()` 会返回它能找到的所有与指定域名相关的记录，但你可以通过指定 `type`（其名称必须以 `DNS_` 开头）来简化检索过程。该函数支持 `checkdnsrr()` 中介绍的所有类型，以及其他即将介绍的类型。最后，如果你需要该主机名的完整 DNS 描述，可以通过引用传递 `authns` 和 `addtl` 参数，指定与权威名称服务器相关的信息以及附加记录也应一并返回。

假设提供的 `hostname` 有效且存在，调用 `dns_get_record()` 至少会返回四个属性：

- **host:** 指定所有其他属性对应的 DNS 命名空间的名称。

- **class:** 由于此函数仅返回“Internet”类别的记录，因此该属性始终为 `IN`。

- **type:** 确定记录类型。根据返回的类型，可能还会提供其他属性。

- **ttl:** 记录的生存时间，计算方式为记录的原始 TTL 减去自查询权威名称服务器以来经过的时间。

除了 `checkdnsrr()` 部分介绍的类型外，以下域名记录类型也适用于 `dns_get_record()`：

- **DNS_ALL:** 检索所有可用的记录，即使某些记录在使用特定操作系统的识别功能时可能无法被识别。当你希望绝对确保所有可用记录都被检索到时使用此项。

- **DNS_ANY:** 检索特定操作系统能识别的所有记录。

- **DNS_HINFO:** 主机信息记录，用于指定主机的操作系统和计算机类型。请注意，这些信息并非必需。

- **DNS_NS:** 名称服务器记录，用于确定名称服务器是否为给定域名的权威应答服务器，或者此职责是否最终委派给其他服务器。

为避免重复，上述列表未包含 `checkdnsrr()` 中已介绍的类型。请记住，这些类型同样适用于 `dns_get_record()`。

只需记住，类型名称必须始终以 `DNS_` 开头。