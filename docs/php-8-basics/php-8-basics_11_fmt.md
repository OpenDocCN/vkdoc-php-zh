# 12. 带数据库的网站

在本章中，你将把所有学到的知识整合到一个示例网站中。这个网站将允许你创建、读取、更新和删除（即 CRUD）。一个基本的 CRUD 网站是企业或组织内管理信息的标准方式。仔细想想，几乎所有应用程序都可以归结为 CRUD。Facebook 允许你创建帖子、读取帖子、更新帖子或你的个人资料，以及删除信息。这种功能是大多数网站寻求的基本交互方式，但你的想象力是无限的，可以在此基础上进行拓展。

本章将涵盖以下内容：

- PHP CRUD `GET` 方法及示例函数：`deleteBook`、`showEditBook`、`showAddBook` 和 `showBooks`

- PHP CRUD `POST` 方法及示例：`bookToUpdate` 和 `bookToAdd`

在这个例子中，你将创建基本的 CRUD 供你进行审查和扩展。此 CRUD 同时使用了 `POST` 和 `GET` 方法以及 MySQL PDO 参数绑定。这是朝着开发更动态、更高级的应用程序迈出的很好的一步。

让我们直接进入 `chapter12` 链接中的 `home.php` 文件。

```php
```

这些第一行代码声明了一些全局变量。

```html
Home
Add
```

这部分为应用创建了头部和导航。目前，这些链接是静态的，但可以通过例如从数据库读取菜单项来使其动态化。

```php
try {
    $conn = new PDO($dsn, $user, $pass);
    echo '数据库连接成功';
    echo '';
} catch (\Throwable $t) {
    echo '错误：' . $t->getMessage();
    echo '';
}
```

这是您的基本数据库连接块。在这里，您尝试使用凭据进行连接，并在出现任何问题时返回错误。您将在接下来的几个函数中使用`$conn`变量。由于该变量存在于新函数的作用域之外，您需要在那些函数中使用`global $conn`。

```php
function deleteBook($theBook) {
    global $conn;
    $sql = "delete FROM `books` WHERE `id`=$theBook";
    $result = $conn->query($sql);
    echo "书籍已删除";
}
```

此函数`deleteBook`接收传入的变量`$theBook`，并通过特定的数据库查询定位数据库项。该函数随后返回"书籍已删除。"。此函数可以通过多种方式改进：

- 变量清理以防止 SQL 注入攻击

- 验证要删除的项目是否存在

- 检查 MySQL 错误并显示它们

```php
function showEditBook($theBook) {
    global $conn;
    $sql = "SELECT * FROM `books` WHERE `id`=$theBook";
    $result = $conn->query($sql);
    foreach($result as $row) {
        $addForm = '';
        $addForm .= '标题';
        $addForm .= '作者';
        $addForm .= '分类';
        $addForm .= 'ISBN';
        $addForm .= '';
        $addForm .= '';
        $addForm .= '';
        echo $addForm;
    }
}
```

函数`showEditBook`根据书籍 ID（`$theBook`）显示编辑书籍表单。通过此表单，您可以通过`POST`将其提交回`home.php`。使用此表单，您可以添加验证以确保值已正确填写并能够添加到数据库中。隐藏字段作为指示器，告知`home.php`如何处理表单提交。您将在稍后更新书籍的函数中处理这部分。

```php
function showAddBook() {
    $addForm = '';
    $addForm .= '标题';
    $addForm .= '作者';
    $addForm .= '分类';
    $addForm .= 'ISBN';
    $addForm .= '';
    $addForm .= '';
    $addForm .= '';
    echo $addForm;
}
```

函数`showAddBook`显示添加书籍表单。同样，这里使用隐藏字段通过`POST`通知`home.php`您想要执行的操作。

```php
function showBooks() {
    global $conn;
    $sql = "SELECT * FROM `books` WHERE `id`";
    $result = $conn->query($sql);
    if ($result !== false) {
        $rowCount = $result->rowCount();
        echo "书籍数量：$rowCount";
    }
    foreach($result as $row) {
        echo $row['id'].' - '. $row['title'] .' - '. $row['author'] .' - '. $row['category'] .' - '. $row['isbn'] .'  [  编辑  删除 ]';
    }
}
```

函数`showBooks`是页面的默认显示功能。它显示数据库中所有书籍及其编辑和删除链接。

```php
if (isset($_GET['q'])) {
    if ($_GET['q'] == 'add') {
        echo "添加书籍";
        showAddBook();
    }
    if ($_GET['q'] == 'edit') {
        $theBook = $_GET['book'];
        echo "编辑书籍";
        showEditBook($theBook);
    }
    if ($_GET['q'] == 'delete') {
        $theBook = $_GET['book'];
        echo "删除书籍";
        deleteBook($theBook);
    }
}
```

以上是用于通过`GET`确定要执行何种操作的逻辑。请记住，`GET`变量是在 URL 中使用的变量。您在 URL 中使用`q`作为分配给操作（add, edit, delete）的变量。

```php
if (isset($_POST['bookToUpdate'])) {
    global $conn;
    $sql = "update books set title=?, author=?, category=?, isbn=? where id=?";
    if ($stmt = $conn->prepare($sql)) {
        $stmt->bindParam(1,$_POST['title']);
        $stmt->bindParam(2,$_POST['author']);
        $stmt->bindParam(3,$_POST['category']);
        $stmt->bindParam(4,$_POST['isbn']);
        $stmt->bindParam(5,$_POST['bookToUpdate']);
        if($stmt->execute()) {
            echo "书籍 ". $_POST['title'] ." 已添加";
        }
    } else {
        echo "错误：" . $sql . "" . $conn->error;
        echo "预处理语句错误：".$stmt->error();
    }
}
```

上述`if`语句检查是否正在调用变量`bookToUpdate`。如果设置了此变量，则您将尝试更新一本书。您使用 PDO（如第[10]章所述）来准备一条语句，以确保防止 SQL 注入并指定变量。一旦`$stmt`执行成功，您将返回书名和"已添加"；否则，返回错误。这可以通过以下方式改进：

- 清理`POST`数据

- 验证数据库中的项目是否可更新

```php
if (isset($_POST['bookToAdd'])) {
    global $conn;
    $sql = "insert into books (title, author, category, isbn) VALUES (?,?,?,?)";
    if ($stmt = $conn->prepare($sql)) {
        $stmt->bindParam(1,$_POST['title']);
        $stmt->bindParam(2,$_POST['author']);
        $stmt->bindParam(3,$_POST['category']);
        $stmt->bindParam(4,$_POST['isbn']);
        if($stmt->execute()) {
            echo "新书籍已添加";
        }
    }
}
```

这个`if`语句检查`POST`变量`bookToAdd`。如果找到，则创建并执行 SQL 查询。这可以通过以下方式改进：

- 清理`POST`数据
- 验证项目是否已存在于数据库中
- 所有值都已填写

```php
showBooks();
```

这是此页面的默认视图，显示可用书籍列表：

```php
?>
```

结合上述改进，尝试将其转换为返回 JSON 数据的 API。输出不应返回 HTML，而应类似于以下伪代码：

```php
$sql = "SELECT * FROM `books` WHERE `id`";
$result = $conn->query($sql);
if ($result !== false) {
    $rowCount = $result->rowCount();
    $output[] =  "书籍数量：$rowCount";
}
foreach($result as $row) {
    $output[] = "标题：". $row['title'];
    $output[] = "作者：". $row['author'];
    $output[] = "分类：". $row['category'];
    $output[] = "ISBN：". $row['isbn'];
    $output = json_encode($output);
    Return $output;
}
```

## 总结

在本章中，您结合了迄今为止学到的所有内容，构建了一个示例网站。您学习了如何构建此网站以执行创建、读取、更新和删除（即 CRUD）操作，并将其与`POST`和`GET`方法以及 MySQL PDO 参数绑定结合使用。在下一章中，您将学习框架，这些框架使用了许多最佳实践和设计模式，以便开发人员能够快速使用它们来解决问题。