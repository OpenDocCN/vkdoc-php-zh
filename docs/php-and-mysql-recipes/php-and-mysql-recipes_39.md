# 第 16 章 使用 MySQL 数据库

```php
}
return $albums;
}

$music = new music();
$id = $music->addArtist("Pink Floyd");
$music->addAlbum("The Dark Side Of The Moon", $id);
$music->addAlbum("The Wall", $id);
$music->addAlbum("Widh You Were Here", $id);
```

## 第 16 章 使用 MySQL 数据库

```php
print_r($music->getArtists());
print_r($music->getAlbums(1));
print_r($music->getAlbums(2));
print_r($music->getAlbums());
```

该脚本现在可以插入一位艺术家并获取所有艺术家的列表，而插入专辑的函数已修改为接受一个额外的参数用于指定艺术家 ID。如果省略此参数，专辑将被创建而不与任何艺术家关联。

`getAlbums()` 方法也进行了修改，允许传入可选的艺术家 ID。如果不包含此参数，查询将执行一种特殊的连接，确保 `albums` 表中的所有行都被选中，并与 `artist` 表中的所有行关联。如果在 `artist` 表中没有匹配的行（`artist_id` 为 `null`），该行仍会被选中，但 `artist_id` 和 `artist_name` 将返回 `null`。这种方式称为“左外连接”。

如果提供了艺术家 ID 的值，则会进行简单的连接，只选择同时存在于 `album` 和 `artist` 中的记录，如下面的输出所示。四个数组分别代表：所有艺术家、所有披头士专辑、所有平克·弗洛伊德专辑以及所有专辑。

```php
Array
(
    [0] => Array
        (
            [id] => 1
            [artist_name] => The Beatles
            [description] =>
        )

    [1] => Array
        (
            [id] => 2
            [artist_name] => Pink Floyd
            [description] =>
        )
)

Array
(
    [0] => Array
        (
            [id] => 1
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Yellow Submarine
        )

    [1] => Array
        (
            [id] => 2
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Sgt. Pepper's Lonely Hearts Club Band
        )
```

## 第 16 章 使用 MySQL 数据库

```php
    [2] => Array
        (
            [id] => 3
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Help
        )

    [3] => Array
        (
            [id] => 4
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Abbey Road
        )

    [4] => Array
        (
            [id] => 5
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Let It Be
        )

    [5] => Array
        (
            [id] => 6
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Twist and Shout
        )
)

Array
(
    [0] => Array
        (
            [id] => 7
            [artist_id] => 2
            [artist_name] => Pink Floyd
            [title] => The Dark Side Of The Moon
        )

    [1] => Array
        (
            [id] => 8
            [artist_id] => 2
            [artist_name] => Pink Floyd
            [title] => The Wall
        )
```

## 第 16 章 使用 MySQL 数据库

```php
    [2] => Array
        (
            [id] => 9
            [artist_id] => 2
            [artist_name] => Pink Floyd
            [title] => Widh You Were Here
        )
)

Array
(
    [0] => Array
        (
            [id] => 1
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Yellow Submarine
        )

    [1] => Array
        (
            [id] => 2
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Sgt. Pepper's Lonely Hearts Club Band
        )

    [2] => Array
        (
            [id] => 3
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Help
        )

    [3] => Array
        (
            [id] => 4
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Abbey Road
        )

    [4] => Array
        (
            [id] => 5
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Let It Be
        )
```

## 第 16 章 使用 MySQL 数据库

```php
    [5] => Array
        (
            [id] => 6
            [artist_id] => 1
            [artist_name] => The Beatles
            [title] => Twist and Shout
        )

    [6] => Array
        (
            [id] => 7
            [artist_id] => 2
            [artist_name] => Pink Floyd
            [title] => The Dark Side Of The Moon
        )

    [7] => Array
        (
            [id] => 8
            [artist_id] => 2
            [artist_name] => Pink Floyd
            [title] => The Wall
        )

    [8] => Array
        (
            [id] => 9
            [artist_id] => 2
            [artist_name] => Pink Floyd
            [title] => Widh You Were Here
        )
)
```

## 配方 16-5：更新数据

### 问题

向表中添加新记录或行并不是用户可能想要的唯一操作。我们如何创建一个能够更新表中一个或多个列的方法？

### 解决方案

在 SQL 中，可以使用 `update` 语句来更新表中一行或多行的一个或多个列。

与 `insert` 语句一样，`update` 语句只针对单个表进行操作。如果需要更新多个表中的记录，则需要执行多个 `update` 语句。

`update` 语句的结构以标识要更新的表开始，后跟一个逗号分隔的列名和值列表，最后是一个 `where` 子句，用于标识要更新的行。如果省略 `where` 子句，`update` 语句将更新表中的所有行。

在 `where` 子句中指定主键是更新单条记录最简单的方法，但也可以使用其他值来标识要更新的行。

## 第 16 章 使用 MySQL 数据库

### 实现原理

以下代码部分是 `music` 类中一个名为 `updateAlbum()` 的方法示例。该方法接受两个参数。第一个参数是 `id` 列，用于标识要更新的行；第二个参数是一个值数组，类似于 `getAlbum()` 方法返回的值。

```php
function updateAlbum($id, $data = []) {
    $schema = [
        "id" => "int",
        "title" => "char",
        "artist_id" => "int",
        "description" => "char",
    ];
    $columns = [];
    foreach($data as $k => $v) {
        if (array_key_exists($k, $schema)) {
            if (is_null($v)) {
                $column[] = "$k = NULL";
            }
            else {
                switch($schema[$k]) {
                    "int" :
                        $column[] = "$k = " . (int)$v;
                        break;
                    "char" :
                        $column[] = "$k = '" . $this->real_escape_string($v) ."'"; break;
                }
            }
        }
    }
    if (sizeof($columns)) {
        $sql = "update album set " . implode(", ", $columns) .
            " where id = $id;";
        $this->Query($sql);
    }
}
```

该函数包含一个模式定义，用于验证 `$data` 变量的内容。

任何在 `$schema` 定义中找不到的值都将被忽略。这将消除用户尝试更新不存在的列时可能产生的 SQL 错误。模式还定义了列的类型，使代码能够执行正确的类型转换。

模式定义不包含列长度或其他可用于额外验证的限制。可以轻松地将这些作为模式定义中的附加参数添加。

通过添加一个包含表名和模式定义参数的私有方法，可以使此函数更具通用性。然后可以使用此方法为 `album` 和 `artist` 表添加更新方法。

## 第 16 章 使用 MySQL 数据库

这两个表都使用相同类型的主键。在这两种情况下，主键都是一个自动递增的整数值。使用不包含任何实际信息的私有键始终是一个好主意，特别是当主键被用作外键引用时。更改此类键的值将需要更改所有引用它的表中的值。如果键不包含任何实际信息，则永远没有理由更改它。

```php
private function updateRow($table, $schema, $id, $data) {
    $columns = [];
    foreach($data as $k => $v) {
        if (array_key_exists($k, $schema)) {
            if (is_null($v)) {
                $column[] = "$k = NULL";
            }
            else {
                switch($schema[$k]) {
                    "int" :
                        $column[] = "$k = " . (int)$v;
                        break;
                    "char" :
                        $column[] = "$k = '" . $this->real_escape_string($v) ."'"; break;
                }
            }
        }
    }
    if (sizeof($columns)) {
        $sql = "update $table set " . implode(", ", $columns) .
            " where id = $id;";
        $this->Query($sql);
    }
}

function updateAlbum($id, $data = []) {
    $schema = [
        "id" => "int",
        "title" => "char",
        "artist_id" => "int",
        "description" => "char",
    ];
    $this->update("album", $schema, $id, $data);
}

function updateArtist($id, $data = []) {
    $schema = [
        "id" => "int",
        "artist_name" => "char",
        "description" => "char",
    ];
    $this->update("artist", $schema, $id, $data);
}
```

## 第 16 章 使用 MySQL 数据库

## 配方 16-6：删除数据

### 问题

下一个问题是如何清除表中/数据库中不再需要的数据？

### 解决方案

解决方案是使用 SQL 的 `delete` 语句。该语句以 `delete from table` 开头，并带有一个可选的 `where` 子句，用于指定要删除的确切行。如果不包含 `where` 子句，指定表中的所有行都将被删除。



如果其他表的外键关联到正在被删除的任意行，该查询将会失败，因为外键关系将不再指向有效的行。这可以通过更新其他表中与被删除行相关的所有行来解决，使它们不再指向这些行——如果这些行需要保留在数据库中的话。如果这些行也应该被删除，则可以告诉数据库删除被选中的行，以及其他表中指向这些被删除行的所有行。在`delete`查询末尾添加关键字`cascade`即可实现此操作。

用于删除`album`表中所有行的 SQL 语句如下：`delete from album;`

用于删除属于特定艺人的所有行的语句如下：`delete from album where artist_id = 2;`

如果艺人的`Id`未知，可以通过编写子查询来查找：`delete from album where artist_id in (select id from artist where artist_name='Pink Floyd');`这里的`where`子句使用了关键字`in`而不是`=`。如果子查询返回多行，这将非常有用。这将确保删除与子查询返回的任何`id`匹配的所有行。如果子查询返回多行并且主查询使用了`=`字符，SQL 查询将失败并抛出基数异常（cardinal exception）。这表明`artist_id`不能等于多个值，但它可以存在于多个值的列表中。

### 工作原理

与上一节中的`update`示例类似，我们现在可以编写`delete`函数。第一个函数将是一个私有成员函数，能够根据提供的表名和`id`从任何表中删除数据。另外两个函数将是公共成员函数，用于删除特定的`id`。

```
private function deleteRow($table, $id) {
  $sql = "delete from $table where id = " . (int)$id . ";";
  $this->Query($sql);
}
```

```
function deleteArtist($id) {
  $sql = "update album set artist_id=NULL where id = " . (int)$id . ";";
  $this->Query($sql);
  $this->deleteRow("artist", $id);
}
```

```
function deleteAlbum($id) {
  $this->deleteRow("album", $id);
}
```

在`deleteArtist()`函数中，有一个额外的语句用来删除所有指向被删除行的约束。

现在可以将包含插入、更新和删除函数的整个类合并为一个。

```
<?php
// 16_9.php
class music extends mysqli {
  private $schema = [
    'artist' => [
      'id' => 'int',
      'artist_name' => 'char',
      'description' => 'char',
    ],
    'album' => [
      'id' => 'int',
      'title' => 'char',
      'artist_id' => 'int',
      'description' => 'char',
    ]
  ];

  function __construct() {
    parent::__construct('127.0.0.1', 'php', 'secret', 'music');
    if ($this->connect_error) {
      die("Unable to connect to music\n");
    }
  }

  function insertRow($table, $schema, $data = []) {
    $columns = [];
    $values = [];
    foreach($data as $k => $v) {
      if (array_key_exists($k, $schema)) {
        $columns[] = $k;
        if (is_null($v)) {
          $values[] = "NULL";
        }
        else {
          switch($schema[$k]) {
            case 'int' :
              $values[] = (int)$v;
              break;
            case 'char' :
              $values[] = "'" . $this->real_escape_string($v) . "'";
              break;
          }
        }
      }
    }
    if (sizeof($columns)) {
      $sql = "insert into $table (" . implode(", ", $columns) . ") values (" .
        implode(", ", $values) . ");";
      $this->Query($sql);
      return $this->insert_id;
    }
    else {
      return false;
    }
  }

  function addArtist($name, $description = null) {
    return $this->insertRow(
      'artist', $this->schema['artist'], [
        'artist_name' => $name,
        'description' => $description
      ]
    );
  }

  function addAlbum($title, $artist_id, $description = null) {
    return $this->insertRow(
      'album', $this->schema['album'], [
        'title' => $title,
        'artist_id' => $artist_id,
        'description' => $description
      ]
    );
  }

  private function updateRow($table, $schema, $id, $data) {
    $columns = [];
    foreach($data as $k => $v) {
      if (array_key_exists($k, $schema)) {
        if (is_null($v)) {
          $column[] = "$k = NULL";
        }
        else {
          switch($schema[$k]) {
            case 'int' :
              $column[] = "$k = " . (int)$v;
              break;
            case 'char' :
              $column[] = "$k = '" . $this->real_escape_string($v) ."'";
              break;
          }
        }
      }
    }
  }
}
```



