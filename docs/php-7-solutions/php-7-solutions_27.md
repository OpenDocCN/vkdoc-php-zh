# PHP 方案 6-5：添加邮件头并自动设置回复地址

本 PHP 方案为邮件添加了三个邮件头：`From`（发件人）、`Content-Type`（将编码设置为 UTF-8）和 `Reply-To`（回复地址）。在将用户邮箱地址添加到最终的邮件头之前，它使用了一个 PHP 内置过滤器来验证提交的值是否符合有效邮箱地址的格式。

继续使用之前相同的页面进行处理。或者，你也可以使用 `ch06` 文件夹中的 `contact_05.php` 和 `includes/processmail_02.php` 文件。

1. 邮件头通常针对特定的网站或页面，因此 `From` 和 `Content-Type` 邮件头将被添加到 `contact.php` 的脚本中。在页面顶部的 PHP 代码块中，就在引入 `processmail.php` 之前，添加以下代码：

```
    $required = ['name', 'comments', 'email'];
    // 创建额外的邮件头
    $headers[] = 'From: Japan Journey';
    $headers[] = 'Content-Type: text/plain; charset=utf-8';
    require './includes/processmail.php';
```

2. 验证邮箱地址的目的是确保其格式有效，但该字段可能为空，因为你可以决定不将其设为必填项，或者用户直接忽略了它。如果该字段为必填但为空，它将被添加到 `$missing` 数组中，并且你在 PHP 方案 6-2 中添加的警告信息将会显示。如果该字段不为空，但输入无效，则需要显示不同的提示信息。

切换到 `processmail.php`，并在脚本底部添加以下代码：

```
    // 验证用户的邮箱
    if (!$suspect && !empty($email)) {
    $validemail = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
    if ($validemail) {
    $headers[] = "Reply-To: $validemail";
    } else {
    $errors['email'] = true;
    }
    }
```

这段代码首先检查是否没有发现可疑内容（`$suspect` 为假），并且 `email` 字段不为空。这两个条件前面都使用了逻辑非运算符，因此当 `$suspect` 和 `empty($email)` 都为 `false` 时，它们返回 `true`。你在 PHP 方案 6-2 中添加的 `foreach` 循环会将 `$_POST` 数组中的所有预期元素赋值给更简单的变量，因此 `$email` 的值与 `$_POST['email']` 相同。

下一行使用 `filter_input()` 来验证邮箱地址。第一个参数是一个 PHP 常量 `INPUT_POST`，指定该值必须来自 `$_POST` 数组。第二个参数是你要测试的元素名称。最后一个参数是另一个 PHP 常量，指定你要检查该元素是否符合有效的邮箱格式。

如果被测试的值有效，`filter_input()` 函数会返回该值；否则返回 `false`。因此，如果用户提交的值看起来像是一个有效的邮箱地址，`$validemail` 会包含该地址；如果格式无效，`$validemail` 则为 `false`。`FILTER_VALIDATE_EMAIL` 常量只接受单个邮箱地址，因此任何尝试插入多个邮箱地址的行为都会被拒绝。

> **注意**
> `FILTER_VALIDATE_EMAIL` 仅检查格式，并不验证该地址是否真实存在。

如果 `$validemail` 不为 `false`，则可以安全地将其加入到 `Reply-To` 邮件头中。但如果 `$validemail` 为 `false`，则将 `$errors['email']` 添加到 `$errors` 数组中。

1. 现在，你需要修改 `contact.php` 中 `email` 字段的 `<label>` 标签，代码如下：

```
    Email:

Please enter your email address

Invalid email address

```

这为第一个条件语句添加了一个 `elseif` 子句，并在邮箱地址验证失败时显示不同的警告信息。

2. 保存 `contact.php`，通过留空所有字段并点击“发送消息”来测试表单。你会看到原来的错误信息。再次测试，在“Email”字段中输入一个不是邮箱地址的值，或者输入两个邮箱地址。你应该会看到无效邮箱的提示信息。

你可以使用 `ch06` 文件夹中的 `contact_06.php` 和 `includes/processmail_03.php` 来检查你的代码。

