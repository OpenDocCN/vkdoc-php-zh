# 第 15 章 ■ 搜索

最后一个新的模板构造是 yield 构造，如清单 15-12 所示。

***清单 15-12.*** `yield()` 处理程序

```php
public function yield($tree, $content)
{
    $key = trim($tree["raw"]);
    $value = addslashes($this->_getValue($key));
    return "\$_text[] = \"{$value}\";";
}
```

`yield()` 处理程序是所有新构造中最简单的，其功能与标准实现中的 `_literal()` 处理程序方法基本相同。

■ **注意** 记得调整 `View` 类以使用这个新的 `Template\Implementation\Extended` 类，否则你将无法使用新的构造。

### 搜索

完成上述工作后，添加搜索页面将变得容易得多。我们已经有登录页面，现在可以使用刚刚添加到扩展模板实现中的 partial 模板渲染。

目标很简单：我们应该为应用程序添加一致的导航，使登录用户可以访问注销/设置/个人资料，未登录用户可以访问登录/注册/搜索。我们首先在新视图文件中生成导航链接，如清单 15-13 所示。

***清单 15-13.*** `application/views/nagivation.html`

```html
<a href="/" > home</a>

{if (isset($user))}
<a href = "/users/profile.html" > profile</a>
<a href = "/users/settings.html" > settings</a>
<a href = "/users/logout.html" > logout</a>
{/if}
{else}
<a href = "/users/register.html" > register</a>
<a href = "/users/login.html" > login</a>
{/else}
```

逻辑很简单：如果模板有 `$user` 变量，则用户已登录，我们应该显示备用链接。你可能想知道这个变量在哪里设置，那么让我们回顾一下登录系统的工作原理。

用户输入他们的电子邮件地址和密码。这些字段然后提交给 `login()` 操作，该操作从数据库请求用户行。如果返回了一行，则该行将被序列化并存储在会话中。



