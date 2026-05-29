# PHP 解决方案 15-3：添加图像外键（PDO）

此 PHP 解决方案使用 PDO，通过将所选图像的主键作为外键添加到 `blog` 表中来更新记录。与 MySQLi 的主要区别在于，PDO 可以使用 `bindValue()` 方法将 `null` 值绑定到占位符。以下说明改编自第 13 章的 `admin/blog_update_pdo.php`。请使用您在第 13 章中创建的版本。或者，将 `ch13` 文件夹中的 `blog_update_pdo_03.php` 复制到 `admin` 文件夹，并从文件名中移除 `_03`。

将 `image_id` 添加到用于检索待更新文章详情的 `SELECT` 查询中，并将结果绑定到 `$image_id`。这涉及到重新编号传递给 `bindColumn()` 作为第一个参数的列，这些列对应 `$title` 和 `$article`。修改后的代码如下所示：

```php
if (isset($_GET['article_id']) && !$_POST) {

// 准备 SQL 查询

$sql = 'SELECT article_id, image_id, title, article FROM blog

WHERE article_id = ?';

$stmt = $conn->prepare($sql);

// 将占位符值作为单元素数组传递给 execute()

$OK = $stmt->execute([$_GET['article_id']]);

// 绑定结果

$stmt->bindColumn(1, $article_id);

$stmt->bindColumn(2, $image_id);

$stmt->bindColumn(3, $title);

$stmt->bindColumn(4, $article);

$stmt->fetch();

}
```

在表单内部，你需要显示存储在 `images` 表中的文件名。由于第二个 `SELECT` 语句不依赖于外部数据，因此使用 `query()` 方法比预处理语句更简单。在 `article` 文本区域之后添加以下代码（虽然这些都是新代码，但为方便参考，PHP 部分已用粗体突出显示）：

```php
<p>

<label for="image_id">已上传的图像：</label>

<select name="image_id" id="image_id">

<option value="">选择图像</option>

<?php

// 获取图像列表

$getImages = 'SELECT image_id, filename

FROM images ORDER BY filename';

foreach ($conn->query($getImages) as $row) {

?>

<option value="<?= $row['image_id']; ?>"

<?php

if ($row['image_id'] == $image_id) {

echo 'selected';

}

?> > <?= $row['filename']; ?></option>

<?php } ?>

</select>

</p>
```

第一个 `<option>` 标签是硬编码的，标签为 `选择图像`，其 `value` 设置为空字符串。其余的 `<option>` 标签由 `foreach` 循环填充，该循环执行 `$getImages SELECT` 查询并将每条记录提取到一个名为 `$row` 的数组中。

一个条件语句检查当前的 `image_id` 是否与 `articles` 表中已存储的相同。如果相同，则在 `<option>` 标签中插入 `selected`，以便在下拉菜单中显示正确的值。

确保不要遗漏以下代码行中的第三个字符：

```
?> > <?= $row['filename']; ?></option>
```

这是 `<option>` 标签的右尖括号，夹在两个 PHP 标签之间。

保存页面并将其加载到浏览器中。您应该会自动重定向到 `blog_list_pdo.php`。选择其中一个 `EDIT` 链接，并确保您的页面看起来像图 15-4。检查浏览器源代码视图以验证 `<option>` 标签的 `value` 属性是否包含每张图片的主键。

最后一步是将 `image_id` 添加到 `UPDATE` 查询中。当博客条目未关联图像时，您需要在 `image_id` 列中输入 `null`。这涉及到更改将值绑定到预处理语句中匿名占位符的方式。您需要使用 `bindValue()` 和 `bindParam()`，而不是将它们作为数组传递给 `execute()` 方法。修改后的代码如下所示：

```php
// 如果表单已提交，则更新记录

if (isset($_POST['update'])) {

// 准备更新查询

$sql = 'UPDATE blog SET image_id = ?, title = ?, article = ?

WHERE article_id = ?';

$stmt = $conn->prepare($sql);

if (empty($_POST['image_id'])) {

$stmt->bindValue(1, NULL, PDO::PARAM_NULL);

} else {

$stmt->bindParam(1, $_POST['image_id'], PDO::PARAM_INT);

}

$stmt->bindParam(2, $_POST['title'], PDO::PARAM_STR);

$stmt->bindParam(3, $_POST['article'], PDO::PARAM_STR);

$stmt->bindParam(4, $_POST['article_id'], PDO::PARAM_INT);

// 执行查询

$done = $stmt->execute();

}
```

值通过数字（从 1 开始计数）绑定到匿名占位符，以标识应应用于哪个占位符。一个条件语句检查 `$_POST['image_id']` 是否为空。如果为空，`bindValue()` 将值设置为 `null`，使用关键字 `NULL` 作为第二个参数，并使用 PDO 常量作为第三个参数。如第 11 章中 “在 PDO 预处理语句中嵌入变量” 所述，当绑定的值不是变量时，需要使用 `bindValue()`。

其余值都是变量，因此使用 `bindParam()` 绑定。对于这些值，我使用了整数和字符串的 PDO 常量。这并非严格必要，但能使代码更清晰。

最后，`execute()` 方法括号内的值数组已被移除。

再次测试页面，从下拉菜单中选择一个文件名，然后点击 `更新条目`。您可以通过在 phpMyAdmin 中刷新 `浏览` 或选择同一篇文章进行更新来验证外键是否已插入到 `articles` 表中。这次，下拉菜单中应显示正确的文件名。

如有必要，请对照 `ch15` 文件夹中的 `blog_update_pdo_04.php` 检查您的代码。

## 从多个表中选取记录

在 `SELECT` 查询中有几种链接表的方法，但最常见的是列出表名，并用 `INNER JOIN` 分隔。`INNER JOIN` 本身会产生行的所有可能组合（笛卡尔连接）。要仅选择相关值，你需要指定主键 / 外键关系。例如，要从 `blog` 和 `images` 表中选择文章及其相关图像，你可以使用 `WHERE` 子句，如下所示：

```sql
SELECT title, article, filename, caption

FROM blog INNER JOIN images

WHERE blog.image_id = images.image_id
```

`title` 和 `article` 列仅存在于 `blog` 表中。同样，`filename` 和 `caption` 仅存在于 `images` 表中。它们是无歧义的，不需要限定。但是，`image_id` 同时存在于两个表中，因此你需要为每个引用加上表名和一个句点作为前缀。

多年来，常见做法是使用逗号代替 `INNER JOIN`，如下所示：

```sql
SELECT title, article, filename, caption

FROM blog, images

WHERE blog.image_id = images.image_id
```

> **注意：** 使用逗号连接表可能导致 SQL 语法错误，因为自 MySQL 5.0.12 以来连接的处理方式发生了变化。请改用 `INNER JOIN`。

你可以使用 `ON` 来代替 `WHERE` 子句，如下所示：

```sql
SELECT title, article, filename, caption

FROM blog INNER JOIN images ON blog.image_id = images.image_id
```

当两列名称相同时，你可以使用以下语法，这也是我个人偏好的方式：

```
SELECT title, article, filename, caption

FROM blog INNER JOIN images USING (image_id)
```

> **注意：** `USING` 后面的列名必须放在括号中。