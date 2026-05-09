# 清单 9-13. `Database\Driver\Mysql` 数据访问方法

```php
namespace Framework\Database\Query
{

use Framework\Database as Database;
use Framework\Database\Exception as Exception;

class Mysql extends Database\Query
{

public function all()
{

$sql = $this->_buildSelect();
$result = $this->connector->execute($sql);

if ($result === false)
{

$error = $this->connector->lastError;
throw new Exception\Sql("There was an error with your SQL query: {$error}");
}

$rows = array();

for ($i = 0; $i < $result->num_rows; $i++)
{

$rows[] = $result->fetch_array(MYSQLI_ASSOC);
}

return $rows;
}

}
}
```

`all()` 方法负责根据执行的 `SELECT` 查询返回可变数量的行。它会调用 `_buildSelect()` 方法并尝试执行它。如果执行失败，将抛出 `Database\Exception\Sql` 异常。如果获得有效结果，则循环遍历结果，并将返回的关联数组的对象表示形式赋值给 `$rows` 数组。

`first()` 和 `count()` 方法都基于 `all()` 方法。`first()` 方法会修改 `$_limit` 和 `$_offset` 属性，以便仅返回一行。然后将这些属性重置为原始值，并返回该单行。

`count()` 方法同样修改 `$_limit` 和 `$_offset` 属性，同时还会修改 `$_fields` 属性。它添加了 `count(1)` 字段（这是对表执行单次查询时返回行数的最有效方式）。它调用 `all()`，获取第一行，取得计数值，然后将 `$_limit`、`$_offset` 和 `$_fields` 属性重置为原始值。

有了所有这些数据库代码，我们现在就可以使用这个稳定且多态的数据库库执行多种不同类型的查询。清单 9-14 中的代码提供了一个很好的综合性示例。

