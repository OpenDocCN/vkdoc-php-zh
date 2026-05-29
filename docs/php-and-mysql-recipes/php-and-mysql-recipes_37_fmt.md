# 配方 16-3. 获取数据

### 问题

数据库使用内部格式存储数据，并优化了索引和内存的使用，以便基于查询提取数据。如果使用`mysql`命令行工具，查询结果会以文本形式列在控制台上。那么，如何将数据提取到 PHP 变量中，以便在脚本中使用这些数据执行操作或生成输出呢？

### 解决方案

SQL 数据库的矩形网格特性可以很容易地在 PHP 数组中表示。这将是一个数组的数组。外层数组对应结果中的行，内层数组代表每一行中所有列的值。

### 工作原理

为了在表中有数据可供获取，我们首先需要插入一些数据。使用`mysql`命令行工具执行以下命令：

```sql
use music;

insert into album (title) values('Yellow Submarine');
insert into album (title) values('Sgt. Pepper''s Lonely Hearts Club Band');
insert into album (title) values('Help');
insert into album (title) values('Abbey Road');
```

为了验证数据，从表中查询所有列：

```sql
MariaDB [music]> select * from album;
+----+---------------------------------------+-------------+
| id | title                                 | description |
+----+---------------------------------------+-------------+
|  1 | Yellow Submarine                      | NULL        |
|  2 | Sgt. Pepper's Lonely Hearts Club Band | NULL        |
|  3 | Help                                  | NULL        |
|  4 | Abbey Road                            | NULL        |
+----+---------------------------------------+-------------+
4 rows in set (0.00 sec)
```

注意第二条专辑的插入语句中，撇号处有两个引号。这是因为单引号是字符串分隔符，为了插入单引号，必须对其进行转义，以告知解析器这并非字符串的结束。

`Id`列是自动添加的，因为插入语句未提供该列；`description`字段未指定，因此显示为`NULL`值。

下一个示例演示如何创建扩展`mysqli`类的类。

```php
<?php

// 16_6.php

ini_set('display_errors', 1);

class music extends mysqli {

function __construct() {

parent::__construct('127.0.0.1', 'php', 'secret', 'music');

if ($this->connect_error) {

die("Unable to connect to music\n");

}

}

function getAlbums() {

$albums = [];

$result = $this->Query("select * from album;");

if ($result) {

while($row = $result->fetch_assoc()) {

$albums[] = $row;

}

$result->close();

}

return $albums;

}

}

$music = new music();

print_r($music->getAlbums());
```

`music`类的构造函数不接受任何参数，但它会使用一些硬编码参数（主机、用户、密码和数据库）调用父类的构造函数。`getAlbums()`方法执行查询`"select * from album;"`，如果查询成功执行，它会循环遍历结果，并将每一行插入到一个数组中，最后返回该数组。在函数返回前关闭结果集，这会释放查询所使用的任何内存。

此脚本的输出如下所示：

```
Array
(
    [0] => Array
        (
            [id] => 1
            [title] => Yellow Submarine
            [description] =>
        )

    [1] => Array
        (
            [id] => 2
            [title] => Sgt. Pepper's Lonely Hearts Club Band
            [description] =>
        )

    [2] => Array
        (
            [id] => 3
            [title] => Help
            [description] =>
        )

    [3] => Array
        (
            [id] => 4
            [title] => Abbey Road
            [description] =>
        )

)
```

有几种不同的函数可以从结果集中获取数据。这里使用的是`fetch_assoc()`。它会为每一行返回一个关联数组。这就是为什么内部数组中的每个元素都有与所选列名匹配的键。在这个例子中，查询使用了`"select * from album;"`来选择所有列，但我们也可以编写`"select id, title from album;"`来仅选择前两列。

## Recipe 16-4. 插入数据

### 问题

上一个示例中的类可用于从数据库的特定表中提取数据，但如何添加新行呢？

### 解决方案

在从数据库提取数据之前，必须先添加数据。这通过命令行工具使用`insert`语句完成。`insert`语句可以作用于单个表，并可用于创建包含任意数量列的新记录。在命令行示例中，我们只提供了`title`列。在此示例中，该列是必填列，因为它定义了`NOT NULL`约束。

一个可以向数据库添加行的简单函数可以这样编写：

```php
function addAlbum($title, $description = null) {
    $t = $this->real_escape_string($title);
    if ($description) {
        $d = $this->real_escape_string($description);
        $sql = "insert into album (title, description) values ('$t', '$d')";
    }
    else {
        $sql = "insert into album (title) values ('$t')";
    }
    $this->Query($sql);
    return $this->insert_id;
}
```

此方法使用`real_escape_string()`函数来确保单引号和其他字符在能够被包含在查询中之前得到处理。返回值将是新创建行的`id`。这可以在以后用于将其他数据链接到特定记录。

### 工作原理

用于插入新专辑并获取所有专辑列表的完整脚本如下所示：

```php
<?php

// 16_7.php

ini_set('display_errors', 1);

class music extends mysqli {

function __construct() {

parent::__construct('127.0.0.1', 'php', 'secret', 'music');

if ($this->connect_error) {

die("Unable to connect to music\n");

}

}

function addAlbum($title, $description = null) {

$t = $this->real_escape_string($title);

if ($description) {
```