# PHP 解决方案 16-4：向多张表插入数据

本 PHP 解决方案修改了 `blog_insert_mysqli.php` 中的现有脚本，根据图 16-6 所示的决策链，上传新图片（如有必要），然后将数据插入到 `images`、`blog` 和 `article2cat` 表中。此解决方案假设您已经设置了 `article2cat` 交叉引用表，并完成了 PHP 解决方案 16-2 和 16-3。

不要急于完成本部分内容。代码虽然较长，但它汇集了您之前学到的许多技术。

**注意：** 如果您使用 PDO，本 PHP 解决方案之后会有一个独立部分介绍代码的主要区别。

您在 PHP 解决方案 16-2 中更新的 `Upload` 类使用了命名空间，因此您需要在脚本顶层导入它。在 `blog_insert_mysqli.php` 顶部的 PHP 起始标签之后立即添加以下代码行：

`use PhpSolutions\File\Upload;`

在准备好的语句初始化之后，立即插入以下条件语句，用于处理已上传或已选择的图片。

```
// 初始化准备好的语句
$stmt = $conn->stmt_init();

// 如果已上传文件，则进行处理
if(isset($_POST['upload_new']) && $_FILES['image']['error'] == 0) {
    $imageOK = false;
    require_once '../PhpSolutions/File/Upload.php';
    $loader = new Upload('../images/');
    $loader->upload();
    $names = $loader->getFilenames();
    // 如果上传失败，$names 将是一个空数组
    if ($names) {
        $sql = 'INSERT INTO images (filename, caption) VALUES (?, ?)';
        if ($stmt->prepare($sql)) {
            $stmt->bind_param('ss', $names[0], $_POST['caption']);
            $stmt->execute();
            $imageOK = $stmt->affected_rows;
        }
    }
    // 获取图片的主键或找出问题所在
    if ($imageOK) {
        $image_id = $stmt->insert_id;
    } else {
        $imageError = implode(' ', $loader->getMessages());
    }
} elseif (isset($_POST['image_id']) && !empty($_POST['image_id'])) {
    // 获取之前上传图片的主键
    $image_id = $_POST['image_id'];
}

// 创建 SQL
$sql = 'INSERT INTO blog (title, article, created) VALUES(?, ?, NOW())';
```

这首先检查 `$_POST['upload_new']` 是否已设置。如第 5 章所述，复选框仅在被选中时才会包含在 `$_POST` 数组中。因此，如果复选框未被选中，条件不成立，则转而测试底部的 `elseif` 子句。`elseif` 子句检查 `$_POST['image_id']` 是否存在。如果存在且不为空，则表示已从下拉菜单中选择了一张现有图片，该值存储在 `$image_id` 中。

如果两个测试都失败，则既没有上传图片，也没有从下拉菜单中选择图片。脚本稍后在为 `blog` 表准备 `INSERT` 查询时会考虑这一点，允许您创建不带图片的博客条目。

但是，如果 `$_POST['upload_new']` 存在，则表示复选框已被选中，并且可能已上传了图片。为了确保这一点，条件语句还会检查 `$_FILES['image']['error']` 的值。如您在第 6 章中所学，错误代码 `0` 表示上传成功。任何其他错误代码都表示上传失败或未选择文件。

假设表单已成功上传文件，条件语句会包含 `Upload` 类定义，并创建一个名为 `$loader` 的对象，将目标文件夹设置为 `images`。然后调用 `upload()` 方法来处理文件并将其存储在 `images` 文件夹中。为了避免代码复杂化，我使用了默认的最大尺寸和 MIME 类型。

您在 PHP 解决方案 16-2 中对 `Upload` 类所做的修改，仅当文件成功移动到目标文件夹时，才会将上传文件的名称添加到 `$filenames` 属性中。`getFilenames()` 方法检索 `$filenames` 属性的内容，并将结果赋给 `$names`。

如果文件移动成功，其文件名将作为第一个元素存储在 `$names` 数组中。因此，如果 `$names` 包含一个值，您可以安全地继续执行 `INSERT` 查询，该查询将 `$names[0]` 和 `$_POST['caption']` 的值作为字符串绑定到准备好的语句中。

语句执行后，`affected_rows` 属性会重置 `$imageOK` 的值。如果 `INSERT` 查询成功，`$imageOK` 为 `1`，将被视为 `true`。

如果图片详情已插入到 `images` 表中，则准备好的语句的 `insert_id` 属性会检索新记录的主键，并将其存储在 `$image_id` 中。必须在运行任何其他 SQL 查询之前访问 `insert_id` 属性，因为它包含最新查询的主键。

但是，如果 `$imageOK` 仍然为 false，则 `else` 块会调用上传对象的 `getMessages()` 方法，并将结果赋给 `$imageError`。`getMessages()` 方法返回一个数组，因此使用 `implode()` 函数将数组元素连接成一个字符串。失败最可能的原因是文件太大或 MIME 类型错误。

只要图片上传没有失败，下一步就是将博客条目插入到 `blog` 表中。`INSERT` 查询的形式取决于博客条目是否关联了图片。如果有关联，则 `$image_id` 存在，需要作为外键插入到 `blog` 表中。否则，可以使用原始查询。

像这样修改原始查询：

```
// 如果图片上传失败，则不插入博客详情
if (!isset($imageError)) {
    // 如果已设置 $image_id，则将其作为外键插入
    if (isset($image_id)) {
        $sql = 'INSERT INTO blog (image_id, title, article) VALUES(?, ?, ?)';
        if ($stmt->prepare($sql)) {
            $stmt->bind_param('iss', $image_id, $_POST['title'], $_POST['article']);
            $stmt->execute();
        }
    } else {
        // 创建 SQL
        $sql = 'INSERT INTO blog (title, article)
                VALUES(?, ?)';
        if ($stmt->prepare($sql)) {
            // 绑定参数并执行语句
            $stmt->bind_param('ss', $_POST['title'], $_POST['article']);
            $stmt->execute();
        }
    }

    if ($stmt->affected_rows > 0) {
        $OK = true;
    }
}
```

整个代码块都包裹在一个条件语句中，检查 `$imageError` 是否存在。如果存在，则无需插入新博客条目，因此会忽略整个代码块。

但是，如果 `$imageError` 不存在，则嵌套的条件语句会根据 `$image_id` 是否存在准备不同的 `INSERT` 查询，然后执行已准备好的那一个。

检查 `affected_rows` 属性的条件语句被移到了 `else` 块之外，以便它适用于任何一种 `INSERT` 查询。

流程的下一步是向 `article2cat` 交叉引用表中插入值。代码紧跟在之前步骤的代码之后，如下所示：

```
// 如果博客条目插入成功，检查分类
if ($OK && isset($_POST['category'])) {
    // 获取文章的主键
    $article_id = $stmt->insert_id;

    foreach ($_POST['category'] as $cat_id) {
        if (is_numeric($cat_id)) {
            $values[] = "($article_id, " . (int) $cat_id . ')';
        }
    }

    if ($values) {
        $sql = 'INSERT INTO article2cat (article_id, cat_id)
                VALUES ' . implode(',', $values);
        // 执行查询，如果失败则获取错误消息
        if (!$conn->query($sql)) {
            $catError = $conn->error;
        }
    }
}
```



`$OK`的值由插入`blog`表数据的查询中的`affected_rows`属性决定，且多选的`<select>`列表仅在选中任何分类时才包含在`$_POST`数组中。因此，仅当数据成功插入`blog`表且表单中至少选中一个分类时，才会执行此代码块。它首先从预处理语句的`insert_id`属性获取插入操作的主键，并将其赋值给`$article_id`。

表单将分类值作为数组提交。`foreach`循环检查`$_POST['category']`中的每个值。如果值为数字，则执行以下行：

```
$values[] = "($article_id, " . (int) $cat_id . ')';
```

这将创建一个包含两个主键`$article_id`和`$cat_id`的字符串，用逗号分隔并包裹在括号内。`(int)`类型转换运算符确保`$cat_id`为整数。结果赋值给名为`$values`的数组。例如，如果`$article_id`为`10`且`$cat_id`为`4`，则赋值给数组的结果字符串为`(10, 4)`。

如果`$values`包含任何元素，`implode()`会将其转换为逗号分隔的字符串并附加到 SQL 查询中。例如，如果选中了分类`2`、`4`和`5`，则生成的查询如下所示：

```
INSERT INTO article2cat (article_id, cat_id)
VALUES (10, 2),(10, 4),(10,5)
```

正如“回顾四个基本的 SQL 命令”中第 13 章所述，这是如何使用单个`INSERT`查询插入多行数据的方法。

由于`$article_id`来自可靠源且`$cat_id`的数据类型已检查，因此直接在 SQL 查询中使用这些变量而无需使用预处理语句是安全的。该查询通过`query()`方法执行。如果失败，连接对象的`error`属性将存储在`$catError`中。

代码的最后部分处理成功时的重定向和错误消息。修改后的代码如下所示：

```
// redirect if successful or display error
if ($OK && !isset($imageError) && !isset($catError)) {
    header('Location: http://localhost/phpsols/admin/blog_list_mysqli.php');
    exit;
} else {
    $error = $stmt->error;
    if (isset($imageError)) {
        $error .= ' ' . $imageError;
    }
    if (isset($catError)) {
        $error .= ' ' . $catError;
    }
}
```

控制重定向的条件现在确保`$imageError`和`$catError`不存在。如果存在其中一个，该值将拼接到原始`$error`中，其中包含来自预处理语句对象的任何错误消息。

保存`blog_insert_mysqli.php`并在浏览器中测试。尝试上传过大的图像或类型错误的文件。表单应重新显示，并附带错误消息，同时保留博客详细信息。同时尝试插入带有或不带有图像和/或分类的博客条目。你现在拥有一个通用的插入表单。

如果没有合适的图像上传，请使用`phpsols images`文件夹中的图像。`Upload`类会重命名它们以避免覆盖现有文件。

你可以对照`ch16`文件夹中的`blog_insert_mysqli_05.php`检查你的代码。

#### PDO 版本的主要区别

最终的 PDO 版本可以在`ch16`文件夹的`blog_insert_pdo_05.php`中找到。它遵循与 MySQLi 版本相同的基本结构和逻辑，但在将值插入数据库的方式上存在一些重要区别。

步骤 2 中的代码紧密遵循 MySQLi 版本，但使用命名占位符代替匿名占位符。要获取受影响的行数，PDO 在语句对象上使用`rowCount()`方法。最近一次插入操作的主键通过连接对象的`lastInsertId()`方法获取。与 MySQLi 的`insert_id`属性一样，你需要在`INSERT`查询执行后立即访问它。

最大的变化在步骤 3 中将详细信息插入`blog`表的代码中。由于 PDO 可以使用`bindValue()`将`null`值插入列，因此只需要一个预处理语句。步骤 3 的 PDO 代码如下所示：

```
// don't insert blog details if the image failed to upload
if (!isset($imageError)) {
    // create SQL
    $sql = 'INSERT INTO blog (image_id, title, article)
            VALUES(:image_id, :title, :article)';
    // prepare the statement
    $stmt = $conn->prepare($sql);
    // bind the parameters
    // if $image_id exists, use it
    if (isset($image_id)) {
        $stmt->bindParam(':image_id', $image_id, PDO::PARAM_INT);
    } else {
        // set image_id to NULL
        $stmt->bindValue(':image_id', NULL, PDO::PARAM_NULL);
    }
    $stmt->bindParam(':title', $_POST['title'], PDO::PARAM_STR);
    $stmt->bindParam(':article', $_POST['article'], PDO::PARAM_STR);
    // execute and get number of affected rows
    $stmt->execute();
    $OK = $stmt->rowCount();
}
```

如果已上传图像，高亮显示的条件语句将`$image_id`的值绑定到命名的`:image_id`占位符。但如果没有上传图像，`bindValue()`将值设置为`NULL`。

在步骤 4 中，PDO 版本使用`exec()`而不是`query()`将值插入`article2cat`表。`exec()`方法执行 SQL 查询并返回受影响的行数，因此当不需要预处理语句时，应将其与`INSERT`、`UPDATE`和`DELETE`查询一起使用。

另一个重要区别在于构建错误消息的代码（如果出现问题）。由于在 PDO 中创建和准备语句是一个步骤，因此语句对象可能不存在（如果出现问题）。因此，在尝试调用`errorInfo()`方法之前，需要检查其是否存在。如果没有语句，则从数据库连接对象获取错误消息。还需要将`$error`初始化为空字符串，以便将各种消息拼接到其中，如下所示：

```
// redirect if successful or display error
if ($OK && !isset($imageError) && !isset($catError)) {
    header('Location: http://localhost/phpsols/admin/blog_list_pdo.php');
    exit;
} else {
    $error = '';
    if (isset($stmt)) {
        $errorInfo = $stmt->errorInfo();
    } else {
        $errorInfo = $conn->errorInfo();
    }
    if (isset($errorInfo[2])) {
        $error .= $errorInfo[2];
    }
    if (isset($imageError)) {
        $error .= ' ' . $imageError;
    }
    if (isset($catError)) {
        $error .= ' ' . $catError;
    }
}
```

## 更新和删除多表中的记录

`categories`和`article2cat`表的添加意味着上一章中 PHP Solutions 15-2 和 15-3 中的`blog_update_mysqli.php`和`blog_update_pdo.php`的更改不再充分覆盖`phpsols`数据库中的外键关系。除了修改更新表单外，你还需要创建脚本来删除记录而不破坏数据库的引用完整性。

### 更新交叉引用表中的记录

交叉引用表中的每条记录仅包含一个复合主键。通常，主键绝不应更改。此外，它们必须是唯一的。这对更新`article2cat`表带来了问题。如果在更新博客条目时不对选中的分类进行任何更改，则无需更新交叉引用表。但是，如果更改了分类，则需要确定要删除哪些交叉引用以及要插入哪些新引用。

与其纠结于是否进行了任何更改，一个简单的解决方案是删除所有现有的交叉引用，然后重新插入选中的分类。如果没有进行任何更改，只需重新插入相同的分类即可。



#### PHP 解决方案 16-5：为更新表单添加分类功能

本 PHP 解决方案对上一章 PHP 解决方案 15-2 中的 `blog_update_mysqli.php` 进行了修改，使其能够更新与博客文章关联的分类。为简化结构，对文章关联图片的唯一修改是选择另一张现有图片或不选择任何图片。

请继续使用 PHP 解决方案 15-2 中的 `blog_update_mysqli.php`。或者，从 `ch16` 文件夹中复制 `blog_update_mysqli_04.php`，并将其保存到 `admin` 文件夹中，命名为 `blog_update_mysqli.php`。当页面首次加载时，需要执行第二个查询来获取与博客文章关联的分类。在获取所选记录详细信息的条件语句中，添加以下高亮显示的代码：

```
$stmt->free_result();

// 获取与文章关联的分类
$sql = 'SELECT cat_id FROM article2cat
WHERE article_id = ?';

if ($stmt->prepare($sql)) {
    $stmt->bind_param('i', $_GET['article_id']);
    $OK = $stmt->execute();
    $stmt->bind_result($cat_id);

    // 循环遍历结果并将其存储到数组中
    $selected_categories = [];
    while ($stmt->fetch()) {
        $selected_categories[] = $cat_id;
    }
}
```

该查询从交叉引用表中选择所有与所选博客文章主键匹配的记录的 `cat_id`。结果绑定到 `$cat_id`，然后通过 `while` 循环将值提取到名为 `$selected_categories` 的数组中。

在 HTML 页面的主体中，在文本区域和显示图片列表的 `<select>` 下拉菜单之间添加一个多选的 `<select>` 列表。使用另一个 SQL 查询来填充该列表，操作如下：

```
<p>
<label for="category">分类：</label>
<select name="category[]" size="5" multiple id="category">
<?php
// 获取分类
$getCats = 'SELECT cat_id, category FROM categories
ORDER BY category';
$categories = $conn->query($getCats);
while ($row = $categories->fetch_assoc()) {
?>
<option value="<?= $row['cat_id']; ?>" <?php
if (isset($selected_categories) &&
in_array($row['cat_id'], $selected_categories)) {
echo 'selected';
} ?>><?= $row['category']; ?></option>
<?php } ?>
</select>
</p>
```

`while` 循环通过将 `cat_id` 插入 `value` 属性，并在开闭标签之间显示分类名称来构建每个 `<option>` 标签。如果 `cat_id` 存在于 `$selected_categories` 数组中，则在 `<option>` 标签中插入 `selected`。这会选中已与博客文章关联的分类。

保存 `blog_update_mysqli.php`，然后在 `blog_list_mysqli.php` 中选择一个“编辑”链接，确保多选列表已填充分类。如果你在 PHP 解决方案 16-4 中插入了新文章，则与该文章关联的分类应处于选中状态，如下方截图所示。

如有必要，你可以对照 `ch16` 文件夹中的 `blog_update_mysqli_05.php` 检查代码。对应的 PDO 版本位于 `blog_update_pdo_05.php`。

接下来，你需要编辑提交表单时更新记录的代码部分。新代码首先删除交叉引用表中与 `article_id` 匹配的所有条目，然后插入在更新表单中选择的值。内联注释表示为了节省空间而省略了现有代码。

```
// 如果表单已提交，则更新记录
if (isset($_POST ['update'])) {
    // 准备更新查询
    if (!empty($_POST['image_id'])) {
        // 现有代码已省略
    } else {
        // 现有代码已省略
        $stmt->execute();
    }

    // 删除交叉引用表中的现有值
    $sql = 'DELETE FROM article2cat WHERE article_id = ?';
    if ($stmt->prepare($sql)) {
        $stmt->bind_param('i', $_POST['article_id']);
        $stmt->execute();
    }

    // 将新值插入 articles2cat
    if (isset($_POST['category']) && is_numeric($_POST['article_id'])) {
        $article_id = (int) $_POST['article_id'];
        foreach ($_POST['category'] as $cat_id) {
            $values[] = "($article_id, " . (int) $cat_id . ')';
        }
        if ($values) {
            $sql = 'INSERT INTO article2cat (article_id, cat_id)
            VALUES ' . implode(',', $values);
            if (!$conn->query($sql)) {
                $catError = $conn->error;
            }
        }
    }
}
```

插入更新表单中所选值的代码与 PHP 解决方案 16-4 第 4 步中的代码相同。关键点在于它使用的是 `INSERT` 查询，而不是 `UPDATE`。原始值已被删除，因此你是在重新添加它们。

保存 `blog_update_mysqli.php`，并通过更新 `blog` 表中的现有记录进行测试。如有必要，你可以对照 `ch16` 文件夹中的 `blog_update_mysqli_06.php` 检查代码。对应的 PDO 版本位于 `blog_update_pdo_06.php`。

### 在删除时维护参照完整性

在 PHP 解决方案 16-5 中，删除交叉引用表中的记录时无需担心参照完整性，因为每条记录中存储的值都是外键。每条记录仅仅引用存储在 `blog` 和 `categories` 表中的主键。参考本章开头图 16-1，从交叉引用表中删除将 `article_id 2` 与 `cat_id 1` 组合的记录，仅仅是断开了标题为“见习艺伎去购物”的文章与`京都`分类之间的链接。文章和分类本身都不会受到影响。它们都保留在各自的表中。

如果决定删除文章或分类，情况则完全不同。如果从 `blog` 表中删除“见习艺伎去购物”这篇文章，则必须同时从交叉引用表中删除所有对 `article_id 2` 的引用。同样，如果删除`京都`分类，则必须从交叉引用表中删除所有对 `cat_id 1` 的引用。或者，如果某个项目的主键作为外键存储在其他地方，则必须阻止删除操作。

最好的方法是通过建立外键约束来实现。为此，相关表必须使用 InnoDB 存储引擎。如果你使用的是 MySQL 或 MariaDB 5.5 或更高版本，InnoDB 是默认引擎。此外，随书附带的所有 `.sql` 文件都选择 InnoDB 引擎。但是，如果你有使用 MyISAM 存储引擎创建的现有表，则需要先转换它们才能建立外键约束。

#### PHP 解决方案 16-6：将表转换为 InnoDB 存储引擎

本 PHP 解决方案演示了如何将表从 MyISAM 转换为 InnoDB。如果你计划将表上传到远程服务器，该服务器也必须支持 InnoDB（请参阅 PHP 解决方案 16-1）。

在 phpMyAdmin 中选择 `phpsols` 数据库，然后选择 `article2cat` 表。点击屏幕右上角的“操作”选项卡。在“表选项”部分，“存储引擎”字段会显示该表当前使用的引擎。如果显示为 MyISAM，请从下拉菜单中选择 InnoDB，如图 16-10 所示。

在 phpMyAdmin 中更改表的存储引擎非常容易。点击“执行”。更改存储引擎就是这么简单！

每个表都需要单独转换。你无法一次性更改数据库中的所有表。



