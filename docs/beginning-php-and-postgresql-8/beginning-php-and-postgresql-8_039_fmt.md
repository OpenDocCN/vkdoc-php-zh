# 高质量 GUI 设计与一致性

高质量的 GUI 设计很大程度上源于一致性。也就是说，始终追求整个网站的视觉和谐是一个好主意，特别是在用户会直接接触的那些组件中——例如表单。为了促进一致的界面，尽可能复用基于表单的代码可能是个好主意，在数据插入和修改时都使用相同的模板。当然，你可能认为这种策略会很快导致逻辑和呈现的混乱。然而，只要稍加预先思考，实际上很容易在保持良好编码实践的同时促进表单复用。本节将介绍一种实现方法。

上一节演示了如何创建一个通用函数来生成动态下拉列表。为了说明本节介绍的概念，我们继续这个主题，不过这次我们将修改`create_dropdown()`函数，使其既能生成动态列表，又能自动选择预定的值。添加这个额外功能只需定义另一个参数：

- `$key`：此可选参数保存要自动选择的元素的值。如果未赋值，则不会自动选择任何值。

该函数在构建下拉列表时，通过将每个元素与`$key`进行比较来确定是否应自动选择某个元素。为了代码更紧凑，这里使用了三元运算符进行比较。修改后的函数如下：

```php
function create_dropdown($identifier, $pairs, $firstentry,$multiple="", $key="")
{
    $dropdown = "<select name=\"$identifier\" multiple=\"$multiple\">";
    $dropdown .= "<option name=\"\">$firstentry</option>";
    foreach($pairs AS $value => $name)
    {
        $dropdown .= ($value == $key) ?
            "<option name=\"$value\" selected=\"selected\">$name</option>" :
            "<option name=\"$value\">$name</option>";
    }
    echo "</select>";
    return $dropdown;
}
```

如果你想自动选择"意大利语"元素，只需传入其对应的标识符，例如"2"，如下所示：

```php
echo create_dropdown("language",$pairs,"选择一项:", "", 2);
```

这将产生以下输出（为便于阅读已格式化）：

```html
选择你偏好的语言：<br />
<select name="language" >
    <option name="">选择一项：</option>
    <option name="4">荷兰语</option>
    <option name="1">英语</option>
    <option name="2" selected="selected">意大利语</option>
    <option name="3">西班牙语</option>
</select>
```

请注意，"意大利语"元素已被选中。

## PHP、Web 表单与 JavaScript

当然，使用 PHP 作为主要脚本语言并不意味着你应该依赖它来处理所有事情。事实上，将 PHP 与诸如 JavaScript 之类的客户端语言结合使用，往往能极大地扩展应用程序的灵活性。然而，一个常见的混淆点是如何让一种语言与另一种语言通信，因为 JavaScript 在客户端执行，而 PHP 在服务器端执行。实现这一目标比你想象的要简单，如下例所示。

许多网站提供将文章或新闻故事通过电子邮件发送给朋友的功能。有时这是通过使用"弹出"窗口实现的，该窗口会提示用户输入收件人地址和其他信息。提交表单后，文章会被邮寄给收件人，然后用户关闭窗口。通常，弹出窗口操作使用 JavaScript 完成，而邮件提交则使用 PHP 执行。然而，由于 JavaScript 正在启动新窗口，它必须能够传递一些重要信息，例如唯一标识文章的标识符。

以下脚本演示了这一任务，展示了将 PHP 变量传递给 JavaScript 函数是多么容易。在文档头部，定义了一个名为`mail()`的 JavaScript 函数。该函数会打开一个固定大小的新窗口，指向一个 PHP 脚本，该脚本随后会提示用户并处理邮件提交。

```html
<html>
<head>
<title>突发新闻</title>
<script type="text/javascript">
function mail(id) {
    window.open("mail.php?id=" + id, "info",
        "width=250,height=250,scrollbars=0,resizable=0")
}
</script>
</head>
<body bgcolor="#ffffff" text="#000000" link="#0000ff"
    vlink="#800080" alink="#ff0000">
<a href="#" onclick="mail(<?php echo $id; ?>);"> 将本文通过邮件发送给朋友</a>
文章内容在此...
</body>
</html>
```

一旦链接被点击，就会打开一个类似图 13-3 所示的表单。

**图 13-3.** *文章邮件发送表单*

特别要注意，你只需跳转到 PHP，输出变量，然后再跳转回 HTML，就可以将 PHP 变量`$id`传递给 JavaScript 函数`mail()`的调用。

点击链接会触发`onclick()`事件，该事件会打开以下脚本：

```php
<?php
// 如果邮件表单已提交
if (isset($_POST['submit']))
{
    // 指定邮件头和正文
    $headers = "FROM:editor@example.com\n";
    $body = $_POST['name']."认为你可能对这篇文章感兴趣：\nhttp://www.example.com/article.html?id=".$_POST['id'];
    // 发送文章 URL
    mail($_POST['recipient'],"Example.com 新闻文章",$body,$headers);
    // 通知用户
    echo "文章已邮寄到 ".$_POST['recipient'];
}
?>
```

```html
<p>
通过邮件将本文发送给朋友！
</p>
<form action="mail.html" method="post">
<input type="hidden" name="id" value="<?php echo $_GET['id'];?>" />
<p>
收件人邮箱：<br />
<input type="text" name="recipient" size="20" maxlength="40" value="" />
</p>
<p>
你的姓名：<br />
<input type="text" name="name" size="20" maxlength="40" value="" />
</p>
<input type="submit" name="submit" value="发送文章" />
</form>
```

尽管我们使用了一个预定义的 URL 为收件人提供文章参考，但你也可以轻松地通过使用可用的唯一标识符（`$id`）从数据库中检索文章，并将文章信息直接嵌入电子邮件。

### 导航提示

程序员倾向于将可用性相关的问题交给网站设计师处理。确实，虽然网站的呈现方面通常由设计师负责，但程序员在以便捷格式向设计师提供必要的导航数据方面扮演着非常重要的角色。但是，应用程序数据如何为用户提供有助于网站导航的提示呢？严格来说，Web 应用程序的"可用性"程度取决于其使用所带来的有效性和满意度。换句话说，界面设计是否让用户感到舒适，甚至在使用时感到得心应手？用户能否轻松找到所需的工具和数据？它是否提供了达到相同目的的多种途径，通常通过易于获得的视觉提示来实现？综合起来，这些特征定义了应用程序的"可用性"。

本节介绍三种常见的导航提示：用户友好的 URL、面包屑导航和自定义错误文件。这三种都可以用最少的精力实现，并为用户提供相当大的价值。

#### 用户友好的 URL

在 Web 早期，遇到这样的 URL 会让人印象深刻：`http://www.example.com/sports/football/buckeyes.html`

这位用户显然非常认真！毕竟，他花了心思对网站内容进行分类，而从其网址结构来看，他的网站规模庞大，涉及的内容不止一支足球队，甚至不止一项运动。不过，直观的网址结构为网站访客判断当前位置提供了额外帮助，更不用说那些高级用户还能通过直接操作网址来浏览网站了。

[www.it-ebooks.info](http://www.it-ebooks.info/)