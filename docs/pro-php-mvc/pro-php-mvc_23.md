# 注意

**注意** 默认数据库引擎设置为 `InnoDB`，而非 `MyISAM`。这两种数据库存储引擎之间存在若干差异，您在规划应用程序时需要加以考虑。您可以在此处了解更多关于 `InnoDB` 与 `MyISAM` 差异的信息：[`en.wikipedia.org/wiki/Comparison_of_MySQL_database_engines`](http://www.en.wikipedia.org/wiki/Comparison_of_MySQL_database_engines)。我发现通常选择 `InnoDB` 更符合需求，这也是将其设为合理默认值的原因。

因此，您会注意到，我们的 MySQL 连接器类虽然允许我们执行 SQL 字符串，并提供最近执行的 SQL 查询的各种有用信息，但实际上并未编写任何 SQL 语句！

这正是接下来要讨论的 `Query` 类的工作。

#### 查询

在前面的代码清单中，还有一件重要的事情需要注意：`query()` 方法返回的是 `Database\Query\Mysql` 类的一个实例。我们大部分数据库代码都将编写在该类中。`Query` 类负责编写特定于供应商的数据库代码。就 MySQL 而言，包括 `UPDATE`、`DELETE` 和 `SELECT` 查询。它从清单 9-6 所示的代码开始，这段代码看起来应该很熟悉。

**清单 9-6.** `Database\Query` 类

```
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
    }
}
```

在这里，我们可以看到一些可调整的属性以及异常生成方法的重写。这些属性在实际查询生成之前保存查询参数。生成数据库查询的方法会使用这些属性中的数据，因此它们被设置为受保护且大多为只读类型。我们不能让开发者在生成过程中随意修改其内容。正如我们看到的第二个示例，我们应该能够用任何类型的数据替换 `?` 字符，因此我们还需要一种方法来根据 MySQL 的期望对输入数据进行正确的引用。请看清单 9-7。

**清单 9-7.** `_quote()` 方法

```
namespace Framework\Database
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Database\Exception as Exception;

    class Query extends Base
    {
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
    }
}
```

受保护的 `_quote()` 方法用于将传入的 `$value` 包裹在适用的引号中，以便能够以语法正确的形式将其添加到相应的查询中。它包含了处理不同值类型（如字符串、数组和布尔值）的逻辑。我们还需要构建用于修改查询的便捷方法，例如 `from()`、`limit()` 和 `join()`，如清单 9-8 所示。

**清单 9-8.** Query 的糖方法

```
namespace Framework\Database
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Database\Exception as Exception;
```


```php
class Query extends Base
{
    public function from($from, $fields = array("*"))
    {
        if (empty($from))
        {
            throw new Exception\Argument("无效参数");
        }
        $this->_from = $from;
        if ($fields)
        {
            $this->_fields[$from] = $fields;
        }
        return $this;
    }

    public function join($join, $on, $fields = array())
    {
        if (empty($join))
        {
            throw new Exception\Argument("无效参数");
        }
        if (empty($on))
        {
            throw new Exception\Argument("无效参数");
        }
        $this->_fields += array($join => $fields);
        $this->_join[] = "JOIN {$join} ON {$on}";
        return $this;
    }

    public function limit($limit, $page = 1)
    {
        if (empty($limit))
        {
            throw new Exception\Argument("无效参数");
        }
        $this->_limit = $limit;
        $this->_offset = $limit * ($page - 1);
        return $this;
    }

    public function order($order, $direction = "asc")
    {
        if (empty($order))
        {
            throw new Exception\Argument("无效参数");
        }
        $this->_order = $order;
        $this->_direction = $direction;
        return $this;
    }

    public function where()
    {
        $arguments = func_get_args();
        if (sizeof($arguments) < 1)
        {
            throw new Exception\Argument("无效参数");
        }
        $arguments[0] = preg_replace("#\?#", "%s", $arguments[0]);
        foreach (array_slice($arguments, 1, null, true) as $i => $parameter)
        {
            $arguments[$i] = $this->_quote($arguments[$i]);
        }
        $this->_where[] = call_user_func_array("sprintf", $arguments);
        return $this;
    }
}
```

所有这些方法都返回类实例的引用，因此可以链式调用。`from()`方法用于指定要读取或写入数据（对于 INSERT、UPDATE 和 DELETE 操作而言）的表。它还接受第二个参数，用于指定你要查询的字段。虽然可以指定这些字段，但即使你希望执行数据修改查询，当它们不适用于所请求的查询时也会被忽略。

`join()`方法用于指定跨表的连接。它只允许自然连接（NATURAL JOIN），这在大多数情况下已经足够。我们再次强调，只关心简洁的代码，能在大多数情况下满足需求即可。如果我们需要复杂的查询，直接编写纯 SQL 语句，甚至创建存储过程或视图来处理可能更为合适。

`join()`方法的第二个参数是连接所依赖的字段。最后一个（可选）参数与`from()`方法的第二个参数类似，允许你指定从被连接表中返回的字段。稍后我们将看到的`_buildSelect()`方法能够处理`$fields`参数，无论是简单的字段名，还是带别名的字段名。这些方法足够智能，可以区分`$fields = array('field1')`和`$fields = array('field1' => 'alias1')`之间的差异。

`where()`方法是这些方法中最棘手的，因为它允许可变长度的参数。它会接受一个格式为`'foo = ? and bar = ? and baz = ?'`的字符串，并接受另外三个参数来替换这些`?`字符。实际上，它是在运行 PHP 的`sprintf`函数，因此我们不需要处理任何标记化操作；但它还会使用我们之前看到的`_quote()`方法对第二、三、四（等）位置传入的值进行转义，从而保护我们的数据库数据免受注入攻击。

`order()`方法用于指定按哪个字段排序以及排序方向。它只处理按单个字段排序，这是为了适应大多数情况；如果第二个（可选）`$direction`参数未指定，则默认排序方向为`asc`。

`limit()`方法用于指定一次返回多少行，以及从哪一页开始显示结果。这比直接提供偏移量稍微方便一些，因为你不必考虑单个行的位置，而是考虑行数的分页（这正是 SQL 中 LIMIT 语句的典型用途）。`page`参数是可选的。


这些方法都接受某种输入，该输入稍后将在 SQL 查询的构建中使用。

它们几乎没有输入验证，可能更需要合适的异常消息，但目前我们不必担心这个问题。这些方法命名的目的是允许流畅的查询创建（适用于我们可能遇到的大多数情况）。当你可以用通俗语言阅读查询时，调试起来会容易得多。

大多数关系型 SQL 查询是 `INSERT`、`UPDATE`、`DELETE` 或 `SELECT` 查询。我们将创建方法来处理这些类型的查询，从 `SELECT` 查询开始，如 **清单 9-9** 所示。

**清单 9-9.** `_buildSelect()` 方法

```php
namespace Framework\Database
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Database\Exception as Exception;
    
    class Query extends Base
    {
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
    }
}
```

这个方法是查询类的核心——特别是 `all()`、`first()` 和 `count()` 方法。它考虑了之前我们看到的那些受保护属性中存储的所有数据，并从零开始构建一个兼容 MySQL 的 SQL 查询。它声明了 `SELECT` 语句的模板，然后遍历所有存储的字段（来自 `from()` 和 `join()` 方法），并决定是否对它们进行别名处理。通过将它们连接成一个逗号分隔的字符串来整理它们。类似地，它还连接了 `$_join` 和 `$_where` 属性。接着，它创建格式正确的 `ORDER BY` 和 `LIMIT` 子句，最后通过 `sprintf` 执行所有操作，生成一个有效的 SQL 查询。

构建 `INSERT`、`UPDATE` 和 `DELETE` 查询的方法比 `_buildSelect()` 方法简单一些。请看 **清单 9-10**。

**清单 9-10.** `_buildInsert()`、`_buildUpdate()` 和 `_buildDelete()` 方法

```php
namespace Framework\Database
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Database\Exception as Exception;
    
    class Query extends Base
    {
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
            $template = "DELETE FROM %s %s %s";
            
            $_where = $this->where;
            if (!empty($_where))
            {
                // 此处代码不完整，原文已截断
            }
        }
    }
}
```



```php
$joined = join(", ", $_where);

$where = "WHERE {$joined}";

}

$_limit = $this->limit;

if (!empty($_limit))

{

$_offset = $this->offset;

$limit = "LIMIT {$_limit} {$_offset}";

}

return sprintf($template, $this->from, $where, $limit);
```

相比之下，`_buildInsert()`、`_buildUpdate()` 和 `_buildDelete()` 方法较为简单。它们同样用于构建 SQL 查询，但目的不同，且使用不同的数据。

`_buildInsert()` 方法定义了有效 `INSERT` 查询的模板，然后将传入的 `$data` 拆分为两个数组：`$fields` 和 `$values`。接着合并这些数组，最后通过 `sprintf` 处理结果。

`_buildUpdate()` 方法的功能大体相同，只是它将字段和值合并为一个数组（这与 MySQL `UPDATE` 查询的格式一致）。此外，它还需要额外处理 `WHERE` 子句，因为大多数更新操作针对的是特定的数据库行集合。如果 `UPDATE` 查询未能指定有效的 `WHERE` 子句，将导致表中所有行都被更新为相同的数据。

它还会考虑 `LIMIT` 子句，从而可以将更新限制在匹配 `WHERE` 子句的有限记录数量内。为防止 `WHERE` 子句缺失或筛选的行数超出预期，对表更新设置一个限制总是好的做法。

`_buildDelete()` 方法执行 `_buildUpdate()` 方法的所有功能，但不执行 `_buildInsert()` 方法的任何功能。这是因为除了 `WHERE` 子句的目标之外，无需考虑任何行数据。同样建议在删除行时始终使用 `LIMIT` 子句，以免删除的行数超出预期。

这四种方法完成了大部分所需工作，但我们可以通过清单 9-11 所示的方法简化它们的使用。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 9 章 ■ 数据库

### 清单 9-11 便捷方法

```php
namespace Framework\Database
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Database\Exception as Exception;

    class Query extends Base
    {
        public function save($data)
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

        public function delete()
        {
            $sql = $this->_buildDelete();
            $result = $this->_connector->execute($sql);

            if ($result === false)
            {
                throw new Exception\Sql();
            }

            return $this->_connector->affectedRows;
        }
    }
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 9 章 ■ 数据库

`save()` 方法通过检查是否对此查询对象调用了 `where()` 方法，来判断需要哪种类型的查询。如果调用了，则假定该方法是用于指定行 ID（或其他行特定条件），并相应调用 `_buildUpdate()` 方法。如果 `INSERT`/`UPDATE` 查询执行失败，将抛出 `Database\Exception\Sql` 异常。若查询成功执行且为 `INSERT` 查询，则返回最后插入的 ID。如果执行的是 `UPDATE` 查询，则返回 `0`（以此表示操作成功）。

> **注意：** 我们不会通过 `save()` 方法返回受影响的行数或布尔值——因为这些值会使该函数的结果产生歧义。你可以通过调用数据库连接器的 `getAffectedRows()` 方法，获取 `UPDATE` 查询的受影响行数。

`delete()` 方法只需调用 `_buildDelete()` 方法并执行其结果。如果查询无法执行，则抛出 `Database\Exception\Sql` 异常；否则返回受影响的行数。我们还需要最后三个公共方法，它们分别返回匹配查询的所有行、匹配查询的第一行，以及查询将返回的总行数。清单 9-12 和 9-13 演示了这一点。



