# 将连接器与查询类组合是个好主意，因为它们处理不同的事务。`Connector`类是 PHP 与 MySQL 服务器之间的中介，而`Query`类负责构建 SQL 查询。将它们合并为一个类会严重限制库的功能。

通过 PHP 访问的每个数据库引擎都需要独特的方法和类名。这意味着几乎无法抽象出适用于所有数据库引擎的通用逻辑。而关系型 SQL 则并非特定数据库引擎的专属，因此可以将其抽象化，以便在多种关系型 SQL 数据库引擎中使用。

这是我们设置的一项安全措施，用于验证提供给 MySQL 的查询参数。我们也可以使用基本的字符串替换，但这样做安全性较低。

### 练习题

1. 本章主要讲解 MySQL。但我们构建的架构也可用于其他关系型 SQL 数据库引擎，因此请尝试为另一关系型 SQL 数据库引擎创建相应的连接器-驱动组合。

2. 使用`Driver\Mysql`类生成的`WHERE`子句不支持`OR`、`WHERE`或`LIKE`子句。请尝试为这些条件创建相应的方法。

---

## 第 10 章：模型

到目前为止，我们创建了构成框架基础的类和库，但并未特别关注应用程序将如何使用它们。这一切即将改变。

正如我之前提到的，如果没有可供计算的数据，计算本身就毫无意义。你将要参与的多数项目都有一个共同点：它们需要存储、操作、返回和显示数据。通过使用数据库，你可以实现这些功能。

在第 9 章中，我们学习了如何创建可扩展的数据库库，并编写了一对 MySQL 连接器/查询类。直接与数据库对话是完成任务的一种方式，但在控制器中直接这样做并不理想。这只是使用模型的原因之一。

### 目标

- 我们需要理解什么是模型。
- 我们需要构建模型类，以便轻松处理通用且重复的任务。

### 理念

模型使我们能够隔离所有直接的数据库通信，以及大多数与第三方 Web 服务的通信。本章中我们将构建的模型，将提供一个简单的接口，用于对数据库执行变更操作。

> **注意** 本章定义模型的方式有时被称为对象关系映射（ORM）。ORM 库在两个与数据相关的系统之间创建了一个不透明的通信层。你可以在[`en.wikipedia.org/wiki/Object-relational_mapping`](http://en.wikipedia.org/wiki/Object-relational_mapping)了解更多关于 ORM 的信息。

理解模型不限于数据库工作很重要。模型可以连接到任意数量的第三方数据服务，并为我们的控制器提供简单的接口。我们只是恰好将模型聚焦到 ORM 范式上。

同样，我们也可能需要构建一个连接到 Flickr 下载照片的模型，或者一个允许我们从 Amazon S3 等云计算网络上传/下载文件的模型。模型旨在包含所有未直接呈现在视图中、或未被控制器用于连接视图和模型的资源。

### 实现

数据库引擎可以存储多种不同类型的数据。对于我们的模型，我们只需要以下几种。

- **自动编号**  
自动编号字段会生成自动递增的数值，通常用于标识字段。

- **文本**  
文本字段对应`varchar`或`text`数据类型，具体取决于所需的字段长度。

- **整数**  
整数类型的默认字段长度为 11。

- **小数**  
小数字段包含浮点数值。

- **布尔值**  
布尔字段实际上是`tinyint`字段。当为其赋予`true`/`false`时，布尔值会被转换为整数值。

- **日期时间**



我们之所以选择在 ORM 中支持如此小的数据类型子集，是因为这可以保持简单。我们也可以确信绝大多数数据库引擎都支持这些数据类型。我们如何在模型中使用这些数据类型？清单 10-1 中的示例展示了一种为我们模型所需的不同数据类型进行定义的方式。

***清单 10-1.*** User 模型

```
class User extends Framework\Model
{

    /**
     * @column
     * @readwrite
     * @primary
     * @type autonumber
     */
    protected $_id;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     */
    protected $_first;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     */
    protected $_last;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     */
    protected $_email;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     */
    protected $_password;

    /**
     * @column
     * @readwrite
     * @type text
     */
    protected $_notes;

    /**
     * @column
     * @readwrite
     * @type boolean
     * @index
     */
    protected $_live;

    /**
     * @column
     * @readwrite
     * @type boolean
     * @index
     */
    protected $_deleted;

    /**
     * @column
     * @readwrite
     * @type datetime
     */
    protected $_created;

    /**
     * @column
     * @readwrite
     * @type datetime
     */
    protected $_modified;

}
```

关于此示例模型，我们首先注意到它只包含少数几个属性，并且非常容易理解。每个受保护的属性都包含了我们之前见过的`@readwrite`标记。这意味着我们将能够调用诸如`setFirst()`和`getCreated()`这样的 getter/setter 方法，而无需在我们的`Model`类中定义它们。

新属性才是我们真正感兴趣的。每个类属性（它对应数据库中的一个列）都通过`@column`标记来标识。我们的模型初始化代码将忽略所有没有这个标记的类属性。`$_id`类属性有一个`@primary`标记，用于将其标识为主键列。`$_email`、`$_password`、`$_live`和`$_deleted`属性都有`@index`标记，表明它们应该在数据库表中被索引。

其余的标记是`@type`和`@length`。这些标记指明了类属性应该具有的数据类型，以及字段长度（我们只会在文本字段的情况下使用它，以决定它们应该是`varchar`还是`text`）。

如果这个结构要对我们有用，它需要转换为实际的数据库表。我们的数据库/模型层需要将我们创建的列转换为底层的 SQL，以便将其提交到数据库。当我们完成时，它应该类似于清单 10-2 中所示的 SQL。

***清单 10-2.*** User 模型的 MySQL 表表示

```
CREATE TABLE 'user' (
    'id' int(11) NOT NULL AUTO_INCREMENT,
    'first' varchar(100) DEFAULT NULL,
    'last' varchar(100) DEFAULT NULL,
    'email' varchar(100) DEFAULT NULL,
    'password' varchar(100) DEFAULT NULL,
    'notes' text,
    'live' tinyint(4) DEFAULT NULL,
    'deleted' tinyint(4) DEFAULT NULL,
    'created' datetime DEFAULT NULL,
    'modified' datetime DEFAULT NULL,
    PRIMARY KEY ('id'),
    KEY 'email' ('email'),
    KEY 'password' ('password'),
    KEY 'live' ('live'),
    KEY 'deleted' ('deleted')
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

> **注意**：密码绝不应以纯文本形式存储。为了保持简单和专注，我们在此忽略了这个最佳实践。你应该花时间学习如何加密或哈希密码，以确保它们不会被窃取并用于恶意目的。

### 构建 SQL

清单 10-2 中的 SQL 是我们期望模型在数据库中呈现的样子。它说明了我们的`Model`类在我们概述的简单类型（`text`，`integer`，`Boolean`等）和数据库中的实际字段类型之间进行的数据类型转换。因此，让我们开始创建我们的`Model`类，如清单 10-3 所示。

***清单 10-3.*** Model 类

```
namespace Framework
{

    use Framework\Base as Base;
    use Framework\Registry as Registry;
```



```php
use Framework\Inspector as Inspector;
use Framework\StringMethods as StringMethods;
use Framework\Model\Exception as Exception;

class Model extends Base
{
    /**
     * @readwrite
     */
    protected $_table;

    /**
     * @readwrite
     */
    protected $_connector;

    /**
     * @read
     */
    protected $_types = array(
        "autonumber",
        "text",
        "integer",
        "decimal",
        "boolean",
        "datetime"
    );

    protected $_columns;
    protected $_primary;

    public function _getExceptionForImplementation($method)
    {
        return new Exception\Implementation("{$method} method not implemented");
    }
}
```

我们的 `Model` 类起步相当简单，定义了需要 getter/setter 的属性，以及覆写异常生成的方法。`$_types` 属性保存了模型所能理解的简单数据类型，并在内部和外部用于验证。在我们能为模型/数据库表创建 SQL 之前，需要在 `StringMethods` 类中创建一些工具方法，如清单 10-4 所示。

**清单 10-4.** `StringMethods` 的词形变化方法

```php
namespace Framework
{
    class StringMethods
    {
        private static $_singular = array(
            "(matr)ices$" => "\\1ix",
            "(vert|ind)ices$" => "\\1ex",
            "^(ox)en" => "\\1",
            "(alias)es$" => "\\1",
            // [www.it-ebooks.info](http://www.it-ebooks.info/)
            // 第 10 章 ■ 模型
            "([octop|vir])i$" => "\\1us",
            "(cris|ax|test)es$" => "\\1is",
            "(shoe)s$" => "\\1",
            "(o)es$" => "\\1",
            "(bus|campus)es$" => "\\1",
            "([m|l])ice$" => "\\1ouse",
            "(x|ch|ss|sh)es$" => "\\1",
            "(m)ovies$" => "\\1\\2ovie",
            "(s)eries$" => "\\1\\2eries",
            "([^aeiouy]|qu)ies$" => "\\1y",
            "([lr])ves$" => "\\1f",
            "(tive)s$" => "\\1",
            "(hive)s$" => "\\1",
            "([^f])ves$" => "\\1fe",
            "(^analy)ses$" => "\\1sis",
            "((a)naly|(b)a|(d)iagno|(p)arenthe|(p)rogno|(s)ynop|(t)he)ses$" => "\\1\\2sis",
            "([ti])a$" => "\\1um",
            "(p)eople$" => "\\1\\2erson",
            "(m)en$" => "\\1an",
            "(s)tatuses$" => "\\1\\2tatus",
            "(c)hildren$" => "\\1\\2hild",
            "(n)ews$" => "\\1\\2ews",
            "([^u])s$" => "\\1"
        );

        private static $_plural = array(
            "^(ox)$" => "\\1\\2en",
            "([m|l])ouse$" => "\\1ice",
            "(matr|vert|ind)ix|ex$" => "\\1ices",
            "(x|ch|ss|sh)$" => "\\1es",
            "([^aeiouy]|qu)y$" => "\\1ies",
            "(hive)$" => "\\1s",
            "(?:([^f])fe|([lr])f)$" => "\\1\\2ves",
            "sis$" => "ses",
            "([ti])um$" => "\\1a",
            "(p)erson$" => "\\1eople",
            "(m)an$" => "\\1en",
            "(c)hild$" => "\\1hildren",
            "(buffal|tomat)o$" => "\\1\\2oes",
            "(bu|campu)s$" => "\\1\\2ses",
            "(alias|status|virus)" => "\\1es",
            "(octop)us$" => "\\1i",
            "(ax|cris|test)is$" => "\\1es",
            "s$" => "s",
            "$" => "s"
        );

        public static function singular($string)
        {
            $result = $string;
            foreach (self::$_singular as $rule => $replacement)
            {
                $rule = self::_normalize($rule);
                // [www.it-ebooks.info](http://www.it-ebooks.info/)
                // 第 10 章 ■ 模型
                if (preg_match($rule, $string))
                {
                    $result = preg_replace($rule, $replacement, $string);
                    break;
                }
            }
            return $result;
        }

        function plural($string)
        {
            $result = $string;
            foreach (self::$_plural as $rule => $replacement)
            {
                $rule = self::_normalize($rule);
                if (preg_match($rule, $string))
                {
                    $result = preg_replace($rule, $replacement, $string);
                    break;
                }
            }
            return $result;
        }
    }
}
```

我们还需要向之前创建的 `Model` 类中添加一些方法，如清单 10-5 所示。

**清单 10-5.** 覆写 Getter 方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public function getTable()
        {
            if (empty($this->_table))
            {
                $this->_table = strtolower(StringMethods::singular(get_class($this)));
            }
            return $this->_table;
        }

        public function getConnector()
        {
            if (empty($this->_connector))
            {
                // [www.it-ebooks.info](http://www.it-ebooks.info/)
                // 第 10 章 ■ 模型
                $database = Registry::get("database");
                if (!$database)
                {
                    throw new Exception\Connector("没有可用的连接器");
                }
                $this->_connector = $database->initialize();
            }
            return $this->_connector;
        }
    }
}
```

我们给 `StringMethods` 类添加了两个新方法（称为*词形变化*方法）。这些方法使用一系列正则表达式将字符串转换为单数或复数形式。接下来，我们在 `Model` 类中覆写了 `$_table` 和 `$_connector` 属性的 getter 方法。对于 `getTable()` 方法，我们希望返回用户定义的表名，否则默认使用当前模型类名的单数形式（使用 PHP 的 `get_class()` 方法，以及我们添加到 `StringMethods` 类中的一个新词形变化方法）。

我们覆写 `getConnector()` 方法，以便返回 `$_connector` 属性的内容、存储在 `Registry` 类中的连接器实例，或者抛出 `Model\Exception\Connector` 异常。这是我们第一次从 `Registry` 类中检索内容，我们在这里这样做，是因为此时数据库连接很可能已经被初始化。

我们还需要创建一个方法，该方法返回按顺序排列的列及其所有元数据数组，以便我们构建 SQL。请看清单 10-6，了解我们如何实现这一点。

**清单 10-6.** `getColumns()` 方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public function getColumns()
        {
            if (empty($_columns))
            {
                $primaries = 0;
                $columns = array();
                $class = get_class($this);
                $types = $this->types;
                $inspector = new Inspector($this);
                $properties = $inspector->getClassProperties();
                $first = function($array, $key)
                {
                    if (!empty($array[$key]) && sizeof($array[$key]) == 1)
                    {
                        // [www.it-ebooks.info](http://www.it-ebooks.info/)
                        // 第 10 章 ■ 模型
                        return $array[$key][0];
                    }
                    return null;
                };

                foreach ($properties as $property)
                {
                    $propertyMeta = $inspector->getPropertyMeta($property);
                    if (!empty($propertyMeta["@column"]))
                    {
                        $name = preg_replace("#^_#", "", $property);
                        $primary = !empty($propertyMeta["@primary"]);
                        $type = $first($propertyMeta, "@type");
                        $length = $first($propertyMeta, "@length");
                        $index = !empty($propertyMeta["@index"]);
                        $readwrite = !empty($propertyMeta["@readwrite"]);
                        $read = !empty($propertyMeta["@read"]) || $readwrite;
                        $write = !empty($propertyMeta["@write"]) || $readwrite;
                        $validate = !empty($propertyMeta["@validate"]) 
                            ? $propertyMeta["@validate"] : false;
                        $label = $first($propertyMeta, "@label");

                        if (!in_array($type, $types))
                        {
                            throw new Exception\Type("{$type} 不是有效的类型");
                        }

                        if ($primary)
                        {
                            $primaries++;
                        }

                        $columns[$name] = array(
                            "raw" => $property,
                            "name" => $name,
                            "primary" => $primary,
                            "type" => $type,
                            "length" => $length,
                            "index" => $index,
                            "read" => $read,
                            "write" => $write,
                            "validate" => $validate,
                            "label" => $label
                        );
                    }
                }

                if ($primaries !== 1)
                {
                    throw new Exception\Primary("{$class} 必须且只能有一个 @primary 列");
                }

                // [www.it-ebooks.info](http://www.it-ebooks.info/)
                // 第 10 章 ■ 模型
                $this->_columns = $columns;
            }
            return $this->_columns;
        }
    }
}
```

`getColumns()` 方法可能看起来像是上一章构建的模板库中的东西，但实际上它相当简单。它创建了一个 `Inspector` 实例和一个工具函数（`$first`），用于返回元数据数组中的第一个元素。接着，它遍历模型中的所有属性，并筛选出所有带有 `@column` 标记的属性。此时，任何其他属性都会被忽略。



检查`column`的`@type`标志是否有效，若无效则抛出`Model\Exception\Type`异常。如果列类型有效，则将其添加到`$_columns`属性中。每个有效的`$primary`列都会使`$primaries`变量递增，该方法最后会检查该变量，确保只定义了一个主键列。本质上，该方法接收`User`模型定义，并返回一个关联数组形式的列数据。我们还可以创建两个便捷方法，从该关联数组中返回单独的列，如清单 10-7 所示。

***清单 10-7.*** 列获取器

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public function getColumn($name)
        {
            if (!empty($this->_columns[$name]))
            {
                return $this->_columns[$name];
            }
            return null;
        }

        public function getPrimaryColumn()
        {
            if (!isset($this->_primary))
            {
                $primary;
                foreach ($this->columns as $column)
                {
                    if ($column["primary"])
                    {
                        $primary = $column;
                        break;
                    }
                }
                $this->_primary = $primary;
            }
            return $this->_primary;
        }
    }
}
```

`getColumn()`方法按名称返回列。你可能注意到，列类属性假定以下划线（`_`）字符开头。`getColumn()`方法延续了这一假定，它检查不带`_`字符的列名。当声明为列属性时，列名类似`_firstName`，但在任何公开的获取器/设置器/方法中引用时，则类似`setFirstName`/`firstName`。

`getPrimaryColumn()`方法遍历所有列，返回标记为主键的列。

现在我们有办法描述和迭代模型属性（进而描述数据库表），可以开始构建最终生成物理表的 SQL 语句。由于表使用 SQL 语法创建，且该语法通常与具体数据库相关，我们将在连接器类上创建一个方法，用于将结构同步到数据库。此方法会将列数组转换为所需的 SQL 语句，如清单 10-8 所示。

***清单 10-8.*** `Database\Connector\Mysql`的`sync()`方法

```php
namespace Framework\Database\Connector
{
    use Framework\Database as Database;
    use Framework\Database\Exception as Exception;

    class Mysql extends Database\Connector
    {
        public function sync($model)
        {
            $lines = array();
            $indices = array();
            $columns = $model->columns;
            $template = "CREATE TABLE '%s' (\n%s,\n%s\n) ENGINE=%s DEFAULT CHARSET=%s;";
            foreach ($columns as $column)
            {
                $raw = $column["raw"];
                $name = $column["name"];
                $type = $column["type"];
                $length = $column["length"];

                if ($column["primary"])
                {
                    $indices[] = "PRIMARY KEY ('{$name}')";
                }

                if ($column["index"])
                {
                    $indices[] = "KEY '{$name}' ('{$name}')";
                }

                switch ($type)
                {
                    case "autonumber":
                    {
                        $lines[] = "'{$name}' int(11) NOT NULL AUTO_INCREMENT";
                        break;
                    }
                    case "text":
                    {
                        if ($length !== null && $length <= 255)
                        {
                            $lines[] = "'{$name}' varchar({$length}) DEFAULT NULL";
                        }
                        else
                        {
                            $lines[] = "'{$name}' text";
                        }
                        break;
                    }
                    case "integer":
                    {
                        $lines[] = "'{$name}' int(11) DEFAULT NULL";
                        break;
                    }
                    case "decimal":
                    {
                        $lines[] = "'{$name}' float DEFAULT NULL";
                        break;
                    }
                    case "boolean":
                    {
                        $lines[] = "'{$name}' tinyint(4) DEFAULT NULL";
                        break;
                    }
                    case "datetime":
                    {
                        $lines[] = "'{$name}' datetime DEFAULT NULL";
                        break;
                    }
                }
            }

            $table = $model->table;
            $sql = sprintf(
                $template,
                $table,
                join(",\n", $lines),
                join(",\n", $indices),
                $this->_engine,
                $this->_charset
            );

            $result = $this->execute("DROP TABLE IF EXISTS {$table};");
            if ($result === false)
            {
                $error = $this->lastError;
                throw new Exception\Sql("There was an error in the query: {$error}");
            }

            $result = $this->execute($sql);
            if ($result === false)
            {
                $error = $this->lastError;
            }
        }
    }
}
```


```php
throw new Exception\Sql("There was an error in the query: {$error}");
```

我们创建的 `sync()` 方法会将类/属性转换为有效的 SQL 查询语句，并最终生成物理数据库表。其实现过程是：首先通过调用模型的 `getColumns()` 方法获取列名列表，然后遍历这些列名，创建索引数组和字段字符串。

**注意：** 此 `CREATE TABLE` SQL 语句专为 MySQL 设计，可在 `Database\Connector` 类中找到。如果您使用的数据库引擎在执行最终 SQL 查询时遇到问题，可能需要在您的 `Database\Connector` 子类中重写 `sync()` 方法，为其提供正确的 SQL 查询语法。

在创建完所有字段字符串后，它们会被拼接在一起（连同索引一起），并应用到 `CREATE TABLE` 的 `$template` 字符串中。系统会先创建并执行一条 `DROP TABLE` 语句来清理旧表，然后再执行最终的 SQL 查询（用于创建新表）。任何 SQL 错误都会引发 `Database\Exception\Sql` 异常。我们可以在代码清单 10-9 中看到 `sync()` 方法的使用示例。

**代码清单 10-9.** `sync()` 方法的使用示例

```php
$database = new Database(array(
    "type" => "mysql",
    "options" => array(
        "host" => "localhost",
        "username" => "prophpmvc",
        "password" => "prophpmvc",
        "schema" => "prophpmvc"
    )
));

$database = $database->initialize()->connect();

$user = new User(array(
    "connector" => $database
));

$database->sync($user);
```

在代码清单 10-9 中，我们首先创建了一个活动的数据库连接，并将其（作为连接器）分配给新的 `User` 模型实例。最后，我们在 `User` 模型实例上调用 `sync()` 方法，数据库表便创建完成了。由于我们在 `Registry` 类中检查了数据库键的存在性，我们也可以使用代码清单 10-10 所示的代码实现相同的结果。

**代码清单 10-10.** 替代连接器用法

```php
$database = new Database(array(
    "type" => "mysql",
    "options" => array(
        "host" => "localhost",
        "username" => "prophpmvc",
        "password" => "prophpmvc",
        "schema" => "prophpmvc"
    )
));

Registry::set("database", $database->initialize()->connect());

$database->sync(new User());
```

### 修改记录

现在我们有了可用的数据库表，接下来需要扩展模型以直接处理数据库操作，并为我们提供简单的接口。我们要实现的首要功能是：当提供了主键列的值时，模型能够加载对应的记录。为此，我们需要修改构造函数并添加 `load()` 方法，如代码清单 10-11 所示。

**代码清单 10-11.** 模型的 `__construct()` 和 `load()` 方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public function __construct($options = array())
        {
            parent::__construct($options);
            $this->load();
        }

        public function load()
        {
            $primary = $this->primaryColumn;
            $raw = $primary["raw"];
            $name = $primary["name"];

            if (!empty($this->$raw))
            {
                $previous = $this->connector
                    ->query()
                    ->from($this->table)
                    ->where("{$name} = ?", $this->$raw)
                    ->first();

                if ($previous == null)
                {
                    throw new Exception\Primary("Primary key value invalid");
                }

                foreach ($previous as $key => $value)
                {
                    $prop = "_{$key}";

                    if (!empty($previous->$key) && !isset($this->$prop))
                    {
                        $this->$key = $previous->$key;
                    }
                }
            }
        }
    }
}
```

回顾 `Base` 类，我们会想起传递给构造函数的关联数组会被应用于现有的 getter/setter 方法。这意味着我们可以通过向构造函数提供与主键列名匹配的正确键值对，以及对应现有记录的值，来加载已有的记录。

`load()` 方法极大地简化了记录检索过程。它确定模型的主键列并检查该值是否不为空，从而判断是否提供了主键，这为我们查找目标记录提供了可行的方法。如果主键类属性为空，我们则假定该模型实例用于创建新记录，不做进一步处理。

要加载数据库记录，我们首先获取当前模型的连接器，如果未找到连接器则停止执行。

我们根据主键列属性的值创建数据库查询来检索记录。如果未找到记录，则抛出 `Model\Exception\Primary` 异常。这种情况发生在提供了主键列值，但该值不对应数据库表中任何有效记录标识符时。

最后，我们遍历已加载记录的数据，仅设置在 `__construct()` 方法中未设置的属性值。这确保了模型初始化后不会丢失任何数据。模型另一个有用的方法是允许创建或更新记录，如代码清单 10-12 所示。

**代码清单 10-12.** `save()` 方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public function save()
        {
            $primary = $this->primaryColumn;
            $raw = $primary["raw"];
            $name = $primary["name"];

            $query = $this->connector
                ->query()
                ->from($this->table);

            if (!empty($this->$raw))
            {
                $query->where("{$name} = ?", $this->$raw);
            }

            $data = array();

            foreach ($this->columns as $key => $column)
            {
                if (!$column["read"])
                {
                    $prop = $column["raw"];
                    $data[$key] = $this->$prop;
                    continue;
                }

                if ($column != $this->primaryColumn && $column)
                {
                    $method = "get".ucfirst($key);
                    $data[$key] = $this->$method();
                    continue;
                }
            }

            $result = $query->save($data);

            if ($result > 0)
            {
                $this->$raw = $result;
            }

            return $result;
        }
    }
}
```

`save()` 方法创建一个查询实例，并定位与 `Model` 类相关的表。如果主键属性值不为空，则应用 `WHERE` 子句，并根据 `getColumns()` 方法返回的列构建数据数组。最后，调用查询实例的 `save()` 方法将数据提交到数据库。由于 `Database\Connector` 类会根据 `WHERE` 子句条件执行 `INSERT` 或 `UPDATE` 语句，此方法将根据主键属性是否有值来插入新记录或更新现有记录。我们最后需要的修改方法是 `delete()` 和 `deleteAll()` 方法，用于从数据库中删除记录。您可以在代码清单 10-13 中看到这些方法。

**代码清单 10-13.** `delete()` 和 `deleteAll()` 方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public function delete()
        {
            $primary = $this->primaryColumn;
            $raw = $primary["raw"];
            $name = $primary["name"];

            if (!empty($this->$raw))
            {
                return $this->connector
                    ->query()
                    ->from($this->table)
                    ->where("{$name} = ?", $this->$raw)
                    ->delete();
            }
        }

        public static function deleteAll($where = array())
        {
            $instance = new static();
            $query = $instance->connector
                ->query()
                ->from($instance->table);

            foreach ($where as $clause => $value)
            {
                $query->where($clause, $value);
            }

            return $query->delete();
        }
    }
}
```


`delete()`方法是我们模型中修改器方法中最简单的一个。它创建一个查询对象，仅当主键属性值不为空时执行，并调用查询的`delete()`方法。`deleteAll()`方法执行几乎相同的操作，区别在于它是静态调用的。到目前为止，我们只处理了单条记录的交互，但最好是能支持多条记录的操作，类似于我们数据库的`all()`、`first()`和`count()`方法。我们将创建的第一个方法是`all()`方法，如代码清单 10-14 所示。

**代码清单 10-14.** `all()`方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public static function all($where = array(), $fields = array("*"), 
            $order = null, $direction = null, $limit = null, $page = null)
        {
            $model = new static();
            return $model->_all($where, $fields, $order, $direction, $limit, $page);
        }

        protected function _all($where = array(), $fields = array("*"), 
            $order = null, $direction = null, $limit = null, $page = null)
        {
            $query = $this
                ->connector
                ->query()
                ->from($this->table, $fields);

            foreach ($where as $clause => $value)
            {
                $query->where($clause, $value);
            }

            if ($order != null)
            {
                $query->order($order, $direction);
            }

            if ($limit != null)
            {
                $query->limit($limit, $page);
            }

            $rows = array();
            $class = get_class($this);

            foreach ($query->all() as $row)
            {
                $rows[] = new $class($row);
            }

            return $rows;
        }
    }
}
```

`all()`方法是受保护的`_all()`方法的一个简单的静态包装器。`_all()`方法创建一个查询对象，并考虑各种过滤器和标志，以返回所有匹配的记录。

我们费心将实例方法包装在静态方法中的原因是，我们创建了一个上下文，其中模型实例等同于数据库记录。在此上下文中，多记录操作作为类方法更有意义。

`first()`方法与`all()`方法类似，它也是受保护实例方法的简单静态包装器。`_first()`方法仅返回第一个匹配的记录，如代码清单 10-15 所示。

**代码清单 10-15.** `first()`方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public static function first($where = array(), $fields = array("*"), 
            $order = null, $direction = null)
        {
            $model = new static();
            return $model->_first($where, $fields, $order, $direction);
        }

        protected function _first($where = array(), $fields = array("*"), 
            $order = null, $direction = null)
        {
            $query = $this
                ->connector
                ->query()
                ->from($this->table, $fields);

            foreach ($where as $clause => $value)
            {
                $query->where($clause, $value);
            }

            if ($order != null)
            {
                $query->order($order, $direction);
            }

            $first = $query->first();
            $class = get_class($this);

            if ($first)
            {
                return new $class($query->first());
            }

            return null;
        }
    }
}
```

`count()`方法与前面两个静态方法类似。`_count()`方法返回匹配记录的计数。代码清单 10-16 展示了`count()`方法。

**代码清单 10-16.** `count()`方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        public static function count($where = array())
        {
            $model = new static();
            return $model->_count($where);
        }

        protected function _count($where = array())
        {
            $query = $this
                ->connector
                ->query()
                ->from($this->table);

            foreach ($where as $clause => $value)
            {
                $query->where($clause, $value);
            }

            return $query->count();
        }
    }
}
```



所有代码就位后，我们来看看如何使用模型，如代码清单 10-17 所示。

**代码清单 10-17.** 模型使用示例

```
$database = new Database(array(
    "type" => "mysql",
    "options" => array(
        "host" => "localhost",
        "username" => "prophpmvc",
        "password" => "prophpmvc",
        "schema" => "prophpmvc"
    )
));
$database = $database->initialize();
$user = new User(array(
    "connector" => $database
));
$database->sync($user);
$elijah = new User(array(
    "connector" => $database,
    "first" => "Chris",
    "last" => "Pitt",
    "email" => "chris@example.com",
    "password" => "password",
    "live" => true,
    "deleted" => false,
    "created" => date("Y-m-d H:i:s"),
    "modified" => date("Y-m-d H:i:s")
));
$elijah->save();
$all = User::all(array(
    "last = ?" => "Pitt"
));
$elijah->delete();
```

这显然比直接使用 `Database\Connector` 或 `Database\Query` 要简单得多。它提供了一种通过 ORM 代码来检查数据库表的方式。我们可以使用简单的面向对象代码来查询或修改数据库记录。代码清单 10-18 展示了完整的 `Model` 类。

**代码清单 10-18.** `Model` 类

```
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\StringMethods as StringMethods;
    use Framework\Model\Exception as Exception;

    class Model extends Base
    {
        /**
         * @readwrite
         */
        protected $_table;

        /**
         * @readwrite
         */
        protected $_connector;

        /**
         * @read
         */
        protected $_types = array(
            "autonumber",
            "text",
            "integer",
            "decimal",
            "boolean",
            "datetime"
        );

        protected $_columns;
        protected $_primary;

        public function _getExceptionForImplementation($method)
        {
            return new Exception\Implementation("{$method} method not implemented");
        }

        public function __construct($options = array())
        {
            parent::__construct($options);
            $this->load();
        }

        public function load()
        {
            $primary = $this->primaryColumn;
            $raw = $primary["raw"];
            $name = $primary["name"];

            if (!empty($this->$raw))
            {
                $previous = $this->connector
                    ->query()
                    ->from($this->table)
                    ->where("{$name} = ?", $this->$raw)
                    ->first();

                if ($previous == null)
                {
                    throw new Exception\Primary("Primary key value invalid");
                }

                foreach ($previous as $key => $value)
                {
                    $prop = "_{$key}";
                    if (!empty($previous->$key) && !isset($this->$prop))
                    {
                        $this->$key = $previous->$key;
                    }
                }
            }
        }

        public function delete()
        {
            $primary = $this->primaryColumn;
            $raw = $primary["raw"];
            $name = $primary["name"];

            if (!empty($this->$raw))
            {
                return $this->connector
                    ->query()
                    ->from($this->table)
                    ->where("{$name} = ?", $this->$raw)
                    ->delete();
            }
        }

        public static function deleteAll($where = array())
        {
            $instance = new static();
            $query = $instance->connector
                ->query()
                ->from($instance->table);

            foreach ($where as $clause => $value)
            {
                $query->where($clause, $value);
            }

            return $query->delete();
        }

        public function save()
        {
            $primary = $this->primaryColumn;
            $raw = $primary["raw"];
            $name = $primary["name"];
            $query = $this->connector
                ->query()
                ->from($this->table);

            if (!empty($this->$raw))
            {
                $query->where("{$name} = ?", $this->$raw);
            }

            $data = array();

            foreach ($this->columns as $key => $column)
            {
                if (!$column["read"])
                {
                    $prop = $column["raw"];
                    $data[$key] = $this->$prop;
                    continue;
                }

                if ($column != $this->primaryColumn && $column)
                {
                    $method = "get".ucfirst($key);
                    $data[$key] = $this->$method();
                    continue;
                }
            }

            $result = $query->save($data);

            if ($result > 0)
            {
                $this->$raw = $result;
            }

            return $result;
        }

        public function getTable()
        {
            if (empty($this->_table))
            {
                $this->_table = strtolower(StringMethods::singular(get_class($this)));
            }

            return $this->_table;
        }

        public function getConnector()
        {
            if (empty($this->_connector))
            {
                $database = Registry::get("database");

                if (!$database)
                {
                    throw new Exception\Connector("No connector availible");
                }

                $this->_connector = $database->initialize();
            }

            return $this->_connector;
        }

        public function getColumns()
        {
            if (empty($_columns))
            {
                $primaries = 0;
                $columns = array();
                $class = get_class($this);
```

> 注意：原始文本中的 `[www.it-ebooks.info](http://www.it-ebooks.info/)` 和 `CHAPTER 10 ■ MOdEls` 属于无关的页眉页脚或广告信息，按排版要求已删除。


```php
$types = $this->types;

$inspector = new Inspector($this);

$properties = $inspector->getClassProperties();

$first = function($array, $key)

{

if (!empty($array[$key]) && sizeof($array[$key]) == 1)

{

return $array[$key][0];

}

return null;

};
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

