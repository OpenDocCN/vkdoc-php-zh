# 清单 9-14. 数据库使用示例

```php
$database = new Database(array(
"type" => "mysql",
"options" => array(
"host" => "localhost",
"username" => "prophpmvc",
"password" => "prophpmvc",
"schema" => "prophpmvc",
"port" => "3306"
)
));

$database = $database->initialize();

$all = $database->query()
->from("users", array(
"first_name",
"last_name" => "surname"
))
->join("points", "points.id = users.id", array(
"points" => "rewards"
))
->where("first_name = ?", "chris")
->order("last_name", "desc")
->limit(100)
->all();

$print = print_r($all, true);
echo "all => {$print}";

$id = $database->query()
->from("users")
->save(array(
"first_name" => "Liz",
"last_name" => "Pitt"
));

echo "id => {$id}\n";

$affected = $database->query()
->from("users")
->where("first_name = ?", "Liz")
->delete();

echo "affected => {$affected}\n";

$id = $database->query()
->from("users")
->where("first_name = ?", "Chris")
->save(array(
"modified" => date("Y-m-d H:i:s")
));

echo "id => {$id}\n";

$count = $database->query()
->from("users")
->count();

echo "count => {$count}\n";
```



根据您可能需要支持的数据库引擎类型，可以选择在基础 `Query` 类中保留最少的代码，而让每个 `Query` 子类为相应的引擎实现类似的逻辑。

**注意** 您选择应用于数据库表的排序规则将决定 SQL 查询在字符串值与表列数据比较时是否区分大小写。这可以在单个查询的基础上进行覆盖，但这种高级 SQL 内容超出了本书的讨论范围。

由于我更多熟悉关系型 SQL 数据库引擎，因此我选择将所有可复用的 SQL 生成代码保留在基础 `Query` 类中，而实际的 MySQL `Query` 子类只需要一个方法。下面让我们看看清单 9-15 中的完整 `Query` 类。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 9 章 ■ 数据库

### ***清单 9-15.*** `Database\Query` 类

```php
namespace Framework\Database
{

use Framework\Base as Base;
use Framework\ArrayMethods as ArrayMethods;
use Framework\Database\Exception as Exception;

class Query extends Base
{

    /**
     * @readwrite
     */
    protected $_connector;

    /**
     * @read
     */
    protected $_from;

    /**
     * @read
     */
    protected $_fields;

    /**
     * @read
     */
    protected $_limit;

    /**
     * @read
     */
    protected $_offset;

    /**
     * @read
     */
    protected $_order;

    /**
     * @read
     */
    protected $_direction;

    /**
     * @read
     */
    protected $_join = array();

    /**

     * @read
     */
    protected $_where = array();

    protected function _getExceptionForImplementation($method)
    {
        return new Exception\Implementation("{$method} method not implemented");
    }

    protected function _quote($value)
    {
        if (is_string($value))
        {
            $escaped = $this->connector->escape($value);
            return "'{$escaped}'";
        }
        if (is_array($value))
        {
            $buffer = array();
            foreach ($value as $i)
            {
                array_push($buffer, $this->_quote($i));
            }
            $buffer = join(", ", $buffer);
            return "({$buffer})";
        }
        if (is_null($value))
        {
            return "NULL";
        }
        if (is_bool($value))
        {
            return (int) $value;
        }
        return $this->connector->escape($value);
    }

    protected function _buildSelect()
    {
        $fields = array();
        $where = $order = $limit = $join = "";
        $template = "SELECT %s FROM %s %s %s %s %s";

        foreach ($this->fields as $table => $_fields)
        {
            foreach ($_fields as $field => $alias)
            {
                if (is_string($field))
                {
                    $fields[] = "{$field} AS {$alias}";
                }
                else
                {
                    $fields[] = $alias;
                }
            }
        }
        $fields = join(", ", $fields);

        $_join = $this->join;
        if (!empty($_join))
        {
            $join = join(" ", $_join);
        }

        $_where = $this->where;
        if (!empty($_where))
        {
            $joined = join(" AND ", $_where);
            $where = "WHERE {$joined}";
        }

        $_order = $this->order;
        if (!empty($_order))
        {
            $_direction = $this->direction;
            $order = "ORDER BY {$_order} {$_direction}";
        }

        $_limit = $this->limit;
        if (!empty($_limit))
        {
            $_offset = $this->offset;
            if ($_offset)
            {
                $limit = "LIMIT {$_limit}, {$_offset}";
            }
            else
            {
                $limit = "LIMIT {$_limit}";
            }
        }

        return sprintf($template, $fields, $this->from, $join, $where, $order, $limit);
    }

    protected function _buildInsert($data)
    {
        $fields = array();
        $values = array();
        $template = "INSERT INTO '%s' ('%s') VALUES (%s)";

        foreach ($data as $field => $value)
        {
            $fields[] = $field;
            $values[] = $this->_quote($value);
        }
        $fields = join("', '", $fields);
        $values = join(", ", $values);

        return sprintf($template, $this->from, $fields, $values);
    }

    protected function _buildUpdate($data)
    {
        $parts = array();
        $where = $limit = "";
        $template = "UPDATE %s SET %s %s %s";

        foreach ($data as $field => $value)
        {
            $parts[] = "{$field} = ".$this->_quote($value);
        }
        $parts = join(", ", $parts);

        $_where = $this->where;
        if (!empty($_where))
        {
            $joined = join(", ", $_where);
            $where = "WHERE {$joined}";
        }

        $_limit = $this->limit;
        if (!empty($_limit))
        {
            $_offset = $this->offset;
            $limit = "LIMIT {$_limit} {$_offset}";
        }

        return sprintf($template, $this->from, $parts, $where, $limit);
    }

    protected function _buildDelete()
    {
        $where = $limit = "";
    }
}
```



`$template = "DELETE FROM %s %s %s";`

`$_where = $this->where;`

```php
if (!empty($_where))
{
    $joined = join(", ", $_where);
    $where = "WHERE {$joined}";
}
```

`$_limit = $this->limit;`

```php
if (!empty($_limit))
{
    $_offset = $this->offset;
    $limit = "LIMIT {$_limit} {$_offset}";
}
```

`return sprintf($template, $this->from, $where, $limit);`

```php
}
```

`public function save($data)`

```php
{
    $isInsert = sizeof($this->_where) == 0;
    if ($isInsert)
    {
        $sql = $this->_buildInsert($data);
    }
    else
    {
        $sql = $this->_buildUpdate($data);
    }
    $result = $this->_connector->execute($sql);
    if ($result === false)
    {
        throw new Exception\Sql();
    }
    if ($isInsert)
    {
        return $this->_connector->lastInsertId;
    }
    return 0;
}
```

`public function delete()`

```php
{
    $sql = $this->_buildDelete();
    $result = $this->_connector->execute($sql);
    if ($result === false)
    {
        throw new Exception\Sql();
    }
    return $this->_connector->affectedRows;
}
```

`public function from($from, $fields = array("*"))`

```php
{
    if (empty($from))
    {
        throw new Exception\Argument("Invalid argument");
    }
    $this->_from = $from;
    if ($fields)
    {
        $this->_fields[$from] = $fields;
    }
    return $this;
}
```

`public function join($join, $on, $fields = array())`

```php
{
    if (empty($join))
    {
        throw new Exception\Argument("Invalid argument");
    }
    if (empty($on))
    {
        throw new Exception\Argument("Invalid argument");
    }
    $this->_fields += array($join => $fields);
    $this->_join[] = "JOIN {$join} ON {$on}";
    return $this;
}
```

`public function limit($limit, $page = 1)`

```php
{
    if (empty($limit))
    {
        throw new Exception\Argument("Invalid argument");
    }
    $this->_limit = $limit;
    $this->_offset = $limit * ($page - 1);
    return $this;
}
```

`public function order($order, $direction = "asc")`

```php
{
    if (empty($order))
    {
        throw new Exception\Argument("Invalid argument");
    }
    $this->_order = $order;
    $this->_direction = $direction;
    return $this;
}
```

`public function where()`

```php
{
    $arguments = func_get_args();
    if (sizeof($arguments) < 1)
    {
        throw new Exception\Argument("Invalid argument");
    }
    $arguments[0] = preg_replace("#\?#", "%s", $arguments[0]);
    foreach (array_slice($arguments, 1, null, true) as $i => $parameter)
    {
        $arguments[$i] = $this->_quote($arguments[$i]);
    }
    $this->_where[] = call_user_func_array("sprintf", $arguments);
    return $this;
}
```

`public function first()`

```php
{
    $limit = $this->_limit;
    $offset = $this->_offset;
    $this->limit(1);
    $all = $this->all();
    $first = ArrayMethods::first($all);
    if ($limit)
    {
        $this->_limit = $limit;
    }
    if ($offset)
    {
        $this->_offset = $offset;
    }
    return $first;
}
```

`public function count()`

```php
{
    $limit = $this->limit;
    $offset = $this->offset;
    $fields = $this->fields;
    $this->_fields = array($this->from => array("COUNT(1)" => "rows"));
    $this->limit(1);
    $row = $this->first();
    $this->_fields = $fields;
    if ($fields)
    {
        $this->_fields = $fields;
    }
    if ($limit)
    {
        $this->_limit = $limit;
    }
    if ($offset)
    {
        $this->_offset = $offset;
    }
    return $row["rows"];
}
```

```php
}
}
```

如果你计划使用非关系型或非 SQL 的数据库引擎，可以自由地设计一个相对空的 `Query` 类，并搭配一个完整的 `Query` 子类。

**注意**

转义查询值的过程非常重要。它通过确保特殊查询字符被安全地转义为其文本字符串表示形式，从而防止 SQL 注入攻击。SQL 注入攻击是一种相当常见的安全问题，你可以在 [`en.wikipedia.org/wiki/SQL_injection`](http://en.wikipedia.org/wiki/SQL_injection) 上了解更多相关信息。

### 问题

1. 在本章中，我们构建了一个由三部分组成的数据库引擎：工厂、连接器和驱动程序。为什么这比使用单一类或仅使用工厂-驱动组合更可取？

2. `Connector` 类比 `Connector\Mysql` 类简单得多，然而 `Query\Mysql` 类比 `Query` 类简单得多。这是为什么？

3. 为什么 `Query` 类需要用查询值替换 `?` 字符？

### 答案

1. 如果我们计划拥有多个连接器或查询类，那么工厂模式是一个好主意。


