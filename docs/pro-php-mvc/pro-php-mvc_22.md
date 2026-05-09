# 第 9 章

### 数据库

大多数大型应用程序都有一个共同点：它们都是为与数据库协同工作而构建的。如果没有数据，世界上所有的计算都毫无意义。想想任何一项运动的规则。规则的存在是为了确保秩序，并判定谁是赢家。如果没有创造比分的球队，规则本身就毫无意义。

同样，除非涉及真实世界的数据，否则业务流程和计算也毫无意义。但为了获取、存储或修改这些数据，我们需要编写代码来连接数据库并执行各种必要的功能。这正是我们现在要创建的代码。

### 目标

- 我们需要构建一组类，以便与数据库进行交互。
- 这些类应允许我们通过链式方法调用来表达性地构建查询。

### 实现

出于时间考虑，我们将专注于 MySQL 数据库引擎。MySQL 是一个关系型 SQL 数据库引擎。如果你不熟悉 SQL，现在可能是时候补习一下了。请参考清单 9-1 中的示例。

***清单 9-1.*** 使用 PHP/MySQL 进行操作

```
/*
 * 假设有以下数据库表：
 *
 * CREATE TABLE 'users' (
 *   'id' int(11) NOT NULL AUTO_INCREMENT,
 *   'first_name' varchar(32) DEFAULT NULL,
 *   'last_name' varchar(32) DEFAULT NULL,
 *   'modified' datetime DEFAULT NULL,
 *   PRIMARY KEY ('id')
 * ) ENGINE = InnoDB DEFAULT CHARSET = utf8 AUTO_INCREMENT = 1 ;
 *
 * CREATE TABLE 'points' (
 *   'id' int(11) DEFAULT NULL,
 *   'points' int(11) DEFAULT NULL
 * ) ENGINE = InnoDB DEFAULT CHARSET = utf8;
 */

$database = new MySQLi(
    "localhost",
    "prophpmvc",
    "prophpmvc",
    "prophpmvc",
);

$rows = array();
$result = $database->query("SELECT first_name, last_name AS surname, points AS discount FROM users JOIN points ON users.id = points.id WHERE first_name = 'chris' ORDER BY last_name DESC LIMIT 100");

for ($i = 0; $i < $result->num_rows; $i++)
{
```

[www.it-ebooks.info](http://www.it-ebooks.info/)



`$rows[] = (object) $result->fetch_array(MYSQLI_ASSOC);`

这个示例包含两个部分。第一部分是用于执行 SQL 语句的 PHP 语句，第二部分则是该 SQL 语句本身。这是一个 PHP 与 MySQL 协同工作的非常基础的示例，但我们还可以更进一步，正如你在清单 9-2 中所见。

**清单 9-2.** 富有表现力的 SQL 生成

```
$all = $database->query()
    ->from("users", array(
        "name",
        "surname" => "last_name"
    ))
    ->join("points", "points.user_id = users.id", array(
        "points" => "rewards"
    ))
    ->where("name = ?", "chris")
    ->order("last_name", "desc")
    ->limit(100)
    ->all();
```

这段代码展示了我们所追求的那种流畅的查询创建方式。当我们完成本章的学习后，第二个示例应该能够实现与第一个示例完全相同的功能，但它更易于阅读和理解。如果我们把所需的代码构建得足够好，应该还能在不重写任何查询语句的情况下，切换到不同的数据库引擎！

我们构建数据库代码的平台必须足够健壮——它将处理大量数据。在大多数情况下，它需要易于使用，并且必须确保安全。与配置和缓存类似，我们会创建一个数据库工厂类，该类负责加载一个数据库驱动。这些数据库驱动与配置和缓存的驱动略有不同，因为它们被分成了两个部分：连接器和查询器。在深入探讨连接器和查询器之前，让我们先看看清单 9-3 中展示的数据库工厂。

**清单 9-3.** 数据库工厂类

```
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Database as Database;
    use Framework\Database\Exception as Exception;

    class Database extends Base
    {
        /**
         * @readwrite
         */
        protected $_type;

        /**
         * @readwrite
         */
        protected $_options;

        protected function _getExceptionForImplementation($method)
        {
            return new Exception\Implementation("{$method} method not implemented");
        }

        public function initialize()
        {
            if (!$this->type)
            {
                throw new Exception\Argument("Invalid type");
            }

            switch ($this->type)
            {
                case "mysql":
                {
                    return new Database\Connector\Mysql($this->options);
                    break;
                }
                default:
                {
                    throw new Exception\Argument("Invalid type");
                    break;
                }
            }
        }
    }
}
```

我们从 `Database` 工厂类开始我们的数据库工作。它会返回一个 `Database\Connector` 的子类（在这个例子中是 `Database\Connector\Mysql`）。连接器是负责与特定数据库引擎进行实际交互的类。它们执行查询并返回元数据（例如最后插入的 ID 和受影响的行数）。

查询器负责生成发送给数据库引擎的实际指令，例如针对 `INSERT`、`UPDATE` 和 `DELETE` 的 SQL 查询。

#### 连接器

我们的数据库连接器并不仅限于关系型数据库引擎，但由于本书篇幅有限，我们只会构建 MySQL 连接器/查询器类。我将其他连接器/查询器的创建留作读者练习。

> **注：** 我们并不局限于使用 MySQL 数据库引擎，但我们只会创建适用于 MySQL 数据库服务器的连接器/查询器类。你也可以选择使用 SQL Server（www.microsoft.com/sqlserver）或 PostgreSQL（www.postgresql.org），但我选择使用 MySQL，因为它免费、易于安装，而且是三者中最流行的一个。

与我们之前配置和缓存驱动继承自一个基础驱动类类似，我们的连接器类将继承自清单 9-4 中所示的基础连接器类。

**清单 9-4.** `Database\Connector` 类

```
namespace Framework\Database
{
    use Framework\Base as Base;
    use Framework\Database\Exception as Exception;

    class Connector extends Base
    {
        public function initialize()
        {
            return $this;
        }
    }
}
```



`protected function _getExceptionForImplementation($method)`

```
{
    return new Exception\Implementation("{$method} method not implemented");
}
}
}
```

`Database\Connector` 类相当简单。它仅用特定于数据库的变体重写了异常生成方法。具体连接到数据库引擎和/或返回相关查询类实例的实际工作，则由各个供应商特定的连接器类负责。如清单 9-5 所示，我们的 MySQL 连接器类做了更多的工作。

**清单 9-5.** `Database\Connector\Mysql` 类

```
namespace Framework\Database\Connector
{
    use Framework\Database as Database;
    use Framework\Database\Exception as Exception;

    class Mysql extends Database\Connector
    {
        protected $_service;

        /**
         * @readwrite
         */
        protected $_host;

        /**
         * @readwrite
         */
        protected $_username;

        /**
         * @readwrite
         */
        protected $_password;

        /**
         * @readwrite
         */
        protected $_schema;

        /**
         * @readwrite
         */
        protected $_port = "3306";

        /**
         * @readwrite
         */
        protected $_charset = "utf8";

        /**
         * @readwrite
         */
        protected $_engine = "InnoDB";

        /**
         * @readwrite
         */
        protected $_isConnected = false;

        // 检查是否已连接到数据库
        protected function _isValidService()
        {
            $isEmpty = empty($this->_service);
            $isInstance = $this->_service instanceof \MySQLi;
            if ($this->isConnected && $isInstance && !$isEmpty)
            {
                return true;
            }
            return false;
        }

        // 连接到数据库
        public function connect()
        {
            if (!$this->_isValidService())
            {
                $this->_service = new \MySQLi(
                    $this->_host,
                    $this->_username,
                    $this->_password,
                    $this->_schema,
                    $this->_port
                );

                if ($this->_service->connect_error)
                {
                    throw new Exception\Service("无法连接到服务");
                }
                $this->isConnected = true;
            }
            return $this;
        }

        // 断开与数据库的连接
        public function disconnect()
        {
            if ($this->_isValidService())
            {
                $this->isConnected = false;
                $this->_service->close();
            }
            return $this;
        }

        // 返回相应的查询实例
        public function query()
        {
            return new Database\Query\Mysql(array(
                "connector" => $this
            ));
        }

        // 执行提供的 SQL 语句
        public function execute($sql)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("未连接到有效的服务");
            }
            return $this->_service->query($sql);
        }

        // 转义提供的值，使其对查询安全
        public function escape($value)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("未连接到有效的服务");
            }
            return $this->_service->real_escape_string($value);
        }

        // 返回最后插入行的 ID
        public function getLastInsertId()
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("未连接到有效的服务");
            }
            return $this->_service->insert_id;
        }

        // 返回上次执行的 SQL 查询所影响的行数
        public function getAffectedRows()
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("未连接到有效的服务");
            }
            return $this->_service->affected_rows;
        }

        // 返回最近发生的错误
        public function getLastError()
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("未连接到有效的服务");
            }
            return $this->_service->error;
        }
    }
}
```

`Database\Connector\Mysql` 类要复杂得多。它定义了一些可适配的属性和方法，用于执行 `MySQLi` 类特定的函数，并返回 `MySQLi` 类特定的属性。我们希望将这些与外部隔离，使我们的系统基本上是即插即用的。如果为其他数据库引擎构建了连接器，并且它们实现了我们在此处看到的函数，那么我们的系统将无法区分其中的差异。这正是我们的目标。



