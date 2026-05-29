# 使用 MySQL 数据库

## 模式信息

### 问题

在之前的示例中，两张表的模式定义是硬编码的，如果数据库有任何更改，都需要进行更新。是否有可能从数据库中提取模式信息？

### 解决方案

MySQL 数据库中有一个特殊用途的数据库或模式，名为 `INFORMATION_SCHEMA`，它包含了服务器上任何数据库中所有表和列的信息。`INFORMATION_SCHEMA` 在许多 SQL92 数据库引擎中都是通用的。

在命令行中使用的 `show databases;` 和 `show tables;` 命令也可以从 `mysqli` 类的 `Query()` 方法中使用。这些命令利用 `INFORMATION_SCHEMA` 中的信息来提取有关系统的数据。

`show databases;` 命令等价于以下语句：

```sql
select schema_name as `Database` from information_schema.schemata;
```

别名 `Database` 两边的反引号是因为 `database` 是一个保留字，在查询中用作列名时必须进行转义。

类似地，我们可以编写一条语句来获取名为 `music` 的数据库中的所有表，这等价于以下命令：

```sql
use music;
Show tables;
```

使用信息模式可以让我们从任何位置执行此操作，无需在查询此模式之前选择特定的数据库。

```sql
select table_name from information_schema.tables where table_schema='music';
```

最后，我们可以使用以下 SQL 语句获取每张表中的列列表：

```sql
select column_name, column_type from information_schema.columns where table_name='album';
```

### 工作原理

有了这些信息，我们现在可以修改 `music` 类的构造函数，使其向数据库查询模式信息。

```php
<?php
// 16_10.php

class music extends mysqli {
    private $schema = [];

    function __construct() {
        parent::__construct('127.0.0.1', 'php', 'secret', 'music');
        if ($this->connect_error) {
            die("无法连接到音乐数据库\n");
        }

        $sql = "select table_name from information_schema.tables where table_schema='music';";
        $rs = $this->Query($sql);
        while ($table = $rs->fetch_assoc()) {
            $sql = "select column_name, column_type from information_schema.columns where table_name='{$table['table_name']}';";
            $rs1 = $this->query($sql);
            while($col = $rs1->fetch_assoc()) {
                $this->schema[$table['table_name']][$col['column_name']] = $col['column_type'];
            }
            $rs1->close();
        }
        $rs->close();
        print_r($this->schema);
    }
}

$music = new music();
```

`addArtist()` 和 `addAlbum()` 方法也被重构，以使用通用的模式定义。

```php
if (sizeof($columns)) {
    $sql = "update $table set " . implode(", ", $columns) .
           " where id = $id;";
    $this->Query($sql);
}
}

function updateArtist($id, $data = []) {
    $this->update("artist", $this->schema['artist'], $id, $data);
}

function updateAlbum($id, $data = []) {
    $this->update("album", $this->schema['album'], $id, $data);
}

private function deleteRow($table, $id) {
    $sql = "delete from $table where id = " . (int)$id . ";";
    $this->Query($sql);
}

function deleteArtist($id) {
    $sql = "update album set artist_id=NULL where id = " . (int)$id . ";";
    $this->Query($sql);
    $this->deleteRow("artist", $id);
}

function deleteAlbum($id) {
    $this->deleteRow("album", $id);
}

function getArtists() {
    $artists = [];
    $sql = "select * from artist";
    $result = $this->Query($sql);
    if ($result) {
        while($row = $result->fetch_assoc()) {
            $artists[] = $row;
        }
        $result->close();
    }
    return $artists;
}

function getAlbums($artist_id = null) {
    $albums = [];
    if ($artist_id) {
        $sql = "select album.id, artist_id, artist_name, title from album, artist where album.artist_id=artist.id and artist_id=$artist_id";
    } else {
        $sql = "select album.id, artist_id, artist_name, title from album left outer join artist on (album.artist_id=artist.id)";
    }
    $result = $this->Query($sql);
    if ($result) {
        while($row = $result->fetch_assoc()) {
            $albums[] = $row;
        }
        $result->close();
    }
    return $albums;
}
```