# PHP 与 LDAP

## `ldap_start_tls()`

`boolean ldap_start_tls (resource $link_id)`

尽管`ldap_start_tls()`并非严格意义上的连接特定函数，但在此节中介绍它，是因为如果开发者希望使用传输层安全（TLS）协议安全地连接到 LDAP 服务器，通常会在调用`ldap_connect()`后立即执行它。关于此函数有几点值得注意：

- 仅当使用 LDAPv3 时才能建立 LDAP 的 TLS 连接。由于 PHP 默认使用 LDAPv2，因此在调用`ldap_start_tls()`之前，需要先使用`ldap_set_option()`明确声明使用版本 3。更多信息请参阅后面的“配置函数”一节。

- 你可以在绑定到目录之前或之后调用`ldap_start_tls()`，但如果你关心保护绑定凭据，则在绑定之前调用更为合理。

示例如下：

```php
<?php

$ldapconn = ldap_connect("ldap://ad.example.com");

ldap_set_option($ldapconn, LDAP_OPT_PROTOCOL_VERSION, 3);

ldap_start_tls($ldapconn);

?>
```

由于`ldap_start_tls()`用于安全连接，新用户常错误地尝试使用`ldaps://`而非`ldap://`进行连接。请注意，从上述示例可知，使用`ldaps://`是不正确的，而应始终使用`ldap://`。

## 绑定到 LDAP 服务器

成功连接到 LDAP 服务器（参见`ldap_connect()`）后，需要提供一组凭据，后续所有 LDAP 查询将在这些凭据的授权下执行。这些凭据包括一个类似于用户名的标识，通常称为 RDN（相对可分辨名称，Relative Distinguished Name）以及一个密码。

### `ldap_bind()`

`boolean ldap_bind (resource $link_id [, string $bind_rdn [, string $bind_pswd]])`

虽然任何人都可以连接到 LDAP 服务器，但在检索或操作数据之前，通常需要有效的凭据。这一任务通过`ldap_bind()`完成。该函数至少需要来自`ldap_connect()`的`$link_id`，并且很可能还需要一个用户名和密码，分别由`$bind_rdn`和`$bind_pswd`表示。示例如下：

```php
<?php

$ldapHost = "ldap://ad.example.com";

$ldapPort = "389";

$ldapUser = "ldapreadonly";

$ldapPswd = "iloveldap";

$ldapconn = ldap_connect($ldapHost, $ldapPort)

    or die("Can't establish LDAP connection");

ldap_bind($ldapconn, $ldapUser, $ldapPswd)

    or die("Can't bind to the server.");

?>
```

请注意，提供给`ldap_bind()`的凭据是在 LDAP 服务器中创建和管理的，与您连接时所使用的服务器或工作站上的任何帐户无关。因此，如果您无法匿名连接到 LDAP 服务器，则需要与系统管理员沟通，以安排一个合适的帐户。

## 关闭 LDAP 服务器连接

完成与 LDAP 服务器的所有交互后，应进行清理并正确关闭连接。有一个函数`ldap_unbind()`可用于此目的。

### `ldap_unbind()`

`boolean ldap_unbind (resource $link_id)`

`ldap_unbind()`函数终止与`$link_id`关联的 LDAP 服务器连接。用法示例如下：

```php
<?php

$ldapUser = "ldapreadonly";

$ldapPswd = "iloveldap";

$ldapconn = ldap_connect("ldap://ad.example.com", 389)

    or die("Can't establish LDAP connection");

ldap_bind($ldapconn, "ldapreadonly", "iloveldap")

    or die("Can't bind to LDAP.");

/* 执行各种 LDAP 相关命令。 */

ldap_unbind($ldapconn)

    or die("Could not unbind from LDAP server.");

?>
```

> **注意** PHP 函数`ldap_close()`在操作上与`ldap_unbind()`相同，但由于 LDAP API 使用后者这一术语，出于可读性考虑，建议使用`ldap_unbind()`而非`ldap_close()`。