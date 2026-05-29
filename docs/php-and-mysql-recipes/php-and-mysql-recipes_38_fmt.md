# 第 16 章：使用 MySQL 数据库

包含专辑的表中缺少一些信息。到目前为止，只添加了单个乐队的专辑，但如果要添加更多乐队呢？简单的解决方案是向 `album` 表中添加一个额外的字符列。这样每条记录都可以插入一个文本值。使用文本列可以为每条记录设置不同的值，即使是同一乐队也不例外。在这种情况下，我们可能会使用 "The Beatles"、"Beatles"、"Beatles, The" 以及许多其他组合。这可能会导致难以查找某个乐队的全部专辑。

为了解决这个问题，我们可以创建一个名为 `artist` 的新表，并将 `album` 表链接到该表。

执行这些操作的 SQL 语句如下：

```sql
create table artist (
    id int auto_increment,
    artist_name varchar(250) not null,
    description varchar(2000),
    primary key(id)
);
```

以及用来更新 `album` 表、添加一个允许链接到 `artist` 表的额外列的 SQL 语句：

```sql
alter table album add artist_id int;
alter table album add foreign key(artist_id) references artist(id);
```

第一条语句更改了 `album` 表的列定义，第二条语句则在 `album` 和 `artist` 之间创建了一个外键关系。如果存在一个或多个专辑与该艺术家相关联，此关系将阻止删除该艺术家。

到目前为止，`albums` 表中只创建了披头士乐队的专辑。我们可以先将他们创建为一个艺术家，然后将专辑链接到这个新艺术家。

```sql
insert into artist (artist_name) values ('The Beatles');
update album set artist_id=1;
```

在这种情况下，第一个添加的艺术家将获得 `id=1`，因此通过为 `album` 表中的所有行设置 `artist_id=1` 来建立链接。

现在我们可以扩展这个类，以便插入新的艺术家和专辑，并在查询中返回 `artist_name`。

```php
<?php
// 16_8.php
ini_set('display_errors', 1);

class music extends mysqli {
    function __construct() {
        parent::__construct('127.0.0.1', 'php', 'secret', 'music');
        if ($this->connect_error) {
            die("Unable to connect to music\n");
        }
    }

    function addArtist($name, $description = null) {
        $n = $this->real_escape_string($name);
        if ($description) {
            $d = $this->real_escape_string($description);
            $sql = "insert into artist (artist_name, description) values ('$n', '$d')";
        } else {
            $sql = "insert into artist (artist_name) values ('$n')";
        }
        $this->Query($sql);
        return $this->insert_id;
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

    function addAlbum($title, $artist_id = null, $description = null) {
        if (is_null($artist_id)) {
            $id = "NULL";
        } else {
            $id = (int)$artist_id;
        }
        $t = $this->real_escape_string($title);
        if ($description) {
            $d = $this->real_escape_string($description);
            $sql = "insert into album (title, artist_id, description) values ('$t', $id, '$d')";
        } else {
            $sql = "insert into album (title, artist_id) values ('$t', $id)";
        }
        $this->Query($sql);
        return $this->insert_id;
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
}

$music = new music();

$music->addAlbum("Let It Be");
$music->addAlbum("Twist and Shout");

print_r($music->getAlbums());
```