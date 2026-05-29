# PHP 解决方案 13-4：统计结果集中的记录数（PDO）

PDO 没有与 MySQLi 的 `num_rows` 属性直接对应的功能。对于大多数数据库，您需要执行一条 SQL 查询来统计表中的项目数，然后获取结果。然而，PDO 的 `rowCount()` 方法在 MySQL 和 MariaDB 中具有双重用途。通常，它只报告插入、更新或删除记录所影响的行数，但在 MySQL 和 MariaDB 中，它也会报告 `SELECT` 查询找到的记录数。

1.  在 `phpsols-4e` 站点中创建一个名为 `pdo` 的新文件夹。然后，在刚创建的文件夹中创建一个名为 `pdo.php` 的文件。该页面最终将用于显示一个表格，因此它应该包含一个 `DOCTYPE` 声明和 HTML 骨架。

2.  在 `DOCTYPE` 声明上方的 PHP 代码块中包含连接文件，然后使用只读账户创建到 `phpsols` 数据库的 PDO 连接，如下所示：

    ```php
    require_once '../includes/connection.php';
    $conn = dbConnect('read', 'pdo');
    ```

3.  接下来，准备 SQL 查询：

    ```php
    $sql = 'SELECT * FROM images';
    ```

    这意味着“选择 `images` 表中的每条记录”。星号（`*`）是“所有列”的简写。

4.  现在执行查询并将结果存储在变量中，如下所示：

    ```php
    $result = $conn->query($sql);
    ```

5.  要检查查询是否存在问题，可以使用连接对象的 `errorInfo()` 方法从数据库中获取错误信息数组。如果出现问题，数组的第三个元素包含问题的简要描述。添加以下代码：

    ```php
    $error = $conn->errorInfo()[2];
    ```

    我们只关心第三个元素，因此可以使用在 PHP 解决方案 7-1（“获取文本文件的内容”）中遇到的数组解引用技术：在 `$conn->errorInfo()` 调用的右侧直接添加方括号内的数组索引，并将值赋给 `$error`。

6.  如果查询执行成功，`$error` 将为 `null`，PHP 将其视为 `false`。因此，如果没有错误，我们可以通过调用 `$result` 对象的 `rowCount()` 方法来获取结果集中的行数，如下所示：

    ```php
    if (!$error) {
    $numRows = $result->rowCount();
    }
    ```

7.  现在可以在页面主体中显示查询结果，如下所示：

    ```php
    $error";
    } else {
    echo "共找到 $numRows 条记录。";
    }
    ?>
    ```

8.  保存页面并在浏览器中加载。您应该会看到与 PHP 解决方案 13-2 的第 8 步相同的结果。如有必要，请对照 `pdo_01.php` 检查您的代码。

## 在其他数据库中使用 PDO 统计记录数

使用 PDO 的 `rowCount()` 来报告 `SELECT` 查询找到的项目数适用于 MySQL 和 MariaDB，但不能保证在所有其他数据库上都能工作。如果 `rowCount()` 不起作用，请改用以下代码：

```php
// 准备 SQL 查询
$sql = 'SELECT COUNT(*) FROM images';
// 提交查询并捕获结果
$result = $conn->query($sql);
$error = $conn->errorInfo()[2];
if (!$error) {
// 找出检索到的记录数
$numRows = $result->fetchColumn();
// 释放数据库资源
$result->closeCursor();
}
```

这使用了带星号的 SQL `COUNT()` 函数来统计表中的所有项目。只有一个结果，因此可以使用 `fetchColumn()` 方法检索它，该方法从数据库结果中获取第一列。将结果存储在 `$numRows` 中后，必须调用 `closeCursor()` 方法来释放数据库资源，以便进行进一步的查询。