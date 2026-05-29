# 使用 SSL 保障敏感页面安全

显然，并非站点所有区域都需要 SSL 连接，也不应在所有地方强制使用 SSL，因为这会降低性能。但必须确保敏感页面仅能通过 SSL 访问。现在，你应该为管理员登录页面和管理员页面强制启用 SSL。（在后续章节中，当我们自行处理支付时，还需要为结账、客户登录、客户注册和客户信息修改页面强制启用 SSL。）

如果要确保对管理脚本`admin.php`的所有请求都通过 HTTPS 完成，只需在`admin.php`的开头添加以下代码：

```php
<?php

// Load Smarty library and config files

require_once 'include/app_top.php';

// Enforce page to be accessed through HTTPS
if (USE_SSL != 'no' and getenv('HTTPS') != 'on')
{
    header ('Location: https://' . getenv('SERVER_NAME') .
    getenv('REQUEST_URI'));
    exit();
}
```

注意，如果`config.php`中定义的`USE_SSL`常量设置为`no`，则不会强制使用安全连接。在开发网站时，若无法访问支持 SSL 的服务器，将常量设置为`no`可能很有用。