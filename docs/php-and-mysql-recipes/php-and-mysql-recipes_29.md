# LoadModule php5_module modules/libphp5.so
LoadModule php7_module /usr/lib64/httpd/modules/libphp7.so
</IfModule>
```

现在可以启动 Apache 服务器，并在浏览器中访问 `phpinfo.php` 脚本，页面将显示以下信息：

## 第 1 章 ■ 安装与配置

通过使用参数 `-m` 运行 CLI 版本的 `php`，可以显示已安装的模块列表。

```
$ php -m
[PHP Modules]
Core
ctype
date
dom
fileinfo
filter
hash
iconv
json
libxml
mysqli
mysqlnd
pcre
PDO
pdo_sqlite
Phar
posix
Reflection
session
SimpleXML
SPL
sqlite3
standard
tokenizer
xml
xmlreader
xmlwriter
[Zend Modules]
```

### 方案 1-4. 安装 MySQL

### 问题

我们现在拥有一个系统，其 Web 服务器支持执行 PHP 脚本并返回脚本输出，而不是原始源文件。为了完成全功能开发环境的安装，我们需要安装 MySQL 数据库服务器。

### 解决方案

在 CentOS 7 上安装 MySQL 兼容数据库的基本方案是 MariaDB。官方的 MySQL 数据库归 Oracle 所有。它仍然保持开源，并且有几个基于社区的版本。当 Oracle 收购 Sun Microsystems 并因此获得 MySQL 源代码时，原始开发团队创建了一个新版本。他们一直在开发新功能并提升性能。到目前为止，这两个产品提供的功能集是兼容的，但随着时间的推移，我们可能会看到它们各自开发出彼此不兼容的功能。

要以 root 用户身份安装 MariaDB 服务器及其所有依赖项，请运行以下命令：

```
$ yum install mariadb-server
$ systemctl enable mariadb
$ systemctl start mariadb
```

之后，建议运行命令 `mysql_secure_installation`。此脚本将设置 root 密码、禁用远程 root 登录，并执行其他增强安全性的更改。除设置 root 密码外，对所有问题回答“yes”是安全的。该脚本的名称在 MySQL 和 MariaDB 安装中完全相同。

### 工作原理

至此，数据库服务器的安装完成，但 PHP 尚不支持该数据库。为了安装 MySQLi 扩展，需要重新配置 PHP 以添加该扩展，然后重新编译和重新安装：

```
$ ./configure --with-apxs2 --with-mysqli
$ make
$ sudo make install
```

在此环境中，需要重新编辑 `/etc/httpd/conf/httpd.conf` 文件，并移除为 `php7_module` 添加的条目。

PHP 安装现已完成。如果需要其他扩展，可以重复上述步骤。

在 Windows 上安装 MariaDB 同样简单。最新的安装包可以在 [`mariadb.org/download/`](https://mariadb.org/download/) 找到，如果你更喜欢 MySQL 版本，可以从 [`www.mysql.com/downloads/`](http://www.mysql.com/downloads/) 下载安装文件。

### 方案 1-5. 虚拟机

### 问题

当开发在 Windows 或 Mac OSX 计算机上进行，而部署在 Linux 环境中时，很容易遇到由操作系统差异引起的问题，或者可能因某个平台缺少对特定扩展的支持而出现问题。有没有办法将这些问题的风险降到最低？

### 解决方案

有许多基于云的服务提供相对廉价的虚拟 Linux 环境。其中许多环境可以在几分钟内配置并启动，用户只需为配置激活的时间付费。成本低至每月 10 美元。

在云端拥有虚拟服务器需要互联网连接才能使用。如果离线工作，那么在开发环境中安装虚拟机更为方便。虚拟机是一种系统，允许计算机上的主操作系统作为一台或多台客户系统的主机。这在 Windows、Mac OSX 和 Linux 系统上都得到支持，无论是通过原生软件、第三方免费软件（VirtualBox）还是商业产品（VM-Ware 或 Parallels）。

### 工作原理

VirtualBox 由 Oracle 支持，可以在 [`www.virtualbox.org/`](https://www.virtualbox.org/) 免费下载。安装该软件后，你可以根据需要创建任意数量的虚拟机，只要硬盘空间足够。

每台虚拟机需要 10-100GB 的磁盘空间。可以安装 Windows 和 Linux 客户系统。客户系统可以从 ISO 镜像安装，该镜像可以从首选的 Linux 发行版网站下载。Windows 甚至提供预构建系统的发行版，可以直接下载并启动。其中大部分可以用于最长 90 天的测试目的。如果你拥有有效的 Windows 许可证，还可以创建一个永不过期的安装。

## 第 2 章 ■ 类与对象

面向对象编程相比过程化编程具有许多优势。当代码在团队内或团队之间共享时，它有助于封装，并且可以保护代码元素不被滥用。当代码以编译形式共享时尤其如此，但同样的机制也可以应用于像 PHP 这样的脚本语言。本章描述了面向对象编程在 PHP 中的工作方式。

### 方案 2-1. 编写一个简单的类

### 问题

PHP 可以作为过程化编程语言使用，但你希望利用面向对象编程的众多特性（如类和对象）来创建高级库和应用程序，从而简化协作和封装。

### 解决方案

类是面向对象编程的基础。类是一组属性和方法的集合，因此编写面向对象的 PHP 应用程序时，从这里开始。

与大多数其他面向对象编程语言一样，关键字 `class` 用于定义类，该类由类变量、类常量和方法组成。类用于定义与可以从该类实例化的对象相关的所有数据和所有功能。换句话说，类是定义，对象是实时实例，并且可以从同一个类创建多个实例，每个实例作为变量中的一个值存在。

### 工作原理

第一个示例是一个简单的类定义，其中类 `foo` 有一个名为 `a()` 的方法。方法被定义为任何常规函数，但由于它在类内部定义，因此成为属于该类的成员方法。

```php
<?php
// 2_1.php
class foo {
    function a() {
        echo "Method foo::a()\n";
    }
}
```

© Frank M. Kromann 2016
F.M. Kromann, *PHP and MySQL Recipes*, DOI 10.1007/978-1-4842-0605-8_2

### 方案 2-2. 编写一个基类

### 问题

PHP 支持不同类型的类，它们服务于不同的目的。这些被称为基类、抽象类和接口类。你希望使用一个基类来包含多个子类共享的功能，这些子类都共享相同的基础功能。

### 解决方案

基类通常是一个简单的类，可用于实例化对象，这些对象可以存储成员数据并对数据执行操作。基类也可以被扩展以添加额外的或更改的功能。这个原则可以重复应用，因此扩展类可以进一步扩展，以此类推。

### 工作原理

在下面的示例中，我们有一个包含一个成员函数的类，以及第二个类，它扩展了第一个类并添加了第二个成员。

```php
<?php
// 2_2.php
class foo {
    function a() {
        echo "Method foo::a()\n";
    }
}
```


```php
class bar extends foo {

    function b() {

        echo "Method bar::b()\n";

    }

}
```

如上例所示，**类**（`Classes`）充当数据和行为的定义。它们本身不执行任何操作，必须实例化后才能使用。如果方法被定义为 `static`，则无需实例化即可使用，这将在本章后面进行描述。实例化通过为变量赋值来实现，该变量是调用 `new` 运算符、后跟类名以及构造函数可能需要的任何参数的结果。构造函数是一个特殊函数，可以定义它来自动初始化对象。构造函数方法不是必须存在的，但它确实允许像函数一样传递参数。这也在后面关于魔法方法的章节中进行了描述。

只要可用内存允许，就可以实例化一个类的多个对象。

下一个示例基于重用之前的脚本文件，该文件定义了类 `foo` 和 `bar`。变量 `$foo` 将是类 `foo` 的一个实例，`$bar` 将是类 `bar` 的一个实例。尽管类 `bar` 只包含一个名为 `b()` 的方法，但它继承了 `foo` 的所有方法，因为它扩展自该类。因此，也可以在类 `bar` 的实例上调用方法 `a()`。

```php
<?php
// 2_3.php
include "./2_2.php";

$foo = new foo();
$foo->a();

$bar = new bar();
$bar->b();
$bar->a();
?>
```

成员变量和方法的访问通过解引用符号 `->` 进行。类或对象的实例可以像任何其他变量一样使用。它可以被复制到其他变量，并作为参数传递给函数。

## 配方 2-3. 编写抽象类

### 问题

PHP 支持不同类型的类，用于不同的目的。这些被称为基类、抽象类和接口类。你想使用抽象类定义来定义一个或多个子类共有的函数。每个子类将定义每个函数的具体行为，但所有扩展抽象基类的类都必须实现所有方法。当一个类被定义为抽象类时，它必须包含至少一个抽象函数，并且抽象类不能直接实例化，只能通过扩展抽象类的类来实例化。

### 解决方案

抽象类不能实例化为对象，其唯一目的是为派生类提供基本的方法定义，并且它可以包含提供方法定义但没有任何功能的抽象方法，或者可以被覆盖或保留原样的完全定义的方法。所有抽象方法必须由派生类声明并完整定义。在构建一组具有相同功能集但实现方式不同的类时，抽象类非常有用。例如，一组数据库访问类。它们都需要 `connect`、`disconnect` 和其他函数，但实现方式略有不同。

通过定义一个包含所有公共函数的抽象基类，任何派生类都必须实现完全相同的函数，从而通过简单地实例化不同的类来更容易地在数据库之间切换。

> **注意**：要比较抽象类和基类，请参阅配方 2-2 了解使用基类的详细信息。

### 工作原理

让我们看一个这样的例子：一个数据库抽象系统。根类是一个抽象类，它定义了所有需要的方法（`connect`、`disconnect`、`query` 等），然后每个派生类将实现每种数据库类型的逻辑。甚至可能存在所有数据库共有的功能，这些功能将在抽象类中定义一次。

```php
<?php
// 2_4.php
abstract class MyDB {

    protected $link = null;

    abstract function connect($database, $host = null, $user = null, $password = null);

    abstract function disconnect();
?>
```


# 抽象函数 query($sql);

```php
function escape($str) {
    return str_replace("'", "\'", $str);
}
```

```php
class MySQL extends MyDB {
    function connect($database, $host = null, $user = null, $password = null) {
        $this->link = mysql_connect($host, $user, $password);
        mysql_selectdb($database, $this->link);
    }

    function disconnect() {
        mysql_disconnect($this->link);
        $this->link = null;
    }

    function query($sql) {
        $rs = mysql_query($sql);
    }
}
```

```php
class SQLite extends MyDB {
    function connect($database, $host = null, $user = null, $password = null) {
        $this->link = sqlite_open($database);
    }

    function disconnect() {
        sqlite_close($this->link);
        $this->link = null;
    }

    function query($sql) {
        $rs = sqlite_query($sql);
    }
}
```

在上面的示例中，抽象类 `MyDB` 被扩展为 `MySQL` 和 `SQLite` 类。这两个扩展类具有完全相同的方法，因此如果在 `MySQL` 和 `SQLite` 数据库中定义了相同的模式，开发者可以选择实现业务逻辑以同时适用于这两种系统，并且只需一个简单的配置参数即可决定在生产环境中使用哪个数据库。也很容易扩展此系统以支持其他数据库系统。这个简单系统并未为每个数据库系统查询语言的差异提供抽象，但这也可以包含在数据库抽象层中。因此，与其将查询字符串传递给 `query()` 方法，不如创建方法来访问特定类型的数据并执行连接等操作。

## 第 2 章 ■ 类与对象

### 配方 2-4. 编写接口

### 问题

虽然 PHP 支持类的扩展，但每个类只能扩展一个类。PHP 不支持多重继承。有时需要定义一组函数，这些函数必须在基类或抽象类提供的功能之外实现。当类需要预定义的一组行为来解决特定功能（如成员数据的排序）时，这一点非常有用。

### 解决方案

解决方案是使用接口类通过一组预定义的函数来定义额外的功能。

接口类更像是对一份契约的定义，而 PHP 允许任何类根据需求实现任意数量的接口。如果一个类实现了一个或多个接口，它必须实现每个接口所要求的所有方法。接口类只包含方法的定义，而不包含实际代码。可以将它们比作抽象类中抽象方法的集合。

### 工作原理

每个接口类通常是一组定义行为的方法，如下例所示，其中类 `records` 实现了包含方法 `ascending()`、`descending()` 和 `shuffle()` 的接口 `sort`。

```php
<?php
// 2_5.php
interface sort {
    function ascending();
    function descending();
    function shuffle();
}

class records implements sort {
    private $data = array();

    function add($title) {
        $this->data[] = $title;
    }

    function ascending() {
        sort($this->data);
    }

    function descending() {
        rsort($this->data);
    }

    function shuffle() {
        shuffle($this->data);
    }

    function get() {
        return $this->data;
    }
}
```

```php
$beatles = new records();
$beatles->add("Yellow Submarine");
$beatles->add("Sgt. Pepper's Lonely Hearts Club Band");
$beatles->add("Help");
$beatles->add("Abbey Road");

echo "升序排列\n";
$beatles->ascending();
print_r($beatles->get());

echo "降序排列\n";
$beatles->descending();
print_r($beatles->get());

echo "随机排列\n";
$beatles->shuffle();
print_r($beatles->get());
```

在这个示例中，变量 `$beatles` 是 `records` 类的一个实例，并且向该对象添加了披头士乐队的四张著名专辑名称。然后使用该对象对数据进行升序、降序和随机顺序排序并打印。输出结果如下：

```
升序排列
Array
(
    [0] => Abbey Road
    [1] => Help
    [2] => Sgt. Pepper's Lonely Hearts Club Band
    [3] => Yellow Submarine
)

降序排列
Array
(
    [0] => Yellow Submarine
    [1] => Sgt. Pepper's Lonely Hearts Club Band
    [2] => Help
    [3] => Abbey Road
)

随机排列
Array
(
    [0] => Yellow Submarine
    [1] => Help
)
```



[2] => `Sgt. Pepper's Lonely Hearts Club Band`

[3] => `Abbey Road`

)

# 第 2 章 ■ 类与对象

## 菜谱 2-5：将类成员作为普通函数使用

### 问题

PHP 支持两种使用类的上下文，即静态上下文和对象上下文。

在对象上下文中，类用于实例化对象：本质上这意味着创建一个包含类副本的变量，并且如果定义了构造函数，则会执行该构造函数。然后可以使用这个副本来与类的变量和方法进行交互。

在静态上下文中，使用类之前无需将其实例化为对象，且构造函数不会执行。静态调用方法几乎与普通函数调用类似，但具有对象提供的封装优势。

在某些情况下，将类成员当作普通函数来调用会很有用。通常，只有先基于类实例化一个对象，然后在该对象上执行成员函数，才能实现这一点。

### 解决方案

通过在函数定义前使用关键字 `static`，可以使该方法在静态上下文中可访问。它仍然可以作为常规成员函数使用。

### 工作原理

在下面的例子中，我们展示了一个成员如何在静态和对象两种上下文中使用。

```php
<?php
// 2_6.php

class foo {
    static function a() {
        echo "测试\n";
    }
}

foo::a();

$foo = new foo();
$foo->a();
```

在本例中，方法 `a()` 被直接/静态地调用，就像普通函数一样，只需在它前面加上类名和两个冒号即可。同样的函数也可以通过从类 `foo` 实例化一个对象 `$foo`，然后在对象上调用该方法来执行。

当静态调用成员函数时，无法使用 `$this` 引用成员变量，但可以使用 `self::` 来引用静态定义的变量。只有在对象上下文中使用类时，才允许引用 `$this`。

```php
<?php
// 2_7.php

class foo {
    static private $b = 5;

    static function a() {
        foo::$b++;
        echo foo::$b;
    }
}

foo::a();
```

请注意，成员变量是用 `private` 关键字定义的。这可以防止从任何成员函数外部访问该变量。从全局作用域中引用 `foo::$b` 会导致运行时错误。如果变量是用 `public` 关键字定义的，则可以通过 `foo::$b` 来访问它。

类中可以同时包含静态和非静态的变量与方法，但它们必须在正确的上下文中使用。如果静态函数使用 `$this` 引用来访问成员变量，脚本将产生运行时错误。

当在静态上下文中使用类时，每个成员变量在内存中只会存在一份。

当在对象上下文中使用时，成员变量会被复制到每个对象中。在一个对象中操作成员变量不会影响其他对象中的任何成员变量，即使这些对象是从同一个类实例化而来，并且该变量被定义为 `static` 也是如此。

## 菜谱 2-6：保护数据和方法

### 问题

在处理类的属性和方法时，通常需要保护数据，使其不被类作用域之外的操作所修改。这样一来，只有类本身或派生类才能访问类的变量和成员函数。

### 解决方案

在定义类变量和成员函数时使用可见性关键字，可以限制它们的访问方式。有三个可用的关键字：`public`、`protected` 和 `private`。如果未使用任何关键字，默认可见性将设置为 `public`。

任何声明为 `public` 的类属性或方法都可以直接访问。如果声明为 `protected`，则只能由对象本身或从派生类实例化的任何对象访问。最后，声明为 `private` 的类属性或方法只能由其定义所在的对象访问。

### 工作原理

在下面的例子中，使用一个包含单个成员变量的类来实例化一个对象。

```php
<?php
// 2_8.php
```


```php
class foo {

    private $a = 5;

}

$foo = new foo();

$foo->a = 7;
```

由于没有使用可见性关键字，变量 `$a` 在对象实例化后即可被操作。

使用 `private` 关键字将使该变量不可见，并且将无法再对其进行操作：

```php
<?php
// 2_9.php

class foo {

    private $a = 5;

}

$foo = new foo();

$foo->a = 7;
```

尝试操作该变量将导致以下致命错误：`Fatal error: Cannot access private property foo::$a in 2_x.php on line 6`

当成员变量为 `protected` 或 `private` 时，如果需要从类外部访问这些变量，则必须提供设置或获取值的函数，如下一个示例所示，其中使用特殊函数 `__construct()` 在对象实例化时设置值。

```php
<?php
// 2_10.php

class foo {

    private $type = null;

    function __construct($type) {
        $this->type = $type;
    }

    function getType() {
        return $this->type;
    }

}

$t = new foo('Book');
echo $t->getType() . "\n";
```

在某些情况下，需要提供对私有成员的访问权限。通过使用 `public` 关键字声明可以实现这一点，但这将同时提供对变量的读写访问。为了将访问权限限制为只读，可以将成员声明为 `private`，并提供一个返回数据副本的方法，如上一个示例所示。属性 `$type` 被声明为 `private`，方法 `getType()` 用于访问该值。该属性只能通过构造函数设置，构造函数可以在设置值之前包含验证逻辑。

## 配方 2-7：混合静态与对象上下文

### 问题

当类在静态和对象上下文中同时使用时，并且需要引用成员变量和方法，我们不能使用 `$this` 关键字。

### 解决方案

在这种情况下，使用 `self::` 前缀可以独立于类使用的上下文来访问变量和成员。

### 工作原理

声明为 `static` 的成员变量或方法可以使用 `self::` 前缀进行访问，这意味着访问声明为 `static` 的类变量。当类在静态和对象上下文中使用时，使用 `self::` 都会生效，但前提是变量必须被声明为 `static`。

```php
<?php
// 2_11.php

class foo {

    static private $value = 6;

    function getValue() {
        return self::$value;
    }

}

echo foo::getValue() . "\n";

$bar = new foo();
echo $bar->getValue() . "\n";
```

## 配方 2-8：引用类成员

### 问题

访问父类的公共或受保护属性或方法通常是通过像访问子类直接成员一样来引用它们。在某些情况下，子类中会重写方法，因此需要不同的访问方法。

### 解决方案

使用 `parent::` 前缀可以显式访问父类的变量或方法。此外，还可以通过使用类名和两个冒号（`::`）作为变量前缀，来访问类层次结构中更高层类的成员。

### 工作原理

在处理类和对象时，有几个关键关键字值得注意。它们用于在对象树中引用成员变量和方法。当一个类被实例化为对象时，可以使用 `$this` 来引用成员变量或方法。使用 `$this` 将可以访问当前对象中的所有成员和方法，以及父对象中受保护和公共的成员和方法。

类似地，我们还有 `parent::` 前缀用于访问父类的成员。这可以用来访问任何非私有的成员变量或方法，即使扩展类重写了父类的方法。

```php
<?php
// 2_12.php

class foo {

    protected $value = 6;

    function method() {
        echo "foo::method()\n";
    }

}

class bar extends foo {

    function method() {
        parent::method();
        echo "bar::method()\nvalue = {$this->value}\n";
    }

}

$a = new bar();
$a->method();
```

也可以使用类层次结构中更高层类的名称来访问公共或受保护的成员。在下面的示例中，使用 `foo::` 来引用父类的一个成员。这是在同名的方法上执行的。这会导致该方法被重写，因此必须使用 `foo::` 或 `parent::` 显式调用原始版本。使用 `$this->` 仅在对象上下文中有效，并且仅当该方法未在派生类中被重写时才能工作。

```php
<?php
// 2_13.php

class foo {

    protected $value = 6;

    function method() {
        echo "foo::method()\n";
    }

}

class bar extends foo {

    function method() {
        foo::method();
        echo "bar::method()\nvalue = {$this->value}\n";
    }

}

$a = new bar();
$a->method();
```

## 配方 2-9：类的实例化

### 问题

当对象被实例化或销毁时，有时需要执行代码来初始化对象或清理与资源的连接等。

### 解决方案

PHP 中的类有两个特殊函数，称为 `__construct()` 和 `__destruct()`。注意名称前的两个下划线。这些函数是可选的。如果定义了它们，则会在使用 `new` 运算符时，或当使用 `unset()` 函数移除对象时，或当包含对象的变量被赋予新值时，或当脚本结束自动销毁对象时执行。

构造函数可以接受任意数量的参数，但析构函数不能有任何参数。

### 工作原理

每次将类实例化为对象时，系统都会调用该类的构造函数方法来初始化对象。如果不需要特殊的初始化，类不一定需要构造函数。

构造函数使用特殊的方法名 `__construct()` 定义。

```php
<?php
// 2_14.php

class foo {

    private $type = null;

    function __construct($type) {
        $this->type = $type;
    }

}

$t = new foo('Book');
```

在这种情况下，构造函数接受一个参数，因此在实例化对象时，必须传递该参数。

类也可以定义一个 `__destruct()` 方法，该方法在对象销毁时被调用。在大多数情况下，对象销毁会在脚本终止时自动发生，但也可以通过将变量的值设置为 `null` 或对对象变量使用 `unset()` 函数以编程方式触发。在下面的示例中，`logger` 类用于将事件写入日志文件。构造函数接受文件名作为参数，`log()` 方法会向文件中添加一行，其中包含时间戳和传递给函数的文本，当对象被销毁时，它会添加一行并关闭文件。

```php
<?php
// 2_15.php

class logger {

    private $fp;

    function __construct($file_name) {
        $this->fp = fopen($file_name, "at");
    }

    function __destruct() {
        $this->log('日志文件已关闭');
        if ($this->fp) {
            fclose($this->fp);
        }
    }

    function log($message) {
        if ($this->fp) {
            $t = date("c"); //ISO 8601 时间戳
            fwrite($this->fp, "{$t} {$message}\n");
        }
    }

}

$logger = new logger('log.txt');
$logger->log('事件');
```

除了 `__construct()` 和 `__destruct()` 之外，还有几个特殊的方法名可以使开发人员的工作更轻松。它们被称为魔术方法，如果用在类上，将提供预定义的功能。所有魔术方法的名字都以两个下划线字符开头。PHP 手册中有一个章节描述了所有魔术方法，地址是：[`php.net/manual/en/language.oop5.magic.php`](http://php.net/manual/en/language.oop5.magic.php)。

## 配方 2-10：打印对象

### 问题

调试对象时，可能需要打印一个对象或对象的某种表示形式。在其他情况下，作为向客户端返回值的应用流程的一部分，需要将对象转换为字符串。


# 预定义方法与魔术方法

有许多预定义的方法可以在类中实现，这些方法通常被称为**魔术方法**。这些方法为类添加特定的行为，包括 getter、setter、构造函数、析构函数等。

通过使用魔术方法 `__toString()`，可以定义对象在字符串上下文中的行为，例如作为 `print()` 或 `echo` 的参数，或者直接嵌入在双引号字符串中。

`__toString()` 方法不接受任何参数。

### 工作原理

`__toString()` 方法定义了对象在字符串上下文中被处理时的行为。当对象与字符串比较、嵌入在字符串中或打印时，它会在字符串上下文中被处理。`__toString()` 方法必须始终返回一个字符串，如果在用作字符串的对象上未定义此方法，结果将是单词 `object`。`__toString()` 方法可用于以结构化形式从对象生成输出。

以下示例展示了一个简单的 HTML 类，可用于生成各种类型的 HTML 元素。

```php
<?php

// 2_16.php

class html {

    private $type;

    private $c = '';

    function __construct($type = 'div') {

        $this->type = $type;

    }

    function content($c) {

        $this->c .= $c;

    }

    function __toString() {

        return "<{$this->type}>{$this->c}</{$this->type}>";

    }

}

$div = new html();

$span = new html('span');

$span->content('place some text here');

$div->content($span);

echo $div;
```

在这个示例中，有两个 `html` 类的实例，分别叫做 `$div` 和 `$span`。`$span` 对象添加了一个字符串作为内容，而 `$div` 对象添加了 `$span` 对象。当 `$div` 对象在 `echo` 语句中使用时，会调用 `__toString()` 方法。当 `$span` 对象作为嵌入在字符串中的变量使用时，这会触发 `$span` 对象的 `__toString()` 方法被调用。

# 第 2 章 ■ 类与对象

## 配方 2-11：变量重载

### 问题

当类属性声明为私有时，只能通过类的方法进行访问。为每个私有成员变量创建 getter 和 setter 函数最终会导致类中出现一长串方法。有没有更简单的方法？

> **注意**：配方 2-5 涵盖了属性可见性。

### 解决方案

借助一组特殊的重载函数，可以像访问公共变量一样访问私有变量，同时对这些变量的设置提供验证。用于管理变量重载的魔术方法有 `__get()`、`__set()`、`__isset()` 和 `__unset()`。

### 工作原理

以下示例展示了如何使用这些方法。

```php
<?php

// 2_17.php

class car {

    private $type;

    private $properties = array();

    function __construct($type) {

        $this->type = $type;

    }

    function __set($name, $value) {

        switch($name) {

            case 'type' :

                $this->type = $value;

                break;

            case 'wheels' :

                if (in_array($value, array(3,4,6))) {

                    $this->properties[$name] = $value;

                }

                else {

                    trigger_error(

                        "Property wheels must be 3,4 or 6",

                        E_USER_NOTICE

                    );

                }

                break;

            case 'seats' :

                if (in_array($value, array(2,4,5,7,8))) {

                    $this->properties[$name] = $value;

                }

                else {

                    trigger_error(

                        "Property seats must be 2,4,5,7 or 8",

                        E_USER_NOTICE

                    );

                }

                break;

            default :

                $this->properties[$name] = $value;

                break;

        }

    }

    function __get($name) {

        if (array_key_exists($name, $this->properties)) {

            return $this->properties[$name];

        }

        else if($name == 'type') {

            return $this->type;

        }

        trigger_error(

            "Undefined property $name",

            E_USER_NOTICE

        );

        return null;

    }

    function __isset($name) {

        return array_key_exists($name, $this->properties);

    }

    function __unset($name) {

        unset($this->properties[$name]);

    }

    function __toString() {

        $prop = array();

        foreach ($this->properties as $k => $v) {

            $prop[] = "$v $k";

        }

        return "The {$this->type} has " . implode(", ", $prop) . "\n";

    }

}

$c1 = new car('truck');

$c1->wheels = 4;

$c1->seats = 2;

$c1->color = 'white';

$c2 = new car('big truck');

$c2->wheels = 6;

$c2->seats = 5;

$c2->color = 'blue';

echo $c1;

echo $c2;

$c2->seats = 9; // This will trigger a notice.
```

运行此脚本将生成以下输出：



# 排版后的文本

卡车有 4 个轮子，2 个座位，白色。

大卡车有 6 个轮子，5 个座位，蓝色。

**注意**：在 `/source/2_15.php` 文件第 35 行中，属性 `seats` 的值必须为 2、4、5、7 或 8。

类 `car` 有两个私有成员，名为 `$type` 和 `$properties`。变量 `$type` 在从类实例化对象时设置。在示例中，`$c1` 以类型 `truck` 实例化，`$c2` 以 `big truck` 实例化。对于类型没有验证，因此可以使用任何值。该示例还允许设置其他属性：`wheels`、`seats`、`color` 等。所有这些都存储在 `$properties` 数组中，其值由 `__get()` 和 `__set()` 魔术方法处理。代码允许添加任何属性，但对 `wheels` 和 `seats` 提供了特殊处理。如果设置的值超出有效范围，代码将使用 `trigger_error()` 函数打印一条通知。该属性不会被设置，但代码会继续执行。

## 配方 2-12. 序列化类型与对象

### 问题

序列化 PHP 变量用于将整数、字符串和数组转换为字符串表示形式，该形式可以写入文件或数据库，并在之后检索并反序列化以恢复其原始值。

序列化对象稍微复杂一些，因为它们可能包含对资源的连接以及其他在序列化和反序列化时需要特殊处理的值。

### 解决方案

当对对象使用 `serialize()` 和 `unserialize()` 函数时，可能需要对对象执行特殊操作，以正确处理成员变量的初始化。在此上下文中会用到 `__sleep()` 和 `__wakeup()` 方法。当对象被序列化时，如果存在 `__sleep()` 函数，它会被调用。该函数应返回一个包含应被序列化的成员变量名称的数组。

当使用 `unserialize()` 函数重新创建对象时，会调用 `__wakeup()` 函数。这可用于处理诸如断开和重新连接数据库之类的操作，或是进行其他形式的清理和初始化，以确保对象的正确使用。

### 工作原理

以下示例展示了一个数据库类的对象实例如何被序列化和反序列化。

创建对象时，构造函数的参数存储在私有变量中，并建立与数据库的连接。序列化对象时，`__sleep()` 函数会返回三个私有变量的名称；反序列化时，`__wakeup()` 函数会自动重新连接到同一个数据库。

```php
<?php
// 2_18.php

class Database {
    protected $pdo;
    private $dsn, $username, $password;

    public function __construct($dsn, $username, $password) {
        $this->dsn = $dsn;
        $this->username = $username;
        $this->password = $password;
        $this->connect();
    }

    private function connect() {
        $this->pdo = new PDO($this->dsn, $this->username, $this->password);
    }

    public function __sleep() {
        return array('dsn', 'username', 'password');
    }

    public function __wakeup() {
        $this->connect();
    }
}

$db = new Database('sqlite:mydb', 'user', 'password');
$a = serialize($db);
echo $a;
```

该脚本将输出一行描述序列化对象的内容：`O:8:"Database":3:{s:13:"Databasedsn";s:11:"sqlite:mydb";s:18:"Databaseusername"; s:4:"user";s:18:"Databasepassword";s:8:"password";}`

## 配方 2-13. 复制对象

### 问题

如果通过使用 `new` 运算符实例化类来创建对象，然后将其分配给第二个变量，PHP 只会创建对该对象的引用，而不是复制该对象。操作存储在第一个变量中的对象也会影响存储在第二个变量中的对象，因为它们都引用同一个对象。

### 解决方案

魔术函数 `__clone()` 用于对新创建的对象副本执行特殊操作。与任何魔术方法一样，这些方法不能直接调用，而是由特殊操作触发。



# `__clone()` 方法

当使用`clone`关键字创建对象时，会调用`__clone()`方法。与将对象赋值给一个新变量（这会创建对象的引用）相比，克隆对象可以提供对象的真正副本。

### 工作原理

以下示例展示了将对象赋值给新变量与克隆对象之间的区别。

```php
<?php

// 2_19.php

class foo {

    public $a = 5;

}

$b = new foo();

$c = $b;

$c->a++;

echo $b->a . "\n";

$d = clone $b;

$d->a++;

echo $b->a . "\n";

echo $d->a . "\n";
```

在第一部分中，对象`$b`被赋值给一个新变量，成员变量`a`增加了 1。这导致`$b->a`和`$c->a`都变为 6，因为变量`$b`和`$c`引用的是同一个对象。

在第二部分中，对象`$b`被克隆并存储在变量`$d`中。修改`$d`中成员变量`a`的值不会影响`$b`中`a`的值，因为`$b`和`$d`引用的是两个不同的对象。此操作没有使用`__clone()`方法。添加这个魔术方法可以在对象被克隆后对其执行操作，如下所示：

```php
<?php

// 2_20.php

class foo {

    public $a = 5;

    function __clone() {

        $this->a = 0;

    }

}

$b = new foo();

$c = $b;

$c->a++;

echo $b->a . "\n";

$d = clone $b;

$d->a++;

echo $b->a . "\n";

echo $d->a . "\n";
```

现在，在复制所有成员变量后，代码会调用`__clone()`方法。因此，在克隆后，`$d`的成员`a`会被设置为 0，然后递增为 1，并且仍然不会影响原始对象。

## 配方 2-14：将对象用作函数

### 问题

PHP 支持可变函数名。这使得可以将函数名存储在字符串`$a = "MyFunction";`中，并通过`$a();`语法执行该函数。如果该变量包含一个对象，PHP 将生成运行时错误。

### 解决方案

在类上定义魔术方法`__invoke()`将导致该对象在作为函数使用时执行该方法。

### 工作原理

在下面的示例中，当对象作为函数被调用时，它将对其自身使用`var_dump()`函数。

```php
<?php

// 2_21.php

class foo {

    private $a = 5;

    function __invoke() {

        var_dump($this);

    }

}

$a = new foo();

$a();
```

该脚本的输出如下：

```
object(foo)#1 (1) {
  ["a":"foo":private]=>
  int(5)
}
```

# 配方 2-15：方法重载

### 问题

PHP 不支持像其他语言那样的方法重载——即同一个方法名在同一个类中可以使用多次，但参数列表不同。

### 解决方案

与变量或属性可以通过`__get()`和`__set()`魔术方法进行重载的方式相同，方法也可以被重载。这通过`__call()`和`__callStatic()`来实现。

如果定义了这些函数，并且使用对象来调用一个不存在的成员函数，则会调用这些函数之一。如果该成员在对象上下文中被调用，将使用`__call()`；当该方法在静态上下文中被调用时，将使用`__callStatic()`。以下示例展示了如何实现方法重载。它还演示了重载方法的方法名将按原样处理（区分大小写），而不是像普通函数和方法那样通常作为不区分大小写的名称处理。

### 工作原理

在下一个示例中，我们创建了一个名为`MethodOverload`的类，并定义了处理方法重载的函数。

```php
<?php

// 2_22.php

class MethodOverload {

    private function call($name, $param, $context) {

        switch ($name) {

            case 'f1' :

            case 'f2' :

                echo "You called $name in $context context\n";

                break;

            default :

                trigger_error("Undefined method '$name'", E_USER_NOTICE);

                break;

        }

    }

    function __call($name, $param) {

        $this->call($name, $param, 'object');

    }

    static function __callStatic($name, $params) {

        self::call($name, $param, 'static');

    }

    static function abc() {

        echo "abc() called\n";

    }

}

$a = new MethodOverload();

$a->f1();

MethodOverload::f2();

MethodOverload::F2();

MethodOverload::ABC();
```



类被实例化，并在对象上下文中调用函数`f1()`。这个方法并不存在，但因为它在`__call()`方法中被定义，因此仍然可以执行该方法而不会出错。

同样地，该对象也支持在静态上下文中调用方法。该脚本的输出如下所示：

```
You called f1 in object context
You called f2 in static context
Notice: Undefined method 'F2' in /source/02/2_22.php on line 11
abc() called
```

## 配方 2-16. 调试对象

### 问题

当对象被传递给诸如`var_export()`或`var_dump()`等函数时，它们可能会暴露数据库凭据或其他本应对类/对象用户隐藏的配置选项。

### 解决方案

使用魔术方法`__set_state()`和`__debugInfo()`可以改变调试函数处理对象的方式。方法`__set_state()`必须声明为静态的，并接受一个参数，该参数是一个包含对象上所有属性的数组，并且它应返回`var_export()`需要处理的值。返回值可以是任何类型，因此可以简单地返回一个移除了某些输入参数的数组。如果在将对象传递给`var_export()`时未定义`__set_state()`，系统将生成一个致命错误并停止执行。

方法`__debugInfo()`是在 PHP 5.6 中引入的，可用于覆盖由`var_dump()`转储的属性。如果未定义`__debugInfo()`，`var_dump()`将转储所有私有、保护和公共成员。

### 工作原理

这可用于在调试输出中隐藏数据库凭据等内容，如下例所示：

```php
<?php
// 2_23.php

class database {
    protected $host;
    protected $user;
    protected $password;

    function __construct($host, $user, $password) {
        $this->host = $host;
        $this->user = $user;
        $this->password = $password;
    }
}

$db = new database('localhost', 'user', 'secret');
var_dump($db);

class database2 {
    protected $host;
    protected $user;
    protected $password;

    function __construct($host, $user, $password) {
        $this->host = $host;
        $this->user = $user;
        $this->password = $password;
    }

    function __debugInfo() {
        return array(
            'host' => $this->host,
        );
    }
}

$db = new database2('localhost', 'user', 'secret');
var_dump($db);
```

当这个脚本在 PHP 5.6 或更高版本上执行时，会产生以下输出，其中第一部分来自没有`__debugInfo()`方法的对象，第二部分仅包含被选择导出的属性。

```
object(database)#1 (3) {
  ["host":protected]=>
  string(9) "localhost"
  ["user":protected]=>
  string(4) "user"
  ["password":protected]=>
  string(9) "secret"
}

object(database2)#2 (1) {
  ["host"]=>
  string(9) "localhost"
}
```

## 配方 2-17. 不使用类使用对象

### 问题

将关联数组转换为对象后，可以像访问常规对象中定义的成员数据一样访问它们。

### 解决方案

PHP 有一个内置类叫做`stdClass`。它在将数组强制转换为对象时被使用，也可以用于在不先定义类的情况下创建一个对象。

### 工作原理

`stdClass`没有任何成员变量或方法，但当数组被转换为对象时，它会将关联数组的所有元素转换为公共成员，如下例所示：

```php
<?php
// 2_24.php
$a = [
    'host' => 'localhost',
    'database' => 'mydb',
    'user' => 'user'
];

$o = (object)$a;
echo $o->host;
```

`stdClass`也可以用于创建一个空对象，并手动向其添加公共成员变量。

```php
<?php
// 2_25.php
$a = new stdClass();
$a->host = 'localhost';
```

在这个例子中，默认的 setter 会导致在对象上创建一个名为`host`的公共成员变量。

## 配方 2-18. 目录迭代

### 问题



# 遍历文件和目录

遍历文件和目录可以通过函数 `opendir()`、`readdir()` 和 `closedir()` 以过程式方式完成。能否用面向对象的方法实现相同的功能呢？

### 解决方案

部分内置函数也会返回对象。例如，`dir()` 函数会返回内置类 `Directory()` 的一个实例。它接受目录路径作为第一个参数。该类提供了 `read()` 和 `close()` 方法。

## 运行原理

下面的示例展示了如何使用这些函数遍历目录，并输出所有文件的名称和类型。

```php
<?php
// 2_26.php

$path = "./";

// 打开已知目录，并继续读取其内容
if (is_dir($path)) {
    if ($dh = opendir($path)) {
        while (($file = readdir($dh)) !== false) {
            echo "filename: $file : filetype: " . filetype($path . $file) . "\n";
        }
        closedir($dh);
    }
}
```

使用面向对象接口和 `Directory` 类编写的相同代码可能如下所示。

```php
<?php
// 2_27.php

$path = "./";

// 打开已知目录，并继续读取其内容
if (is_dir($path)) {
    $dir = dir($path);
    while (($file = $dir->read()) !== false) {
        echo "filename: $file : filetype: " . filetype($path . $file) . "\n";
    }
    $dir->close();
}
```

`dir()` 函数会设置两个公共属性 `path` 和 `handle`，因此可以通过创建 `Directory` 类的实例并手动设置属性值来实现相同的功能。

```php
<?php
// 2_28.php

$path = "./";

// 打开已知目录，并继续读取其内容
if (is_dir($path)) {
    $dir = new Directory();
    $dir->path = $path;
    $dir->handle = opendir($path);
    while (($file = $dir->read()) !== false) {
        echo "filename: $file : filetype: " . filetype($path . $file) . "\n";
    }
    $dir->close();
}
```

最后，还有一个预定义的类 `Exception`，它用于通过 `try`、`catch`、`finally` 模式处理运行时错误，这将在后续章节中讨论。

# 技巧 2-19：编写可复用代码

### 问题

PHP 仅允许类从单个基类继承。它允许使用接口类，但这要求用户每次使用时都必须输入每个接口的功能。

在处理类和对象时，通常需要在多个类中拥有完全相同的功能。这些功能可以是执行验证的私有方法，也可以是提供对私有成员访问权限的公共方法等。在许多情况下，这是通过在基类中创建方法处理的，但当类派生自不同的基类时，这便无法实现。

### 解决方案

PHP 5.4 及更新版本支持使用 **trait** 来处理这种情况。Trait 是可被多个不同类复用的具名代码块。代码块在一个位置编写和维护，并在许多位置使用。修改一处将影响所有使用它的地方。

## 运行原理

Trait 的定义与类定义几乎相同。使用关键字 `trait`，后跟名称以及一组成员变量和/或函数。

以下示例展示了如何将 trait 用于定义单个类中使用的受保护变量。

```php
<?php
// 2_29.php

trait A {
    protected $var;
}

class B {
    use A;

    function Get() {
        return $this->var;
    }

    function Set($var) {
        $this->var = $var;
    }
}

$o = new B();
$o->Set(15);
echo $o->Get();
```

在此示例中，只有一个变量和一个使用该 trait 的类，但如果涉及多个变量以及许多使用同一组变量的类，这将节省输入量，并使代码维护更容易。

当 trait 也包含函数时，节省的效益才真正显现出来，如下一个示例所示，其中 `Get()` 和 `Set()` 函数被移入 trait，并且该 trait 被两个不同的类使用。

```php
<?php
// 2_30.php

trait A {
    protected $var;

    function Get() {
        return $this->var;
    }

    function Set($var) {
        $this->var = $var;
    }
}

class B {
    use A;
}

class C {
    use A;
}

$o = new B();
$o->Set(15);
echo $o->Get();
```



# Trait 在编译时充当复制的粘贴函数

`Trait`在编译时充当复制的粘贴函数。这意味着`trait`的副本会被插入到所有使用它的地方，就好像代码在所有地方都完全相同。如果 trait 用于静态上下文中使用的类，则复制代码的概念与同一基类声明的静态对象略有不同。如果成员变量或函数被声明为`static`并在两个不同的类中使用，它们将成为不同的实例。这种情况仅发生在类用于静态上下文中时。接下来的两个示例展示了继承基类和使用`trait`定义成员变量之间的区别。

第 2 章 ■ 类与对象

```php
// 2_31.php

class A {

    public static $var;

}

class B extends A {}

class C extends A {}

B::$var = 15;

echo B::$var . "\n";

echo C::$var . "\n";
```

在这种情况下，输出将是两行数字 15。

```php
// 2_32.php

trait A {

    public static $var;

}

class B {

    use A;

}

class C {

    use A;

}

B::$var = 15;

echo B::$var . "\n";

echo C::$var . "\n";
```

在这种情况下，输出将是第一行数字 15，接着是一个空行。

为了避免命名冲突，需要处理来自基类的方法、`trait`以及当前类的成员。优先顺序是：当前类的成员会覆盖`Trait`方法，而`Trait`方法又会覆盖继承的方法。这使得可以定义一个包含函数的`trait`，这些函数将覆盖从基类继承的函数，或者在当前类中覆盖从`trait`插入的函数。因此，对于十分之九的实例中函数都相同的情况，可以在需要时通过覆盖`trait`中的一个或多个方法来处理。

Traits 也可以包含抽象函数。在这种情况下，需要使用该 trait 的类必须定义该函数。

就像类一样，traits 可以被扩展以包含其他 traits。这样处理起来会更方便，因为开发者可以包含一个简单的`trait`、一组 traits，或者一个组合了所有所需 traits 的`trait`，如下例所示，其中两个分别用于加法和减法的 traits 被添加到了一个名为`math`的单个`trait`中。

请注意，`trait`的名称与其内部的函数名称相同。这就像类中可以有与类名同名的成员方法一样。

第 2 章 ■ 类与对象

```php
// 2_33.php

trait add {

    function add($a, $b) {

        return $a + $b;

    }

}

trait subtract {

    function subtract($a, $b) {

        return $a - $b;

    }

}

trait math {

    use add, subtract;

}

class calc {

    use add, subtract;

}

class calc2 {

    use math;

}

$o = new calc2();

echo $o->add(4,5) . "\n";
```

如果多个 traits 中使用了相同的方法名，那么在类中插入多个 traits 可能会导致冲突。可以通过使用关键字`insteadof`指定要使用的方法，以及在使用 trait 时重命名冲突的方法来解决这个问题。在下面的示例中，两个 traits `A` 和 `B` 都有相同的函数`f1()`。通过在`use`语句中添加`B::f1 insteadof A`，解析器将使用来自`B` trait 的`f1()`函数。第二行将 trait `A` 中的`f1()`函数重命名为`func1()`，允许其在类`C`中使用。

第 2 章 ■ 类与对象

```php
// 2_34.php

trait A {

    function f1() {}

}

trait B {

    function f1() {}

}

class C {

    use A, B {

        B::f1 insteadof A;

        A::f1 as func1;

    }

}
```

## 配方 2-20：避免名称冲突

### 问题

对于被包含在其他项目中的库的作者来说，一个常见的问题是名称冲突。不能使用相同名称多次定义函数或类。早期，通过在函数和类名中添加前缀来解决这个问题，但这通常会导致代码库可读性降低。

### 解决方案

命名空间的概念在 PHP 5.3 中引入，并被认为是该版本中对该语言最重要的改进之一。随着关键字`namespace`和`use`的引入，可以在命名空间内定义和使用函数，从而允许在多个地方使用相同的函数名。命名空间的定义必须是脚本中的第一个语句。如果在任何其他位置定义，解释器将停止脚本并产生致命错误。如果同一个文件中需要多个命名空间，可以通过再次使用`namespace`关键字来定义它们。这不会导致错误，因为脚本的第一部分已经属于一个命名空间。

虽然在命名空间定义之后允许任何脚本代码，但它仅适用于函数、类和常量。其他所有内容都将出现在全局空间中。

名为`PHP`的命名空间保留给内部函数使用，除此之外，命名名称遵循与变量名相同的规则。它们必须以字母或下划线开头，后面的字符可以是字母、数字和下划线。

反斜杠字符也是允许的，但它实际上不会成为名称的一部分。它充当子命名空间的分隔符。这在大型项目中非常有用，因为项目的多个部分可能有同名函数或类。

### 工作原理

以下示例展示了一个简单的命名空间声明。

```php
// 2_35.php

namespace Test;

function t1() {

}
```

在许多方面，命名空间的工作方式类似于文件系统，其中文件名或路径可以是相对于当前目录的、相对于当前目录的子目录的，或者可以相对于文件系统根目录进行绝对指定。

同样的原则也适用于命名空间，其中当前命名空间由`namespace`关键字设置，而常量、函数和类则是相对于当前命名空间或绝对引用的。以下两个示例脚本对此进行了说明。

```php
// 2_36.php

namespace Test\Utilities;

function f1($a) {

    return $a * 2;

}
```

第 2 章 ■ 类与对象

```php
// 2_37.php

namespace Test;

include "2_36.php";

function f1($a) {

    return $a * 3;

}

echo "相对引用\n";

echo f1(5) . "\n";

echo Utilities\f1(5) . "\n";

namespace MyProject;

echo "绝对引用\n";

echo \Test\f1(5) . "\n";

echo \Test\Utilities\f1(5) . "\n";
```

函数`f1()`在第一个脚本文件中定义，它属于命名空间`Test\Utilities`。

函数`f1()`也在第二个脚本文件中定义，但这次是在命名空间`Test`下，因此这两个函数不会产生冲突。访问这两个函数可以通过相对方式或绝对方式进行：`f1(5)`将调用`Test`命名空间（当前命名空间）中的函数版本，而`Utilities\f1(5)`则调用来自包含文件中定义在子命名空间中的版本。

然后命名空间更改为`MyProject`，此时引用`f1()`函数的两个副本的唯一方法是使用从根开始的绝对路径。

有时，当使用包含子命名空间的长命名空间时，为完整名称创建别名会很有用，如下一个示例所示，其中包含了前面的`2_35.php`文件，并将该文件中使用的命名空间别名为命名空间`U`。

```php
// 2_38.php

include "2_36.php";

use Test\Utilities as U;

echo U\f1(3) . "\n";
```

此脚本没有定义命名空间，尽管包含的文件中定义了一个，但当前命名空间仍然是根命名空间或全局空间。现在可以通过别名`U`访问命名空间`Test\Utilities`中的函数。除了为命名空间创建别名之外，还可以在当前命名空间中导入或使用类和接口。此功能在 PHP 5.6 版本中得到扩展，允许包含函数和常量。这是通过`use function`或`use constant`完成的，其中单独的`use`关键字作用于类。

```php
// 2_39.php

include "2_36.php";

use function Test\Utilities\f1;

echo f1(3) . "\n";
```



# 排版后的文档

如果与 PHP 5.5 或更早版本一起使用，上面的示例将产生解析错误（`parse error`）。

## 第 2 章 ■ 类与对象

### 配方 2-21：在首次使用时自动加载类

### 问题

处理许多类并记住包含正确的类定义文件是很繁琐的。

### 解决方案

自动加载是一种概念，它允许在首次使用类时自动包含定义该类的文件。

### 工作原理

有几种不同的方法可以实现此功能。简单的方法是将每个文件命名为与其定义的类相同的名称，并定义魔术函数`__autoload()`。

```php
<?php
// 2_40.php
function __autoload($class_name) {
    include $class_name . '.php';
}
```

使用这种方法将减少在每个文件开头编写长串`include`语句的需要，并将确保只包含脚本所需的类。函数的内容可以像上面的示例那样是一个简单的`include`，也可以是一个更复杂的语句，通过查找表来确定需要哪个文件。

或者，可以使用来自 SPL 扩展的`spl_autoload_register()`函数来注册一个函数或类方法，该方法可以根据查找表或其他方法将类名称解析为特定的文件名。SPL 扩展在大多数 PHP 安装中默认启用。

下面是一个如何使用`spl_autoload_register()`函数的示例：

```php
<?php
// 2_41.php
function autoload($class) {
    include "../classes/{$class}.inc";
}
spl_autoload_register("autoload");
```

当使用此代码时，如果一个在 PHP 和当前脚本中尚未定义的类被实例化，自动加载器将执行`autoload()`函数。如果文件未找到或者它没有定义该类，将会出现错误；否则，该类将被包含，并且该类被定义。

## 第 3 章

## 执行数学运算

大量数学函数内置于 PHP 核心中，通过扩展（包括任意精度数字、统计和线性代数）可以获得额外的功能。

内置函数涵盖指数、对数、三角函数、双曲函数以及舍入和随机数生成器。

PHP 支持整数和浮点数据类型，可用于执行基本的数学运算：加法（`+`）、减法（`-`）、乘法（`*`）和除法（`/`）。执行这些操作不需要特殊的函数，逻辑可以直接写入代码中。

```php
<?php
// 3_1.php
$i = 5;
$j = 5 * $i;
```

数学运算的执行顺序遵循与其他大多数编程语言相同的标准，即乘法和除法在加法和减法之前执行。

在下一个示例中，计算执行的是`3 * $i`，在这种情况下是 15，然后加上 5，所以`$j`的内容将是 20。

```php
<?php
// 3_2.php
$i = 5;
$j = 5 + 3 * $i;
```

如果需要先加 3 和 5，然后再乘以`$i`，则该操作必须用括号括起来。

```php
<?php
// 3_3.php
$i = 5;
$j = (5 + 3) * $i;
```

在这种情况下，先加 5 和 3 再进行乘法，所以`$j`中的结果是`8 * 5`，即 40。

PHP 5.6 版本引入了一个新的数学运算符（`**`），作为创建指数表达式的一种简单方法。在 5.6 版本之前，使用`pow()`函数是执行此数学运算的唯一方法。

```php
<?php
// 3_4.php
$k = 2 ** 10;
```

© Frank M. Kromann 2016

F.M. Kromann, *PHP and MySQL Recipes*, DOI 10.1007/978-1-4842-0605-8_3

第 3 章 ■ 执行数学运算

使用`pow()`函数编写此代码的旧方式如下：

```php
<?php
// 3_5.php
$k = pow(2, 10);
```

如果底数和指数都是非负整数，并且结果可以用整数表示（小于`PHP_INT_MAX`），则结果将是整数。在所有其他情况下，结果将是一个浮点值。

PHP 还支持取模（`%`）运算符，用于查找整数除法的余数。

## 配方 3-1：更改数字的基数

### 问题

默认情况下，PHP 使用十进制表示数字。当数字作为输出打印，或数字用于给变量赋值或用于计算时，都假定使用的是十进制。在各种情况下，对其他基值数字的使用会使代码更具可读性。

### 解决方案

PHP 支持其他 3 种基值（2、8、16）：二进制、八进制和十六进制。为了告诉 PHP 公式中写的数字不是十进制数，需要为数字添加限定符：二进制为`0b`，八进制为`0`，十六进制为`0x`。

### 工作原理

在下一个示例中，数字 100 分别在不带限定符以及使用三种特殊限定符转换为二进制、八进制和十六进制值的情况下使用。输出不进行相同的转换，因此程序将显示数字 100 在每种表示中的十进制值。

```php
<?php
// 3_6.php
$i = 100;
$b = 0b100;
$o = 0100;
$h = 0x100;
echo $i . "\n";
echo $b . "\n";
echo $o . "\n";
echo $h . "\n";
```

此示例的输出如下：

第 3 章 ■ 执行数学运算

在数字上使用基限定符时，有效的数字如下：

*   `0b`或二进制：只允许 0 和 1，
*   `0`或八进制：只允许 0 到 7，
*   `0x`或十六进制：允许 0-9 以及字母 a-f 或 A-F。

使用无效数字将产生解析错误。

允许这些限定符使得某些类型的值更具可读性。考虑 Linux 文件系统上的文件权限。这些通常用三个八进制数字表示所有者、组和其他权限，其中 4 代表读取，2 代表写入，1 代表执行权限。数字`0750`表示所有者具有读、写和执行权限；组具有读和执行权限；而其他所有人没有任何访问权限。如果这个数字用十进制表示，它将是 488，并且不容易读取访问权限。

### 配方 3-2：转换为不同的基值

### 问题

在数字上使用限定符仅当数字作为 PHP 代码的一部分编写时才是选项。如果数字来自数据库或 API 调用，并且数字作为程序的输出写入，我们需要其他方法来在不同的基值之间进行转换。

### 解决方案

PHP 自带了一些函数使这变得容易。函数`dechex()`将十进制整数值转换为对应的十六进制值。将十六进制字符串转换为整数的相反操作由`hexdec()`函数处理。类似地，函数`decbin()`和`bindec()`在十进制和二进制之间转换，最后函数`decoct()`和`octdec()`在十进制和八进制之间转换。

除了基限定符和转换函数之外，PHP 还带有一个内置函数，可用于在任意基数之间转换数字。函数`base_convert()`接受三个参数，分别称为值（`value`）、源基数（`frombase`）和目标基数（`tobase`）。

`base_convert(number, frombase, tobase);`

该值可以是整数或表示整数的字符串。如果使用整数，PHP 会在进行转换前将数字转换为字符串。返回值始终是一个字符串。如果返回值是十进制的，则生成的字符串可以在计算中用作数字字符串，PHP 会在执行计算时将其转换为数字。

### 工作原理

下面的示例将打印值 4。

```php
<?php
// 3_7.php
echo base_convert(100, 2, 10);
```

请注意，输入值 100 是作为整数给出的，函数会在执行操作之前使用常规的十进制表示将其转换为字符串。如果该数字是以二进制表示形式（`0b100`）编写的，系统会将该值转换为字符串（4），在这种情况下，函数将返回 0，因为 4 在二进制中不是有效数字。

第 3 章 ■ 执行数学运算

```php
<?php
// 3_8.php
echo base_convert(0b100, 2, 10);
```



# 数字进制转换

将数字转换为不同进制有助于精简数字字符串的尺寸。有效进制范围是 2 到 36 之间的任意整数，这一限制由英文字母表的 26 个字符和 10 个数字共同决定。

## 方案 3-3：将二进制值存储在整数中

### 问题描述

编程中通常使用位值来存储只有两种可能值（开/关或真/假）的标志或选项。这允许在单个整数值中存储 32 个或 64 个值。使用十进制值来检查、设置或清除这些标志位略显笨拙，因为数字（1, 2, 4, 8, 16, 32, 64 等）既难记忆又难操作。

### 解决方案

除了基本的数学运算外，PHP 还支持二进制运算。这些运算可用于改变数字中的单个位、将位左移或右移等。根据操作系统和 PHP 编译方式的不同，整数值中的可用位数可以是 32 位或 64 位。这提供了`2 ** 32`或`2 ** 64`种不同的整数值。由于 PHP 的所有整数都是有符号的，首位用于处理符号，当二进制字符串转换为十进制数时，这剩下`2 ** 31`或`2 ** 63`个正负值。

### 实现原理

我们可以使用`base_convert()`函数打印整数值的二进制表示。在 64 位版本的 PHP 中，以下代码将生成 63 个 1：

```php
<?php

// 3_9.php

echo base_convert(PHP_INT_MAX, 10, 2);
```

这是`9223372036854775807`的二进制表示。只有 63 个 1，因为第一位是 0，尽管它仍是 64 位的一部分，但`base_convert()`函数不会打印它。

通过使用二进制非运算符（`~`），数字中的所有位都可以翻转，因此在下一个示例中，我们将得到一个 1 后跟 63 个 0。

```php
<?php

// 3_10.php

echo base_convert(~PHP_INT_MAX, 10, 2);
```

## 方案 3-4：在二进制中设置和清除位

### 问题描述

你想要在二进制表示中设置或清除单个位。

**注意：** 方案 3-2 展示了如何将数字转换为二进制表示以供显示。

### 解决方案

除了非（`~`）运算符外，PHP 还支持按位与（`&`）、或（`|`）、异或（`^`）以及左移（`<<`）和右移（`>>`）。按位非、与、或函数与布尔非（`!`）、与（`&&`）和或（`||`）运算不同。事实上，单词`AND`和`OR`可以替代`&&`和`||`使用，这与使用 SQL92 语法进行数据库查询的方式相同。

按位或运算符使用或逻辑将两个变量中相同位置的位组合起来，形成新变量中的位。如果两个位都是 0，则结果位也为 0。如果其中一个或两个位是 1，则结果位为 1。

按位与运算符使用与逻辑将两个变量中相同位置的位组合起来，形成新变量中的位。如果两个位都是 1，则结果位也为 1。如果其中一个或两个位是 0，则结果位为 0。

按位异或运算符使用异或逻辑将两个变量中相同位置的位组合起来，形成新变量中的位。如果其中一个位是 1 且另一个是 0，则结果位为 1。如果两个位都是 0 或都是 1，则结果位为 0。

### 实现原理

如果一个变量有一个值，并且需要将某些位设置为 1，可以通过或运算，加上精确位的值来实现。在下面的示例中，变量`$v`被赋值为 12（`0b00`）。

然后，使用按位或运算将变量与 2（`0b10`）组合，结果存储在`$x`中。

结果是 14（`0b1110`），因为每个位置的所有位都是从最低有效位开始组合的。

在这个示例中，两个数字分别定义了 2 位和 4 位，但在内部表示中，两个数字都有 32 位或 64 位，未定义的位默认为 0。在这种情况下，数字可以写成`0b1100`和`0b0010`。两种表示都有效，对于八进制和十六进制数也是如此，前导零有助于提高可读性。

```php
<?php

// 3_11.php

$v = 0b1110;

$x = $v | 0b10;
```

如果需要相反的操作，即清除特定位，可以使用与运算符，但还需要使用非运算符。考虑以下示例，其中`$v`被赋值为 14（`0b1110`），目标是清除第三位以得到结果 10（`0b1010`）。

```php
<?php

// 3_12.php

$v = 0b1110;

$x = $v & 0b100;
```

在这种情况下，与运算的结果将是值 4（`0b100`），因为第三位是唯一两个位都是 1 的位置。如果在与运算之前将值改为 13（`0b1011`），结果将是预期的。在值前面添加非运算符将实现这一点。

```php
<?php

// 3_13.php

$v = 0b1110;

$x = $v & ~0b100;
```

如果打印`~0b100`，将显示结果-5。这是因为默认输出是十进制。这可以通过使用`printf()`函数将输出格式化为二进制来说明。

```php
<?php

// 3_14.php

printf("%b\n", 0b100);

printf("%b\n", ~0b100);
```

输出如下：

在这个示例中，所有 64 位都从 0 翻转为 1 或从 1 翻转为 0。

## 方案 3-5：使用十六进制数

### 问题描述

以二进制方式使用整数来表示标志很方便，但编写由 0 和 1 组成的长数字不利于可读性。

### 解决方案

使用十六进制（基数为 16）系统可以有所帮助。十六进制数中的每个数字代表值的 4 位。在 32 位系统上，整数可以用 8 位数字表示，在 64 位系统上，整数可以用 16 位数字表示。

### 实现原理

将示例 14（`0xE`）和 8（`0x8`）中的二进制数改为十六进制，将使代码更简短，同时仍保持可读性。

```php
<?php

// 3_15.php

$v = 0xE;

$x = $v & ~0x8;
```

随着涉及的数字位数增多，这一点变得更加明显。

## 方案 3-6：通过二进制移位提升性能

### 问题描述

过去，两个数字相乘对计算机来说很困难。需要大量 CPU 周期，在资源有限的情况下，这是一个问题。

### 解决方案

通过使用二进制数学，可以通过简单的位移位来将数字乘以 2 或除以 2。按位移位运算在算术上的工作原理是：将所有位向左移动一位等同于乘以 2，向右移动一位等同于除以 2。左移运算符会在右侧添加 0 并丢弃左侧的位。右移运算符会保留符号位并丢弃右侧的位。

### 实现原理

移位运算符可以移动任意数量的位。移位运算符的左侧是要移动位的数字，右侧是要移动的位数。

```php
<?php

// 3_16.php

$bits = PHP_INT_SIZE * 8;

$v = 1;

for ($i = 0; $i < $bits; $i++) {

printf("$v << $i = %64b \n", $v << $i);

}
```

移位数量超过整数位数（32 或 64）将导致未定义行为。整数位数可以通过常量`PHP_INT_SIZE`乘以 8（一个字节中的位数）来计算。上面示例的输出如下：

```
1 << 0 = 1
1 << 1 = 10
1 << 2 = 100
1 << 3 = 1000
1 << 4 = 10000
1 << 5 = 100000
1 << 6 = 1000000
1 << 7 = 10000000
1 << 8 = 100000000
1 << 9 = 1000000000
1 << 10 = 10000000000
1 << 11 = 100000000000
1 << 12 = 1000000000000
1 << 13 = 10000000000000
1 << 14 = 100000000000000
1 << 15 = 1000000000000000
1 << 16 = 10000000000000000
1 << 17 = 100000000000000000
1 << 18 = 1000000000000000000
1 << 19 = 10000000000000000000
1 << 20 = 100000000000000000000
1 << 21 = 1000000000000000000000
1 << 22 = 10000000000000000000000
1 << 23 = 100000000000000000000000
```


```text
1 << 24 = 1000000000000000000000000
1 << 25 = 10000000000000000000000000
1 << 26 = 100000000000000000000000000
1 << 27 = 1000000000000000000000000000
1 << 28 = 10000000000000000000000000000
1 << 29 = 100000000000000000000000000000
1 << 30 = 1000000000000000000000000000000
1 << 31 = 10000000000000000000000000000000
1 << 32 = 100000000000000000000000000000000
1 << 33 = 1000000000000000000000000000000000
1 << 34 = 10000000000000000000000000000000000
1 << 35 = 100000000000000000000000000000000000
1 << 36 = 1000000000000000000000000000000000000
1 << 37 = 10000000000000000000000000000000000000
1 << 38 = 100000000000000000000000000000000000000
1 << 39 = 1000000000000000000000000000000000000000
1 << 40 = 10000000000000000000000000000000000000000
1 << 41 = 100000000000000000000000000000000000000000
1 << 42 = 1000000000000000000000000000000000000000000
1 << 43 = 10000000000000000000000000000000000000000000
1 << 44 = 100000000000000000000000000000000000000000000
1 << 45 = 1000000000000000000000000000000000000000000000
1 << 46 = 10000000000000000000000000000000000000000000000
1 << 47 = 100000000000000000000000000000000000000000000000
1 << 48 = 1000000000000000000000000000000000000000000000000
1 << 49 = 10000000000000000000000000000000000000000000000000
1 << 50 = 100000000000000000000000000000000000000000000000000
1 << 51 = 1000000000000000000000000000000000000000000000000000
1 << 52 = 10000000000000000000000000000000000000000000000000000
1 << 53 = 100000000000000000000000000000000000000000000000000000
1 << 54 = 1000000000000000000000000000000000000000000000000000000
1 << 55 = 10000000000000000000000000000000000000000000000000000000
1 << 56 = 100000000000000000000000000000000000000000000000000000000
1 << 57 = 1000000000000000000000000000000000000000000000000000000000
1 << 58 = 10000000000000000000000000000000000000000000000000000000000
1 << 59 = 100000000000000000000000000000000000000000000000000000000000
1 << 60 = 1000000000000000000000000000000000000000000000000000000000000
1 << 61 = 10000000000000000000000000000000000000000000000000000000000000
1 << 62 = 100000000000000000000000000000000000000000000000000000000000000
1 << 63 = 1000000000000000000000000000000000000000000000000000000000000000

第 3 章 ■ 执行数学运算

举个例子，考虑以下示例：将值 `1` 左移一位，重复 `64` 次，以及将值 1 左移 `64` 位。

```php
<?php
// 3_17.php

$bits = PHP_INT_SIZE * 8;
$v = 1;
for ($i = 0; $i < $bits; $i++) {
    $v = $v << 1;
}
$vv = 1 << $bits;
echo "$v, $vv\n";
```

在这种情况下，打印出的第一个值是 `0`，这是正确的；第二个值是 `1`，这是不正确的，因为在左移 `64` 次后，所有位都应该被设置为 `0`。

## 配方 3-7\. 浮点数取整

### 问题

由于浮点数本质是二进制内部表示，而大多数使用场景需要十进制表示，我们经常会得到带有许多不需要的小数位的数值。

### 解法

使用 `ceil()` 和 `floor()` 可以将浮点数向上或向下取整为最接近的整数，而 `round()` 函数用于将数字舍入到指定的精度。此外，`abs()` 函数用于通过清除符号位来获取一个数的绝对值。

### 工作原理

下一个示例展示了这些基本数学函数的用法。

```php
<?php
// 3_18.php

$a = -3.2556;
echo abs($a) . "\n"; // 3.2556
echo floor($a) . "\n"; // -4
echo floor(abs($a)) . "\n"; // 3
echo ceil($a) . "\n"; // -3
echo ceil(abs($a)) . "\n"; // 4
echo round($a, 2) . "\n"; // -3.26
```

注意 `floor()` 和 `ceil()` 在处理正值和负值时表现似乎有所不同。

第 3 章 ■ 执行数学运算

## 配方 3-8\. 生成随机数

### 问题

随机数常被用于游戏和其他程序中。从数据库中选取一组数据并以随机顺序呈现，可以用于生成对每位访客都不同的 HTML 页面。

### 解法

PHP 中提供了两组随机化函数。`rand()` 函数在两个指定值之间生成一个随机数，或者在 `0` 和 `getrandmax()` 返回的最大可用随机数之间生成。还可以通过播种随机数生成器来生成更随机的数字，这通过 `srand()` 函数完成。在生成随机数之前不再需要调用 `srand()`（或 `mt_srand()`），因为这会自动完成。

另一个随机数生成器叫做 `mt_rand()`，它基于具有已知特性的生成器，并且通常比 `rand()` 函数快得多。该实现被创建为 `rand()`、`srand()` 和 `getrandmax()` 函数的直接替代品，只需加 `mt_` 前缀即可。`mt` 来源于所使用的生成器名称——梅森旋转算法（Mersenne Twister）。

### 工作原理

下一个示例展示了如何生成一个介于 `0` 和最大随机值之间的随机数，以及如何生成两个边界之间的随机数。

```php
<?php
// 3_19.php

$r1 = mt_rand();
$rmax = mt_getrandmax();
$r2 = mt_rand(-5, 5);
echo "$r1\n$rmax\n$r2\n";
```

随机函数总是返回介于默认边界或提供给函数的参数边界之间的整数。提供边界可以生成负随机数。随机数可以是其中一个边界，也可以是两者之间的任意值，这意味着 `mt_rand(1, 5)` 可以产生值 `1`、`2`、`3`、`4` 和 `5`。生成一个介于 `1` 和 `2` 之间、精度为 `3` 位小数的随机数可以这样做：

```php
<?php
// 3_20.php

$r = mt_rand(0, 1000);
$v = 1 + $r / 1000;
echo "Random value between 1 and 2 is $v\n";
```

第 3 章 ■ 执行数学运算

## 配方 3-9\. 用对数函数表示比率

### 问题

两个数之间的比率通常用分贝刻度（`0` dB）表示。这是一个可以使用加法而非乘法来表示变化的刻度。例如，发射器的功率水平通常相对于 `1` mW 或 `0` dBm 的参考值给出。将功率加倍等同于增加 `3` dB，将功率减半等同于减去 `3` dB。

### 解法

将比率转换为分贝刻度需要使用对数函数。PHP 实现了基于自然常数 `e ~ 2.718282` 的自然对数函数 `log()`，以及使用基数为 `10` 的 `log10()` 函数。数字 `e` 被定义为一个名为 `M_E` 的常量。

### 工作原理

以分贝表示功率比率的公式定义为 `10*log10(P2/P1)`，其中 `P1` 和 `P2` 是功率水平。类似的公式用于以分贝表示电压和电流比率，公式为 `20*log10(V2/V1)`。在下一个示例中，我们计算 `1` W 相对于 `1` mW 的比率，并以分贝表示。通常使用 dBm 作为单位，以表明该值是相对于 `1` mW 的。

```php
<?php
// 3_21.php

$ref = 0.001; // 1 mW.
$p = 1.0; // 1 W
$r = $p / $ref;
$db = 10*log10($r);
echo $db . " dBm\n";
```

对数函数只对正数有效。计算 `0` 的对数会得到负无穷或 `–INF`。所有对数函数在参数为 `1` 时返回 `0`。最后，所有对数函数在提供底数时返回 `1`。例如 `log(e) = 1`，`log10(10) = 1` 等。

`log10()` 是一个便捷函数，因为 `log()` 函数接受第二个参数来定义底数。如果未向函数传递该参数，底数参数默认为 `M_E`。因此，调用 `10*log($r, 10);` 将提供与 `10*log10($r);` 相同的结果。

## 配方 3-10\. 计算未来价值

### 问题

计算本金按一定利率的未来价值需要使用指数函数。这些函数在 PHP 中是如何实现和使用的？
```


指数函数定义为基数乘幂，`pow()`函数用于计算`a`的`x`次幂的值。自然指数函数由自然数`e`定义，PHP 有一个专门的函数`exp()`来计算`e`的`x`次幂。该函数只接受一个参数，因为基数值固定为`e`。

# 第 3 章 ■ 执行数学运算

### 工作原理

假设你要计算一笔 1000 美元贷款、年利率 5%、每年计息次数分别为 1、2、4、12 次时的复利：

```php
<?php
// 3_22.php

$principal = 1000;
$interest = 0.05;
$periods = array(1, 2, 4, 12);

foreach ($periods as $p) {
    $ci = $principal * pow(1 + $interest / $p, $p);
    echo "Periods: $p, future value = $ci\n";
}
```

以下结果显示，即使年利率为 5%，一年后的未来价值也会因计息次数的不同而变化。

```
Periods: 1, future value = 1050
Periods: 2, future value = 1050.625
Periods: 4, future value = 1050.9453369141
Periods: 12, future value = 1051.1618978817
```

使用相同的公式并不断增加计息频率，我们可以计算出连续复利（即`e`）的值。

```php
<?php
// 3_23.php

$interest = 1;
$periods = array(1, 2, 4, 12, 52, 365, 8670, 525600, 31536000);

foreach ($periods as $p) {
    $ci = pow(1 + $interest / $p, $p);
    echo "Periods: $p, future value = $ci\n";
}
```

计息频率越高，结果越接近自然数`e`。

```
Periods: 1, future value = 2
Periods: 2, future value = 2.25
Periods: 4, future value = 2.44140625
Periods: 12, future value = 2.6130352902247
Periods: 52, future value = 2.6925969544372
Periods: 365, future value = 2.714567482022
Periods: 8670, future value = 2.7181250813746
Periods: 525600, future value = 2.7182792426664
Periods: 31536000, future value = 2.718281778469
```

该示例展示了当计息频率增加到每秒钟一次时的情况。即使进一步提高计息次数，结果也只会越来越接近自然指数`e`。

# 第 3 章 ■ 执行数学运算

## 配方 3-11：使用三角函数计算距离和方向

### 问题

你正在创建一个提供全国酒店列表的网站，希望提供一项服务，允许用户查找离给定位置最近的酒店。

### 解决方案

三角函数将三角形的角度与其边长相关联，通常用于模拟周期现象以及计算球面上两点之间的距离和方向，但它们还有许多其他用途。

在直角三角形中，有六种基本三角函数，对应六种可能的边比。

PHP 中所有三角函数操作的角度单位均为弧度而非角度。为了在弧度和角度之间进行转换，PHP 提供了`deg2rad()`和`rad2deg()`函数。

你可以使用 Google Maps API 查找列表中每家酒店的确切位置，或者通过邮政编码进行近似定位，执行数据库查询将邮政编码转换为经度和纬度。

当你知道两个位置的坐标后，可以假设地球近似为球形，使用三角函数来计算距离。

### 工作原理

地球半径约为 6271 公里或 3959 英里。这些数字被定义为常量，以便用户选择使用哪种单位。所有三角函数都适用于半径为 1 个单位的圆，这使得缩放至任意球体变得容易。

经度和纬度以浮点数形式提供，单位为角度，因此`Pos2Distance()`函数首先将这些值转换为 PHP 三角函数所使用的弧度。

```php
<?php
// 3_24.php

define('RADIUS_KM', 6371);
define('RADIUS_MILES', 3959);

function Pos2Distance($pos1, $pos2, $unit = RADIUS_KM, $precision = 0) {
    $lon1 = deg2rad($pos1['lon']);
    $lat1 = deg2rad($pos1['lat']);
```


$lon2 = deg2rad($pos2['lon']);

$lat2 = deg2rad($pos2['lat']);

$d = acos(sin($lat1)*sin($lat2) + cos($lat1)*cos($lat2)*cos($lon2-$lon1));

return round($d * $unit, $precision);

}

$p1 = array('lon' => -74.0059, 'lat' => 40.7128);

$p2 = array('lon' => -118.2437, 'lat' => 34.0522);

## 第 3 章 ■ 执行数学运算

echo "从纽约到洛杉矶的距离：" . Pos2Distance($p1, $p2) . " km\n";

从纽约到洛杉矶的距离：3936 km

通过类似的数学计算，可以计算从纽约到洛杉矶直线行进所需的方位角。

```php
<?php
// 3_25.php

function Bearing($pos1, $pos2) {
    $lon1 = deg2rad($pos1['lon']);
    $lat1 = deg2rad($pos1['lat']);
    $lon2 = deg2rad($pos2['lon']);
    $lat2 = deg2rad($pos2['lat']);

    $b = rad2deg(atan2(sin($lon2 - $lon1)*cos($lat2), cos($lat1)*sin($lat2) -
    sin($lat1)*cos($lat2)*cos($lon2 - $lon1)));
    return $b < 0 ? 360 + $b : $b;
}

$p1 = array('lon' => -74.0059, 'lat' => 40.7128);
$p2 = array('lon' => -118.2437, 'lat' => 34.0522);

echo "从纽约到洛杉矶的方位角：" . Bearing($p1, $p2) . "\n";
```

预期结果应在西南方向（225 度）附近，但由于地球是球体，实际结果略高于正西方向。

从纽约到洛杉矶的方位角：273.68719070573

## 技巧 3-12. 处理复数

### 问题

PHP 和许多其他编程语言并不原生支持复数。在计算电子电路的阻抗时，需要使用复数。

### 解决方案

PHP 不支持运算符重载，也没有对复数的原生支持，但仍然可以创建对象来处理复数运算的各种操作。

### 工作原理

复数包含两个部分，可以表示为一个具有实部和虚部的对象。它们通常写作 `r + i*j`，其中 `r` 是实部，`j` 是虚部，`i` 是-1 的平方根得到的虚数单位。下面的示例实现了加法和减法，以及一个用于在字符串上下文中处理对象的魔术函数。

```php
<?php
// 3_26.php

class Complex {
    public $re;
    public $im;

    function __construct($re, $im) {
        $this->re = $re;
        $this->im = $im;
    }

    static function Add(Complex $c1, Complex $c2) {
        return new Complex($c1->re + $c2->re, $c1->im + $c2->im);
    }

    static function Subtract(Complex $c1, Complex $c2) {
        return new Complex($c1->re - $c2->re, $c1->im - $c2->im);
    }

    function __toString() {
        return $this->re . " + " . $this->im . "i";
    }
}

$c1 = new Complex(1, -2);
$c2 = new Complex(3, 5);
$c3 = Complex::Add($c1, $c2);
$c4 = Complex::Subtract($c2, $c1);

echo "$c1\n$c2\n";
```

-1i  
8i

这是一个简单的复数运算实现，仅实现了加法和减法。如需更完整的实现，请查看 PEAR 模块 `Math_Complex`，位于 [`pear.php.net/package/Math_Complex/`](http://pear.php.net/package/Math_Complex/)。

## 数学扩展

通过安装 GNU MP (`gmp`) 和/或 Binary Calculator Math (`bcmath`) 扩展，可以处理超出所用 PHP 版本精度的大数。这些数字将以字符串形式表示。`gmp` 扩展用于任意长度的整数，而 `bcmath` 扩展用于任意精度的数字。

# 第 4 章  
## 处理数组

数组在 PHP 编程中扮演着核心角色。许多函数返回数组，并且数组常用于处理配置。基本数组是一个键值对列表，键可以是整数或字符串，值可以是任何变量类型，包括数组。

## 技巧 4-1. 创建数组

### 问题

在 PHP 中，数组被视为一种基本数据类型。无需调用任何函数或实例化任何对象，即可创建具有数组值的变量。

### 解决方案

PHP 使用语言结构 `array()` 来创建数组。如果将值 `array()` 赋给一个变量，就会创建一个空数组，即没有任何元素的数组。此外，还可以在括号内包含一个以逗号分隔的 `key => value` 对列表，以创建一个已经包含值的数组。如果定义中省略了键，PHP 将从 0 开始自动分配数值作为键。

如上所述，键可以是整数或字符串。如果使用了其他类型的键，PHP 会自动进行类型转换。浮点数和布尔值将被转换为整数。

`null` 值将被转换为空字符串，而有效的数值字符串（如 `"7"`）也会被转换为整数。字符串 `"07"` 不会被转换，因为 `07` 不是有效的十进制数。

数组中的键不必都是相同类型。值也是如此。它们可以混合使用，因为数组中的每个元素都像其他变量一样被对待，与其他元素或其他变量没有关联。

从 PHP 5.3 版本开始，可以使用简写语法来创建数组。这是通过方括号 `[]` 实现的，就像在 JavaScript 中一样。

要访问数组中的特定元素，需要知道其键。键值放在变量名后的方括号中，例如 `$b = $a['key'];`。这将把键 `'key'` 对应的元素值赋给变量 `$b`。如果该数组元素本身也是一个数组，可以通过添加第二组包含键值的方括号来访问该数组中的元素，例如 `$b = $a['key']['key2'];`。只要元素是数组，就可以重复此操作。

### 工作原理

下面的示例展示了六个不同数组的例子。

```php
<?php
// 4_1.php

$a = array();               // 空数组
$b = [];                    // 空数组
$c = array(1, 2, 3, 4, 5);
$d = array(5 => 'Orange', 'Apple', 'Banana', 'test' => 5);
$e = array(
    'host'     => 'localhost',
    'database' => 'orders',
    'user'     => 'root',
    'password' => 'secret',
);
$f = [
    'host'     => 'localhost',
    'database' => 'orders',
    'user'     => 'root',
    'password' => 'secret',
];
```

前两个数组是空数组，没有任何元素。第三个数组包含五个整数元素，键从 0 开始自动分配，到 4 结束。在第四个数组中，键是混合的，第一个键被定义为 5，因此后续的键将自动分配为 6 和 7，最后一个元素的键是字符串值。

最后两个数组是相同的，它们只是分别用短语法和长语法创建的，就像前两个数组一样。

创建数组时，元素按创建顺序存储。如果值是自动分配的，它们将从 0 或上一个使用的数值键开始递增。

```php
<?php
// 4_2.php

$a = [1, 2, 5=>3, 4, 5, 10 => 6];
echo "\$a contains\n";
foreach($a as $k => $v) {
    echo "$k = $v\n";
}

$b = [5 => 5, 4 => 4, 3 => 3, 2 => 2];
echo "\$b contains\n";
foreach($b as $k => $v) {
    echo "$k = $v\n";
}
```

这个例子将产生以下输出，显示数组将按照元素创建的顺序遍历，而不是按照键或值的顺序。

```
$a contains
0 = 1
1 = 2
5 = 3
6 = 4
7 = 5
10 = 6

$b contains
5 = 5
4 = 4
3 = 3
2 = 2
```

注意，所示的输出来自脚本的命令行执行。如果输出显示在浏览器中，则不会显示任何换行符。输出中没有 HTML，但 Web 服务器仍然发送默认的 `Content-Type`（`text/html`）。添加一个头部将 `Content-Type` 设置为 `text/plain` 可以解决这个问题。

如果元素的顺序很重要，则必须按所需顺序创建值，或者如本章后面所述，有许多可用于数组排序的函数。

## 技巧 4-2. 修改数组

### 问题



通过语言结构来创建数组并赋值是非常简单的。值可能并非直接来自程序员，并且在程序执行过程中它们可能需要被更改，因此有必要对数组中的元素进行添加、修改和删除操作。

### 解决方案

向数组添加元素有两种基本方式：要么使用特定的索引值，要么让 PHP 生成下一个可用的索引值。这两种方法都是通过在变量名后添加方括号来定位数组中的元素。

通过指定键来修改值是可行的，只需为数组中的元素赋予一个新值即可。旧值占用的内存会被自动释放。

删除元素是通过对要删除的特定元素使用 `unset()` 函数来完成的。如果在数组本身上使用 `unset()`，那么所有元素都会被删除。通常，变量可以通过将其赋值为 `null` 来取消定义。但这不适用于我们需要同时移除键和值的数组。一个数组元素的值是 `null` 或 `undefined` 是完全允许的。

### 工作原理

在接下来的例子中，我们使用一个 `for` 循环来遍历一组值，从而操作一个空数组并为其填充值。

```php
<?php
// 4_3.php

$a = array();

for ($i = 1; $i < 10; $i++) {
    $a[] = $i;
}

print_r($a);
```

这将产生以下输出：

```
Array
(
    [0] => 1
    [1] => 2
    [2] => 3
    [3] => 4
    [4] => 5
    [5] => 6
    [6] => 7
    [7] => 8
    [8] => 9
)
```

因为未提供键，所以键是自动分配的。在这种情况下，键将是 0 到 8，而值将是 1 到 9。如果需要特定的键，可以像下面修改后的示例那样进行赋值。

```php
<?php
// 4_4.php

$a = array();

for ($i = 1; $i < 10; $i++) {
    $a[$i] = $i;
}

print_r($a);
```

如果该键已存在于数组中，那么它将用于赋值。该值会被替换，且不会产生任何警告。

```php
<?php
// 4_5.php

$a = array(1 => 15, 10 => 20);

for ($i = 1; $i < 10; $i++) {
    $a[$i] = $i;
}

print_r($a);
```

在这个例子中，数组被初始化为包含两个`键 => 值`对。第一个键值对会被循环中的第一次赋值所替换，但最后一个键值对将保持不变。键的顺序将是 1, 10, 2, 3, 4, 5, 6, 7, 8, 9，因为这是它们被创建的顺序。

这里示例中使用的数组非常简单，与其使用 `for` 循环来填充这些数组，不如像下一个示例那样使用 `range()` 函数更为高效。`range()` 函数接受两个或三个参数。可选的第三个参数定义了所创建值之间的步长。如果省略，步长将为 1。

```php
<?php
// 4_6.php

$a = range(1, 9);

print_r($a);
```

从数组中删除元素是通过 `unset()` 函数完成的。

```php
<?php
// 4_7.php

$a = range(1, 9);

unset($a[4]);

print_r($a);
```

```
Array
(
    [0] => 1
    [1] => 2
    [2] => 3
    [3] => 4
    [5] => 6
    [6] => 7
    [7] => 8
    [8] => 9
)
```

### 技巧 4-3. 数组相加

### 问题

数字可以相加生成新值，字符串可以拼接成更长的字符串，但如果两个数组相加或合并会发生什么？

### 解决方案

有许多内置函数可以处理将两个数组合并成一个数组的情况，甚至可以使用 `+` 运算符来合并两个数组。`+` 运算符的工作原理是合并两个数组中的值，创建它们的并集。考虑 `$b = $a1 + $a2;` 这种情况，其中 `$a1` 和 `$a2` 都是数组。

结果数组 (`$b`) 将包含 `$a1` 中的所有元素，以及 `$a2` 中所有在 `$a1` 里不存在的键所对应的元素。如果两个数组使用了相同的键，则只会包含 `$a1`（即 `+` 运算符左侧的数组）中的元素。给定相同的两个数组，以相反的顺序相加可能会得到不同的结果，这取决于两个数组的键：`$c = $a2 + $a1;`。优先使用左侧数组的键这一规则不依赖于键的类型。



合并两个或多个数组可以使用`array_merge()`函数。此函数的行为与`+`运算符略有不同，它会用右侧数组中相同键的值覆盖左侧数组的对应值，并且对于数值键，它会追加元素而非覆盖。

### 工作原理

以下示例展示了`+`运算符和`array_merge()`函数在两个简单数组上的不同行为：

```php
<?php
// 4_8.php

$a1 = array('a', 'b');

$a2 = array('c', 'd');

$b1 = $a1 + $a2;

$b2 = $a2 + $a1;

$c1 = array_merge($a1, $a2);

$c2 = array_merge($a2, $a1);

print_r($b1);

print_r($b2);

print_r($c1);

print_r($c2);
```

两个数组`$a1`和`$a2`都有两个数值键（0, 1）。当使用`+`运算符相加时，会使用左侧数组对应键的值，因此结果数组仅保留相同的两个键。

当使用`array_merge()`函数合并数组时，结果数组将有四个键（0, 1, 2, 3），且元素的顺序取决于它们作为参数传递给`array_merge()`函数的顺序。脚本的输出如下：

```
Array
(
    [0] => a
    [1] => b
)
Array
(
    [0] => c
    [1] => d
)
Array
(
    [0] => a
    [1] => b
    [2] => c
    [3] => d
)
Array
(
    [0] => c
    [1] => d
    [2] => a
    [3] => b
)
```

将键从数值型改为字符串型会略微改变行为，如下一个示例所示，其中键（0, 1）被改为字符串（'x', 'y'）。

```php
<?php
// 4_9.php

$a1 = array('x' => 'a', 'y' => 'b');

$a2 = array('x' => 'c', 'y' => 'd');

$b1 = $a1 + $a2;

$b2 = $a2 + $a1;

$c1 = array_merge($a1, $a2);

$c2 = array_merge($a2, $a1);

print_r($b1);

print_r($b2);

print_r($c1);

print_r($c2);
```

相同的逻辑仍然适用，但由于没有数值键，结果数组将只有两个键。但使用`+`运算符会保留左侧数组的值，而`array_merge()`函数会覆盖相同键的值，如下所示：

```
Array
(
    [x] => a
    [y] => b
)
Array
(
    [x] => c
    [y] => d
)
Array
(
    [x] => c
    [y] => d
)
Array
(
    [x] => a
    [y] => b
)
```

如果数组之间没有键重叠，结果数组将包含两个数组的所有值，并且这两种合并数组的方法将得到相同的结果。

```php
<?php
// 4_10.php

$a1 = array('a', 'b');

$a2 = array('x' => 'c', 'y' => 'd');

$b1 = $a1 + $a2;

$b2 = $a2 + $a1;

$c1 = array_merge($a1, $a2);

$c2 = array_merge($a2, $a1);

print_r($b1);

print_r($b2);

print_r($c1);

print_r($c2);
```

在此情况下，结果数组都将有四个键，并按照添加顺序显示值。

```
Array
(
    [0] => a
    [1] => b
    [x] => c
    [y] => d
)
Array
(
    [x] => c
    [y] => d
    [0] => a
    [1] => b
)
Array
(
    [0] => a
    [1] => b
    [x] => c
    [y] => d
)
Array
(
    [x] => c
    [y] => d
    [0] => a
    [1] => b
)
```

除了添加或合并数组，我们还可以查找两个或多个数组的交集。  
`array_intersect()`函数会返回一个数组，其中包含存在于第一个参数以及所有后续参数中的所有值。该函数忽略所有键。例如，如果第一个数组中包含`'x' => 'c'`，第二个数组中包含`'y' => 'c'`，则值`'c'`会出现在结果中。

```php
<?php
// 4_11.php

$a1 = array('x' => 'orange', 'y' => 'grape', 'z' => 'banana');
$a2 = array('grape', 'banana');

$b1 = array_intersect($a1, $a2);

$b2 = array_intersect($a2, $a1);

print_r($b1);

print_r($b2);
```

在此示例中，我们展示了返回的数据如何保留第一个数组中的键值，并且仅保留所有后续数组中都存在的值。

```
Array
(
    [y] => grape
    [z] => banana
)
Array
(
    [0] => grape
    [1] => banana
)
```

## Recipe 4-4\. 数组的数组

### 问题

大多数数据库以行和列的概念工作。SQL 数据库使用查询来选择数据行，其中每一行都有完全相同的一组列但值不同。如何用数组来表示数据库查询的结果？

### 解决方案



# PHP 数组操作

大多数 PHP 数据库 API 都提供了一个函数，可以将单行数据作为索引数组或关联数组获取。如果执行一个查询来从表中选取所有行和一组定义的列，很容易创建一个结构来遍历所有行，并创建一个数组的数组。外部数组对应行，而数组中的每个元素都是一个包含所有列的数组。

### 工作原理

在本例中，我们使用一个名为`person`的表，它有三列：`first_name`、`last_name`和`date_of_birth`。该表包含四行数据，分别对应披头士乐队的四位成员。

```sql
-- 4_12.sql

create database samples;

use samples;

create table person (
    first_name varchar(50),
    last_name varchar(50),
    date_of_birth int
);

insert into person (first_name, last_name, date_of_birth)
values ('John', 'Lenon', -922406400);

insert into person (first_name, last_name, date_of_birth)
values ('Paul', 'McCartney', -869097600);

insert into person (first_name, last_name, date_of_birth)
values ('Ringo', 'Star', -930528000);

insert into person (first_name, last_name, date_of_birth)
values ('George', 'Harrison', -847324800);
```

查询此表并将所有值存储在数组中的代码类似如下：

```php
<?php
// 4_12.php

$con = mysqli_connect("127.0.0.1", "root", "secret", "book");
$rs = mysqli_query($con, "select * from person");

if ($rs) {
    $persons = array();
    while($row = mysqli_fetch_assoc($rs)) {
        $persons[] = $row;
    }
    mysqli_free_result($rs);
}

mysqli_close($con);
print_r($persons);
```

`date_of_birth`列用 Unix 时间戳表示。这些值为负数，因为它们表示 1970 年 1 月 1 日之前的秒数。此示例的输出如下所示：

```
Array
(
    [0] => Array
        (
            [first_name] => John
            [last_name] => Lenon
            [date_of_birth] => -922406400
        )

    [1] => Array
        (
            [first_name] => Paul
            [last_name] => McCartney
            [date_of_birth] => -869097600
        )

    [2] => Array
        (
            [first_name] => Ringo
            [last_name] => Star
            [date_of_birth] => -930528000
        )

    [3] => Array
        (
            [first_name] => George
            [last_name] => Harrison
            [date_of_birth] => -847324800
        )
)
```

这展示了一个两级数组结构，其中外部数组代表选定的行，每个内部数组代表一条记录，所有内部数组都显示相同的三个键值。外部数组使用数字键，而内部数组使用字符串键，键值代表查询的列名。如果使用`mysqli_fetch_row()`函数而不是`mysqli_fetch_assoc()`函数，内部数组也会使用数字键，键的顺序与查询字符串中列名的顺序一致。这两个函数都可以多次调用。只要结果集中还有数据，函数就会返回一行。当结果中没有更多行时，函数将返回`NULL`。

## 配方 4-5：遍历数组

### 问题

访问数组的单个元素和值需要了解键名。

### 解决方案

有几种方法可以在使用或不使用键名的情况下遍历数组。如果所有键都是数字且从 0 开始按顺序排列，则可以使用简单的`for`循环来遍历数组。如果值顺序混乱或使用数字和字符串键的混合，我们可以使用`each()`函数返回当前索引处的值。索引是一个内部指针，用于跟踪数组中的当前元素。每次调用`each()`函数时，都会返回当前元素，并且内部指针会递增。如果内部指针递增到最后一个元素之上，`each()`函数将返回`false`，表示没有更多元素。如果需要再次遍历同一个数组，必须调用`reset()`函数将内部指针重置回数组的开头。

除了`each()`和`reset()`函数之外，还有函数可以返回数组的当前元素、下一个元素、上一个元素、最后一个元素以及当前元素的键。这些函数分别是`current()`、`next()`、`previous()`、`key()`和`end()`。`current()`和`key()`函数不会改变内部指针。所有其他函数都会改变内部指针。这些函数的使用通常与`while`循环相关联。

如果你不关心内部指针，并且希望多次遍历数组并轻松访问键和值，最简单的方法是使用`foreach()`语言结构。

### 工作原理

在本例中，我们使用上一个示例中的数组来创建格式化字符串列表，显示披头士乐队每位成员的全名和出生日期。我们使用所有三种迭代方法。

```php
<?php
// 4_13.php

$con = mysqli_connect("127.0.0.1", "root", "secret", "book");
$rs = mysqli_query($con, "select * from person");

if ($rs) {
    $persons = array();
    while($row = mysqli_fetch_assoc($rs)) {
        $persons[] = $row;
    }
    mysqli_free_result($rs);
}

mysqli_close($con);

for ($i = 0; $i < sizeof($persons); $i++) {
    echo "{$persons[$i]['first_name']} {$persons[$i]['last_name']} " .
         gmdate("M j Y", $persons[$i]['date_of_birth']) . "\n";
}
```

MySQL 查询的结果是一个数组的数组，因此为了访问姓名和生日，我们需要为第一级和第二级数组提供键。即使使用`each()`函数也是如此，因为它返回一个包含四个键的数组。键`0`和`1`指向键和值，此外我们还得到了键`'key'`和`'value'`。

```php
<?php
// 4_14.php

$con = mysqli_connect("127.0.0.1", "root", "secret", "book");
$rs = mysqli_query($con, "select * from person");

if ($rs) {
    $persons = array();
    while($row = mysqli_fetch_assoc($rs)) {
        $persons[] = $row;
    }
    mysqli_free_result($rs);
}

mysqli_close($con);

reset($persons);

while ($person = each($persons)) {
    echo "{$person['value']['first_name']} {$person['value']['last_name']} " .
         gmdate("M j Y", $person['value']['date_of_birth']) . "\n";
}
```

最后一种方法是使用`foreach()`结构。此函数将遍历数组的所有元素，且无需记住重置内部指针。我们还可以直接操作键和元素，因此代码变得稍微简单一些。

```php
<?php
// 4_15.php

$con = mysqli_connect("127.0.0.1", "root", "secret", "book");
$rs = mysqli_query($con, "select * from person");

if ($rs) {
    $persons = array();
    while($row = mysqli_fetch_assoc($rs)) {
        $persons[] = $row;
    }
    mysqli_free_result($rs);
}

mysqli_close($con);

foreach ($persons as $person) {
    echo "{$person['first_name']} {$person['last_name']} " .
         gmdate("M j Y", $person['date_of_birth']) . "\n";
}
```

如果需要每个元素的键，我们可以将语句改为`foreach ($persons as $key => $person)`；如果需要更新数组中的值，我们可以将其改为`foreach ($persons as $key => &$person)`，让`$person`成为数组中每个元素的引用，这样如果为`$person['date_of_birth']`赋予新值，就会更新数组本身，而不是元素的副本。此外，我们还可以通过为`$person['new_key']`赋值来向数组添加新值。

## 配方 4-6：排序数组

### 问题

正如本章开头所述，数组中的元素将按照添加的顺序出现。遍历数组时，键和值会按照它们在数组中出现的顺序处理，而不是按照索引值递增或字母顺序的逻辑顺序处理。



有多种方法可以对数组进行排序。可以按键排序，也可以按值排序，并且可以按升序或降序排列。按值排序通过函数 `sort()` 和 `rsort()` 实现。这两个函数都会返回一个数组，其中键被替换为从 0 开始的数值。任何字符串键或键之间的间隔都会丢失。如果你想保留键值关联，可以使用 `asort()` 或 `arsort()` 函数，它们会移动值的位置，但保留键值对应关系。

也可以使用 `ksort()` 和 `krsort()` 函数，按照键的值对数组进行排序。

### 工作原理

最简单的数组是使用数值键来存储一些值的数组。这种数组可以按值的正序或逆序排序。

```php
// 4_16.php

$fruit = array('pear', 'apple', 'orange', 'banana', 'kiwi');

sort($fruit);

print_r($fruit);

rsort($fruit);

print_r($fruit);
```

请注意，这些排序函数直接作用于数组变量，因为它是通过引用传递的。此示例将输出以下内容：

```
Array
(
    [0] => apple
    [1] => banana
    [2] => kiwi
    [3] => orange
    [4] => pear
)
Array
(
    [0] => pear
    [1] => orange
    [2] => kiwi
    [3] => banana
    [4] => apple
)
```

在这个例子中，所有的值都是小写字符串。如果字符串的大小写混合，排序将根据 ASCII 表进行，其中所有大写字母都在小写字母之前，这样排序结果看起来就会不正确。排序函数允许使用第二个参数来指示进行不区分大小写的字符串排序，如下一个示例所示。

```php
// 4_17.php

$fruit = array('pear', 'apple', 'Orange', 'Banana', 'kiwi');

sort($fruit);

print_r($fruit);

sort($fruit, SORT_FLAG_CASE | SORT_STRING);

print_r($fruit);
```

此脚本的输出将显示数组首先按区分大小写的方式排序，所有以大写字母开头的水果排在最前面，然后是不区分大小写的排序结果。

```
Array
(
    [0] => Banana
    [1] => Orange
    [2] => apple
    [3] => kiwi
    [4] => pear
)
Array
(
    [0] => apple
    [1] => Banana
    [2] => kiwi
    [3] => Orange
    [4] => pear
)
```

排序的另一个复杂点在于字符串包含数字时。人们的期望是 `item1`、`item2` 排在 `item11` 和 `item12` 之前，但由于排序是按字符逐个进行的，计算机将把这些项排序为 `item1`、`item11`、`item12` 和 `item2`。为了解决这个问题，可以在第二个参数中添加一个额外的标志 `SORT_NATURAL`。

```php
// 4_18.php

$items = array('item1', 'Item11', 'Item12', 'item2', 'item20', 'item21');

sort($items);

print_r($items);

sort($items, SORT_FLAG_CASE | SORT_NATURAL | SORT_STRING);

print_r($items);
```

该数组首先使用标准排序函数进行排序，导致以大写字符开头的元素出现在最前面。添加不区分大小写和自然排序的标志后，数组将按预期方式排序。

```
Array
(
    [0] => Item11
    [1] => Item12
    [2] => item1
    [3] => item2
    [4] => item20
    [5] => item21
)
Array
(
    [0] => item1
    [1] => item2
    [2] => Item11
    [3] => Item12
    [4] => item20
    [5] => item21
)
```

除了向 `sort()` 函数使用标志之外，也可以使用 `natsort()` 和 `natcasesort()` 来达到同样的效果。

对使用字符串作为键的数组进行排序，如果使用 `sort()` 或 `rsort()`，键将会丢失。为了保留字符串键，我们必须使用 `asort()` 和 `arsort()` 函数。如果我们扩展之前的 fruit 示例，将水果的值变成键，并添加库存值（每种水果的库存数量），示例如下所示：

```php
// 4_19.php

$fruit = array('pear' => 10, 'apple' => 3, 'orange' => 15, 'banana' => 5, 'kiwi' => 2);

sort($fruit);

print_r($fruit);

$fruit = array('pear' => 10, 'apple' => 3, 'orange' => 15, 'banana' => 5, 'kiwi' => 2);

asort($fruit);

print_r($fruit);
```



```markdown
注意，数组`$fruit`被定义了两次，因为`sort()`函数会用数字键替换原键。对数组排序会导致项目按库存数量排序，但使用`sort()`函数会丢失键，因此我们不再知道每个项目是什么。在示例的后半部分，我们使用了`asort()`，它保留键，我们仍然能区分项目内容。

```
Array
(
    [0] => 2
    [1] => 3
    [2] => 5
    [3] => 10
    [4] => 15
)
Array
(
    [kiwi] => 2
    [apple] => 3
    [banana] => 5
    [pear] => 10
    [orange] => 15
)
```

如果我们希望按字母顺序排序而不是按库存数量排序，可以使用`ksort()`函数按键排序。

```php
<?php
// 4_20.php
$fruit = array('pear' => 10, 'apple' => 3, 'orange' => 15, 'banana' => 5, 'kiwi' => 2);
ksort($fruit);
print_r($fruit);
```

第 4 章 ■ 数组操作

元素将显示如下：

```
Array
(
    [apple] => 3
    [banana] => 5
    [kiwi] => 2
    [orange] => 15
    [pear] => 10
)
```

到目前为止，排序都是针对简单数组进行的。如果我们回到从数据库检索数据的示例——获取行和列并尝试直接对结果数组排序，则会得到基于数组比较的结果。这是通过比较每个数组中的所有元素来确定哪个“更大”，然后据此设置元素的顺序。

```php
<?php
// 4_21.php
$con = mysqli_connect("127.0.0.1", "root", "Wbp2Theworld", "book");
$rs = mysqli_query($con, "select * from person");
if ($rs) {
    $persons = array();
    while($row = mysqli_fetch_assoc($rs)) {
        $persons[] = $row;
    }
    mysqli_free_result($rs);
}
mysqli_close($con);
sort($persons);
print_r($persons);
```

在此示例中，我们在`4_9.php`的代码中添加了对`sort()`函数的调用，记录不再按 John、Paul、Ringo、George 的顺序显示，而是根据每条记录的第一个元素按字母顺序显示。

```
Array
(
    [0] => Array
        (
            [first_name] => George
            [last_name] => Harrison
            [date_of_birth] => -847324800
        )
    [1] => Array
        (
            [first_name] => John
            [last_name] => Lenon
            [date_of_birth] => -922406400
        )
    [2] => Array
        (
            [first_name] => Paul
            [last_name] => McCartney
            [date_of_birth] => -869097600
        )
    [3] => Array
        (
            [first_name] => Ringo
            [last_name] => Star
            [date_of_birth] => -930528000
        )
)
```

如果我们希望按姓氏列排序，可以更改查询中的列顺序。

```php
<?php
// 4_22.php
$con = mysqli_connect("127.0.0.1", "root", "Wbp2Theworld", "book");
$rs = mysqli_query($con, "select last_name, first_name, date_of_birth from person");
if ($rs) {
    $persons = array();
    while($row = mysqli_fetch_assoc($rs)) {
        $persons[] = $row;
    }
    mysqli_free_result($rs);
}
mysqli_close($con);
sort($persons);
print_r($persons);
```

在此例中，列的顺序已改变，因此当调用`sort()`函数时，比较将从姓氏列开始，从而产生以下结果。

第 4 章 ■ 数组操作

```
Array
(
    [0] => Array
        (
            [last_name] => Harrison
            [first_name] => George
            [date_of_birth] => -847324800
        )
    [1] => Array
        (
            [last_name] => Lenon
            [first_name] => John
            [date_of_birth] => -922406400
        )
    [2] => Array
        (
            [last_name] => McCartney
            [first_name] => Paul
            [date_of_birth] => -869097600
        )
    [3] => Array
        (
            [last_name] => Star
            [first_name] => Ringo
            [date_of_birth] => -930528000
        )
)
```

`sort()`函数有一个可选参数`sort_flags`，用于定义排序方式。默认值是`SORT_REGULAR`，它会使用标准的 PHP 比较函数来确定一个值是否大于或小于另一个值。还有另外四个标志可以使用：

- `SORT_NUMERIC`：将值转换为数字。当值为数字字符串时很有用。
- `SORT_STRING`：在比较前将值转换为字符串。
- `SORT_LOCALE_STRING`：转换为字符串，但使用系统的区域设置进行排序。
- `SORT_NATURAL`：使用字符串比较方法，但针对数字字符串进行自然排序。
```



此外，还有一个标志可使排序不区分大小写。该标志可与任何其他标志组合使用。

当然，使用数据库的排序功能并按应使用顺序返回记录效率更高，但有时可能需要不同的顺序。更改查询来改变列的顺序可能会影响呈现效果，因此这种方法也可能不是最佳选择。

这时，`usort()` 函数就派上了用场。`usort()` 函数接受两个参数：第一个是要排序的数组，第二个是用户自定义回调函数的名称，该函数用于比较数组中的两个元素。如果我们想扩展示例 `3_9.php`，并定义按名字、姓氏或出生日期对人数组进行排序的函数，代码可能如下所示。

```php
<?php

// 4_23.php

function SortFirstName($a, $b) {

if ($a['first_name'] == $b['first_name']) {

return 0;

}

elseif ($a['first_name'] > $b['first_name']) {

return 1;

}

else {

return -1;

}

}

function SortLastName($a, $b) {

if ($a['last_name'] == $b['last_name']) {

return 0;

}

elseif ($a['last_name'] > $b['last_name']) {

return 1;

}

else {

return -1;

}

}

function SortDOB($a, $b) {

if ($a['date_of_birth'] == $b['date_of_birth']) {

return 0;

}

elseif ($a['date_of_birth'] > $b['date_of_birth']) {

return 1;

}

else {

return -1;

}

}

$con = mysqli_connect("127.0.0.1", "root", "Wbp2Theworld", "book"); $rs = mysqli_query($con, "select * from person");

if ($rs) {

$persons = array();

while($row = mysqli_fetch_assoc($rs)) {

$persons[] = $row;

}

mysqli_free_result($rs);

}

mysqli_close($con);

usort($persons, 'SortFirstName');

print_r($persons);

usort($persons, 'SortLastName');

print_r($persons);

usort($persons, 'SortDOB');

print_r($persons);
```

在本例中，我们定义了三个回调函数，每个函数用于按特定列排序。代码调用了每个排序函数，输出如下所示。

```
Array
(
    [0] => Array
        (
            [first_name] => George
            [last_name] => Harrison
            [date_of_birth] => -847324800
        )
    [1] => Array
        (
            [first_name] => John
            [last_name] => Lenon
            [date_of_birth] => -922406400
        )
    [2] => Array
        (
            [first_name] => Paul
            [last_name] => McCartney
            [date_of_birth] => -869097600
        )
    [3] => Array
        (
            [first_name] => Ringo
            [last_name] => Star
            [date_of_birth] => -930528000
        )
)
Array
(
    [0] => Array
        (
            [first_name] => George
            [last_name] => Harrison
            [date_of_birth] => -847324800
        )
    [1] => Array
        (
            [first_name] => John
            [last_name] => Lenon
            [date_of_birth] => -922406400
        )
    [2] => Array
        (
            [first_name] => Paul
            [last_name] => McCartney
            [date_of_birth] => -869097600
        )
    [3] => Array
        (
            [first_name] => Ringo
            [last_name] => Star
            [date_of_birth] => -930528000
        )
)
Array
(
    [0] => Array
        (
            [first_name] => Ringo
            [last_name] => Star
            [date_of_birth] => -930528000
        )
    [1] => Array
        (
            [first_name] => John
            [last_name] => Lenon
            [date_of_birth] => -922406400
        )
    [2] => Array
        (
            [first_name] => Paul
            [last_name] => McCartney
            [date_of_birth] => -869097600
        )
    [3] => Array
        (
            [first_name] => George
            [last_name] => Harrison
            [date_of_birth] => -847324800
        )
)
```

按名字或姓氏排序对于披头士乐队的四位成员来说没有区别，但按出生日期排序则显示了不同的记录顺序。

## 食谱 4-7.  将数组用作栈

### 问题
栈是一种常用结构，元素可以从栈的开口端添加和移除。栈被视为后进先出（LIFO）结构。

### 解决方案
通过使用 `array_pop()`、`array_push()`、`array_shift()` 和 `array_unshift()` 函数，可以将数组用作栈。`Shift` 和 `unshift` 作用于数组的开头，而 `push` 和 `pop` 作用于数组的末尾。这些函数将返回数组的第一个或最后一个元素。如果数组为空或变量不是数组，这些函数将返回 `NULL`。

### 工作原理



函数 `array_push()` 会将一个元素添加到数组的末尾。键名将是下一个可用的数值。如果数组为空，第一个键将是 `0`。 函数 `array_pop()` 会移除数组中的最后一个值并将其返回，以便赋值给一个变量或在比较中使用。

```php
<?php
// 4_24.php

$stack = array();

array_push($stack, 'orange');
array_push($stack, 'apple');
array_push($stack, 'banana');

print_r($stack);

$fruit = array_pop($stack);

print_r($stack);
```

在此例中，我们向栈中添加了三种水果并打印了数组的所有值。然后我们使用 `array_pop()` 移除最后添加的元素，并再次打印数组内容。此时数组中只剩下两个值。

```
Array
(
    [0] => orange
    [1] => apple
    [2] => banana
)
Array
(
    [0] => orange
    [1] => apple
)
```

我们可以通过使用 `array_unshift()` 和 `array_shift()` 实现相同的功能。

```php
<?php
// 4_25.php

$stack = array();

array_unshift($stack, 'orange');
array_unshift($stack, 'apple');
array_unshift($stack, 'banana');

print_r($stack);

$fruit = array_shift($stack);

print_r($stack);
```

水果会以相反顺序显示，并且每次添加或移除元素时，现有元素的所有键都必须向上或向下移动，以便为新元素腾出空间。这将导致 `shift` 和 `unshift` 函数的运行速度比 `push` 和 `pop` 稍慢。

```
Array
(
    [0] => banana
    [1] => apple
    [2] => orange
)
Array
(
    [0] => apple
    [1] => orange
)
```

## 配方 4-8：数组切片与拼接

### 问题

你需要获取数组的一部分，或者将一个数组插入到另一个数组的中间。

### 解决方案

有两个函数可用于处理这些操作。`array_slice()` 函数可用于剪切数组的一部分并将其赋值给一个新数组。`array_splice()` 函数可用于将一个数组插入到另一个数组的任意位置。

### 工作原理

`array_slice()` 函数接受两到四个参数。第一个是源数组，第二个是偏移量。这是从数组开头跳过的元素数量。可选的第三个参数是切片的长度或要复制的元素数量。最后的可选第四个参数定义是否保留数值数组键。默认为 `false`。下面的示例展示了如何从一个大数组中复制一个包含两个元素的小切片。该示例还演示了保留键与不保留键之间的区别。

```php
<?php
// 4_26.php

$a = array(
    'apple',
    'orange',
    'banana',
    'animal' => 'horse',
    'vechicle' => 'truck'
);

print_r(array_slice($a, 2, 2));
print_r(array_slice($a, 2, 2, true));
```

输出如下：

```
Array
(
    [0] => banana
    [animal] => horse
)
Array
(
    [2] => banana
    [animal] => horse
)
```

这是一个微小的差异，但在输出的第一部分中，`banana` 的索引从 `2` 变为了 `0`，而在第二部分中，其位置/键被保留了。

`array_splice()` 函数也接受四个参数。第一个是源数组，第二个是偏移量，第三个是长度，第四个是要插入的数组。该函数可以三种不同的方式使用：第一种是删除数组的一部分，第二种是将一个新数组插入到数组中，第三种是前两种操作的组合。

第一个参数（源数组）是通过引用传递的。这会导致数组的值被该函数更改。此外，该函数会返回一个包含所有被替换元素的数组。

第二个参数可以是正数或负数。如果为正，则从数组开头计算偏移量；如果为负，则从数组末尾开始计算。



如果省略了 `length` 参数，则会移除从 `offset` 到数组末尾的所有元素。如果该参数为正数，函数将从 `offset` 位置开始移除指定数量的元素。如果 `length` 为负数，则通过从数组末尾倒数相应数量的元素来确定移除元素的结束位置。这些计算方式与 `substr()` 函数类似，该函数的 `offset` 和 `length` 参数均可取正值和负值。

以下示例中，有两个数组正在进行拼接操作。由于 `array_splice()` 函数直接作用于源数组，因此在调用该函数之前需要创建数组的副本（仅因为所有三个示例中使用了同一个数组）。

```php
// 4_27.php

$a = array(
    'apple',
    'orange',
    'banana',
    'animal' => 'horse',
    'vechicle' => 'truck'
);

$b = array(
    'kiwi',
    'grape'
);

// 从数组中移除除两个元素外的所有元素
$a1 = $a;
$r = array_splice($a1, 2, 2);
print_r($r);
print_r($a1);

// 将 $b 插入到 $a 中
$a1 = $a;
$r = array_splice($a1, 2, 0, $b);
print_r($r);
print_r($a1);

// 移除一个元素并插入 $b
$a1 = $a;
$r = array_splice($a1, 2, 1, $b);
print_r($r);
print_r($a1);
```

此示例的输出将得到以下六个数组：

```
Array
(
    [0] => banana
    [animal] => horse
)
Array
(
    [0] => apple
    [1] => orange
    [vechicle] => truck
)
Array
(
)
Array
(
    [0] => apple
    [1] => orange
    [2] => kiwi
    [3] => grape
    [4] => banana
    [animal] => horse
    [vechicle] => truck
)
Array
(
    [0] => banana
)
Array
(
    [0] => apple
    [1] => orange
    [2] => kiwi
    [3] => grape
    [animal] => horse
    [vechicle] => truck
)
```

## 食谱 4-9\. 调试数组

### 问题

数组会直接作为输出打印，因此要了解超全局变量或由数据库查询或 API 调用生成的数组变量的确切值，就需要使用其他输出方法。

### 解决方案

PHP 提供了三个使数组变量调试变得简单的函数：`print_r()`、`var_dump()` 和 `var_export()`。`print_r()` 函数会递归遍历数组，并以清晰可读的方式打印出键和值。`var_dump()` 和 `var_export()` 函数还会打印元素的数据类型和大小信息。

### 工作原理

在本例中，我们使用一个包含混合整数键和字符串键的数组，来演示使用三种输出方法时调试信息的呈现方式。

```php
// 4_28.php

$a = array(
    'apple',
    'orange',
    'banana',
    'animal' => 'horse',
    'vechicle' => 'truck'
);

print_r($a);
var_dump($a);
var_export($a);
```

此代码的输出如下：

```
Array
(
    [0] => apple
    [1] => orange
    [2] => banana
    [animal] => horse
    [vechicle] => truck
)
array(5) {
  [0]=>
  string(5) "apple"
  [1]=>
  string(6) "orange"
  [2]=>
  string(6) "banana"
  ["animal"]=>
  string(5) "horse"
  ["vechicle"]=>
  string(5) "truck"
}
array (
  0 => 'apple',
  1 => 'orange',
  2 => 'banana',
  'animal' => 'horse',
  'vechicle' => 'truck',
)
```

第一个和第三个示例最易读，但它们不包含任何类型信息。只有 `var_export()` 函数生成的输出可以用于 `eval` 语句，或直接写入 PHP 脚本文件以供稍后包含或执行。这在编写配置文件时非常有用。

请注意，在浏览器环境中使用这些输出时，应将其包含在 `<pre>..</pre>` 标签中，以确保格式如上所示。

## 食谱 4-10\. 存储数组

### 问题

数组在不同语言中的定义方式不同，并且通常不被文件系统或数据库原生支持，那么如何存储数组的值呢？

### 解决方案

通过使用 `serialize()` 和 `unserialize()` 或 `json_encode()` 和 `json_decode()`，可以将任何数组（以及其他类型的 PHP 变量）字符串化，以便写入文件或作为值保存在数据库的 `varchar` 列中。`json_*` 函数对于将数组转换为可由浏览器中 JavaScript 引擎解释的结构也很有用。



JSON 是 JavaScript Object Notation（JavaScript 对象表示法）的缩写，常用于在 JavaScript 代码中编写和交换数据，或在 PHP 和 JavaScript 等其他系统之间交换数据。它也可以作为 PHP 中的一种序列化格式使用。

### 工作原理

首先，考虑一个序列化数组的格式，以及同一个数组经过 JSON 编码后的字符串。两者都是字符串值，包含足够的信息来描述每个元素，以便其可以被重新生成。

```php
<?php

// 4_29.php

$a = array(

'apple',

'orange',

'banana',

'animal' => 'horse',

'vechicle' => 'truck'

);

echo serialize($a) . "\n";

echo json_encode($a) . "\n";

```
两个输出如下所示：

```
a:5:{i:0;s:5:"apple";i:1;s:6:"orange";i:2;s:6:"banana";s:6:"animal";s:5:"horse";s:8:"vechi cle";s:5:"truck";}
{"0":"apple","1":"orange","2":"banana","animal":"horse","vechicle":"truck"}
```

在序列化版本中，我们看到关于类型和长度的信息。`a:5:`表示一个包含五个元素的数组，`i:`表示整数，`s:6:`表示一个包含六个字符的字符串。在 64 位和 32 位版本的 PHP 之间使用序列化数据可能导致溢出，因为整数在 64 位版本中可以存储更大的值。

序列化数据可以写入数据库或文件，并在之后检索出来。

```php
<?php

// 4_30.php

$a = array(

'apple',

'orange',

'banana',

'animal' => 'horse',

'vechicle' => 'truck'

);

file_put_contents('serialize.dat', serialize($a));
```

在这个例子中，我们将序列化数组写入一个名为`serialize.dat`的文件。在接下来的例子中，读取该文件，进行反序列化，并使用`print_r()`函数打印结果。

```php
<?php

// 4_31.php

$a = unserialize(file_get_contents('serialize.dat'));

print_r($a);
```

输出如下所示：

```
Array
(
    [0] => apple
    [1] => orange
    [2] => banana
    [animal] => horse
    [vechicle] => truck
)
```

## 第五章 日期和时间

PHP 没有专门用于存储日期或时间值的变量类型。相反，使用整数来表示自 1970 年 1 月 1 日（也称为纪元）之前或之后的秒数。当日期和时间被视为单个值时，它们被称为时间戳。使用整数可以轻松计算两个时间戳之间的差异，因为可以应用普通数学运算。

## 配方 5-1：处理时区

### 问题

PHP 中所有的日期和时间函数都假设时间戳是协调世界时（UTC）。

有必要配置 PHP 使用特定的时区，以转换所有日期值的输入和输出，使它们按正确顺序显示。

### 解决方案

PHP 可以通过`php.ini`文件中的设置进行配置。该设置名为`date.timezone`，允许的值包括诸如`CET`（中欧时间）和`PST`（太平洋标准时间）等名称。此外，您还可以按区域和城市进行配置，例如`America/Los_Angeles`或`Europe/Amsterdam`。有关支持时区的完整列表，请参阅 [`php.net/manual/en/timezones.php`](http://php.net/manual/en/timezones.php)。

当网站用户都是本地用户且共享同一时区时，为 Web 服务器配置单个时区效果很好。如果用户来自不同的时区，则有必要能够为每个用户定义时区。这可以通过定义一个偏好设置来实现，用户可以在其中选择时区，并在使用任何日期函数之前使用`date_default_timezone_set()`函数设置当前时区。匿名用户没有定义的偏好，他们将看到的所有日期/时间值都假定为服务器定义的时区。

如果在调用日期/时间函数时未定义时区，系统将打印警告并回退到 UTC 时区。

```
Warning: date(): It is not safe to rely on the system's time zone settings. You are
*required* to use the date.timezone setting or the date_default_timezone_set() function.
```



# 如果您使用了上述方法但仍收到此警告，很可能是您输错了时区标识符。我们暂时选择了 `'UTC'` 时区，但请设置 `date.timezone` 以选择您的时区。

© Frank M. Kromann 2016

F.M. Kromann, *PHP 与 MySQL 实战秘籍*, DOI 10.1007/978-1-4842-0605-8_5

# 第 5 章 ■ 日期和时间

在较旧版本的 PHP 中，警告信息可能有所不同，或者根本不会给出任何警告。强烈建议任何使用 PHP 5.3 之前版本的用户进行升级，这不仅出于安全和性能的考虑，还能享受所有新功能带来的好处。

### 工作原理

确保时区已被配置的简单方法是在 `php.ini` 中设置 `date.timezone` 值。

请记住，在修改 `php.ini` 后需要重启 Web 服务器。通过程序化方式设置时区可以在需要时覆盖 `php.ini` 中的值。

```php
<?php

// 5_1.php

$ts = time();

echo date("Y m d H:i:s", $ts) . "\n";

date_default_timezone_set("UTC");

echo date("Y m d H:i:s", $ts) . "\n";
```

当 `date.timezone` 被设置为 `America/Los_Angeles` 时，这段代码的输出结果如下所示：`2015 03 25 11:36:12`

`2015 03 25 18:36:12`

如果输出显示在浏览器中，由 `"\n"` 产生的空白将不会显示为换行。请使用 HTML（`<br/>`）或添加一个标头将内容类型设置为 `text/plain`。

这里的时间差显示为 7 小时，尽管 UTC 时间和洛杉矶时间的正常差异是 8 小时。这是因为洛杉矶实行了夏令时。夏令时从不适用于 UTC 时间。

在上面的例子中，`date` 函数接收了两个变量。第一个是格式字符串，第二个是时间戳。第二个参数是可选的，如果不传入，`date` 函数在生成字符串时会使用当前时间。

## 方法 5-2. 创建时间戳

### 问题

时间戳在对数据进行排序时非常有用。它们没有日期时间字符串那种排序问题——后者的排序取决于字符串的格式。在欧洲，日期通常格式化为日、月、年；而在美国，通常使用月、日、年。一种简单的方法是使用年、月、日的格式。这样可以提供一种按日期正确排序的方法，但其展示效果并不理想。

### 解决方案

在 PHP 中有多种创建时间戳的方法。最简单的方法是使用 `time()` 函数来获取当前日期和时间的 Unix 时间戳。如果您需要获取特定日期和时间的时间戳，可以使用 `mktime()` 函数。这个函数接受多个参数。所有参数都是可选的，并且可以从右侧开始省略。任何省略的参数都将根据系统时间和选定的时区设置为当前值。这些参数依次是：时、分、秒、月、日、年，以及 Is DST（指示是否应考虑夏令时）。

### 工作原理

将日期和时间拆分成独立的部分并不总是很方便，因此针对这些情况，有一些函数可以帮上忙。`strtotime()` 函数通过解析字符串来创建时间戳。该函数不需要特定的格式。这使得解释 `03 01 2015` 变得困难，因为它既可以表示 3 月 1 日，也可以表示 1 月 3 日，这取决于您在世界上的哪个地方。`strtotime()` 函数还允许传入更具创意的字符串，例如 `"+1 month"`，它会将当前日期和时间加 1 个月；或者 `"January 5th 2015 + 15 weeks"`；又或者 `"first Monday of April 2016"`。

在接下来的例子中，我们将探讨创建时间戳的各种方法。

```php
<?php

// 5_2.PHP

// 获取当前日期和时间的时间戳
$ts = time();
echo $ts . "\n";
echo date("M j Y H:i:s", $ts) . "\n";

// 获取 2015 年 3 月 1 日 00:00:00 的时间戳
$ts = mktime(0, 0, 0, 3, 1, 2015);
echo $ts . "\n";
echo date("M j Y H:i:s", $ts) . "\n";

// 使用 strtotime() 获取特定日期时间的时间戳
$ts = strtotime("first Monday of April 2016");
echo $ts . "\n";
echo date("M j Y H:i:s", $ts) . "\n";
```

在这三个例子中，后两个日期的时间部分默认为 `00:00:00`。这是因为字符串中没有指定时间值。该示例首先打印每个时间戳的整数值，然后使用 `date()` 函数将其转换回字符串值。

`Mar 25 2015 14:06:40`

`Mar 1 2015 00:00:00`

`Apr 4 2016 00:00:00`

尽管 Unix 纪元被设定为从 1970 年 1 月 1 日开始，我们仍然可以使用表示此之前日期的时间戳。这是通过为时间戳使用负值来实现的。在旧版本的 PHP for Windows 中，这是不可能的。该问题已在 PHP 5.1 中修复。

在 32 位系统上，时间戳的范围从 `-2147483648` 到 `2147483647`，这些值分别对应日期 `1901/12/13 20:45:51` 和 `2038/01/19 03:14:08`。在 64 位系统上，对应的值分别是 `-292277022657/01/27 08:29:52` 和 `292277026596/12/04 03:30:07`。

## 方法 5-3. 处理日期

### 问题

当数据跨多个时区使用时，使用时间戳来存储日期值可能会导致一些不希望出现的结果。

这种情况会发生在生日和其他类似日期上。如果您创建了一个网站，来自世界各地的用户可以在上面输入他们的出生日期，而您将其存储为一个整数时间戳，然后使用一个不同用户的时区来调用 `date()` 函数，您可能会得到错误的日期。

### 解决方案

`mktime()` 和 `date()` 有两个特殊版本，它们忽略时区，始终基于 UTC 值进行运算。这两个函数分别叫做 `gmmktime()` 和 `gmdate()`。当您想将日期值存储为时间戳时，它们非常方便。

### 工作原理

在这个例子中，有一个用户以 `Europe/Copenhagen`（欧洲/哥本哈根）时区输入了他的生日：1980 年 10 月 10 日。第二个用户从 `America/Los_Angeles`（美洲/洛杉矶）时区查看这个生日。

```php
<?php

// 5_3.php

// 用户 1 出生于 1980 年 10 月 10 日，地点：丹麦哥本哈根
date_default_timezone_set("Europe/Copenhagen");
$dob = mktime(0, 0, 0, 10, 10, 1980);

// 用户 2 从洛杉矶查看生日
date_default_timezone_set("America/Los_Angeles");
echo date("M j Y", $dob). "\n";
```

现在生日看起来像是 10 月 9 日。如果我们用 `gmdate()` 和 `gmmktime()` 函数替换 `date()` 和 `mktime()`，这个问题就会消失。

```php
<?php

// 5_4.php

// 用户 1 出生于 1980 年 10 月 10 日，地点：丹麦哥本哈根
date_default_timezone_set("Europe/Copenhagen");
$dob = gmmktime(0, 0, 0, 10, 10, 1980);

// 用户 2 从洛杉矶查看生日
date_default_timezone_set("America/Los_Angeles");
echo gmdate("M j Y", $dob). "\n";
```

并非所有月份的天数都相同，PHP 提供了一种快速方法来验证日、月、年的组合是否对应一个有效的日期。这个函数叫做 `checkdate()`，它接受三个参数：月、日、年。如果参数对应一个有效的日期，函数返回 `true`，否则返回 `false`。

## 方法 5-4. 显示时间戳

### 问题

日期和时间在世界各地有许多不同的使用方式。使用整数来表示任何日期、时间或日期/时间值的 Unix 时间戳，可以方便地进行排序和计算日期时间差，但它们对于显示实际日期来说并不实用。

### 解决方案

`date()` 函数用于根据一个包含大量选项的格式字符串，将 Unix 时间戳转换为可读的字符串格式。除了使用英文月份和星期几名称的 `date()` 函数外，`strftime()` 函数提供了相同的功能，但它使用 `setlocale()` 的设置来提供本地化名称，并且其格式字符串与 `date()` 函数使用的值略有不同。

### 工作原理



`date()`函数的格式化字符串由大小写字母组合而成。任何未被识别为格式化字符的字符将保持原样返回，这对添加句点、斜杠、破折号或冒号来分隔值非常有用。

在下一个示例中，我们获取当前时间的时间戳，并将其格式化为欧洲和美国的日期时间字符串。该时间戳将代表服务器配置时区中的日期和时间。

```php
// 5_5.php

$ts = time();

// European
echo date("d/m/Y H:i:s", $ts) . "\n";

// US
echo date("m/d/Y h:i:s a", $ts) ."\n";
```

该程序的输出类似如下：

```
27/03/2015 16:46:28
03/27/2015 04:46:28 pm
```

两个主要区别在于日和月的互换，以及 24 小时制与 12 小时制时间表示的使用。如你所见，空格、斜杠和冒号不会被转换，但所有其他字符都会被转换为返回的日期时间字符串的一部分。支持的完整字符列表可在[`php.net/date`](http://php.net/date)找到。

由于`strftime()`函数依赖于底层操作系统函数来格式化字符串并翻译月份和星期几的名称，系统期望格式化占位符以`%`开头，后跟单个大写或小写字母。

让我们看同样的例子，但这次包含了星期几的名称和月份的完整名称。

```php
// 5_6.php

$ts = time();

echo date("l F d Y h:i:s a", $ts) ."\n";
```

这将输出如下所示的日期时间字符串：

```
Friday March 27 2015 04:54:36 pm
```

为了使用`strftime()`函数，我们需要像下一个示例这样更改格式化字符串。

```php
// 5_7.php

$ts = time();

echo strftime("%A %B %e %Y %I:%M:%S %P ", $ts) ."\n";
```

此示例将生成如下输出：

```
Friday March 27 2015 05:03:40 pm
```

为了以不同语言获取月份和星期几的名称，我们可以使用`setlocale()`函数来改变语言行为。

```php
// 5_8.php

$ts = time();

setlocale(LC_TIME, 'en_US');
echo "English: ";
echo strftime("%A %B %e %Y %I:%M:%S %P", $ts) ."\n";

setlocale(LC_TIME, 'de_DE');
echo "German: ";
echo strftime("%A %B %e %Y %H:%M:%S", $ts) ."\n";

setlocale(LC_TIME, 'fr_FR');
echo "French: ";
echo strftime("%A %B %e %Y %H:%M:%S", $ts) ."\n";
```

现在我们使用操作系统提供的字符串，以`setlocale()`选择的语言显示工作日和月份名称，输出如下：

```
English: Friday March 27 2015 05:16:32 pm
German: Freitag März 27 2015 17:16:32
French: Vendredi mars 27 2015 17:16:32
```

> **注意：** 系统必须安装所需语言的语言包，`setlocale()`函数才能正常使用。

如果你不想依赖`setlocale()`来翻译星期几和月份的名称，简单的解决方案是使用这些名称的数值，但也可以编写自己的格式化函数。

```php
// 5_9.php

$weekdays = array(
    'en' => array(
        'Sunday', 'Monday', 'Tuesday', 'Wednesday',
        'Thursday', 'Friday', 'Saturday', 'Sunday'
    ),
    'da' => array(
        'søndag', 'mandag', 'tirsdag', 'onsdag',
        'torsdag', 'fredag', 'lørdag', 'søndag'
    ),
);

$months = array(
    'en' => array(
        1 => 'January', 'February', 'March', 'April', 'May', 'June',
        'July', 'August', 'September', 'October', 'November', 'December'
    ),
    'da' => array(
        1 => 'januar', 'februar', 'marts', 'april', 'maj', 'juni',
        'juli', 'august', 'september', 'oktober', 'november', 'december'
    ),
);

function FormatDate($ts, $lang = 'en') {
    global $weekdays, $months;
    $d = date('j', $ts);
    $w = date('w', $ts);
    $m = date('n', $ts);
    $y = date('Y', $ts);
    switch($lang) {
        case 'en' :
            return $weekdays[$lang][$w] . ", " . $months[$lang][$m] .
                   " {$m} {$y}";
            break;
        case 'da' :
            return $weekdays[$lang][$w] . ", {$m} " . $months[$lang][$m] .
                   " {$y}";
            break;
    }
}

$ts = time();
echo FormatDate($ts) . "\n";
```



`echo FormatDate($ts, 'da') . "\n";`

此示例将产生类似下方的两行输出：

Sunday, March 3 2015

søndag, 3 marts 2015

这种方法的缺点在于需要维护翻译表，但你可以在应用程序内部完成所有操作，无需依赖宿主环境。

## 配方 5-5：使用 ISO 格式

### 问题

在许多情况下，我们需要一种非常具体的字符串格式化方式。在创建电子邮件时，使用格式化字符串可以方便地生成符合标准的日期值用于邮件头，而无需记忆确切的格式细节。

### 解决方案

`date()` 函数提供了一些特定的格式化字符，它们会根据特定的 ISO 格式将时间戳转换为字符串表示。使用 `"c"` 会生成符合 ISO 8601 标准的格式化字符串，而 `"r"` 则会生成符合 RFC 2822 标准的字符串。

ISO 8601 标准旨在使世界上不同地区（可能使用不同格式）之间能够交换日期和时间值。RFC 2822 标准则涵盖了电子邮件的交换。有一种常用的字符串看起来与此类似，它被用于在 HTTP 响应中设置 `Last-Modified` 和 `Expire` 头部信息。主要区别在于，电子邮件数据包含时区偏移量，而响应头部信息始终以 GMT 值给出。

### 工作原理

如果你创建自己的电子邮件客户端，你将需要在每封电子邮件中设置收件人、发件人、主题和其他参数的头部信息。此外，你还需要为电子邮件的创建日期和时间创建一条头部信息。它可能看起来像这样：

```
<?php

// 5_10.php

$msg = "From: mail@example.com\r\n" .
       "To: mail@somedomain.com\r\n" .
       "Subject: ISO date format\r\n";
$msg .= date("c") . "\r\n\r\n";
$msg .= "This is the body of the email message\r\n";
echo $msg;
```

这只是一个非常简单的电子邮件示例，建议使用众多电子邮件客户端类之一来处理电子邮件的发送和接收。此示例将输出 `$msg` 变量的内容。

From: mail@example.com

To: mail@somedomain.com

Subject: ISO date format

2015-04-01T12:25:38-07:00

This is the body of the email message

为 HTTP 响应设置 `Expires` 和 `Last-Modified` 头部信息可能看起来像这样：

```
<?php

// 5_11.php

$gmt_mtime = gmdate('D, d M Y H:i:s', $last_modified) . ' GMT';
$gmt_etime = gmdate('D, d M Y H:i:s', time() + $ttl) . ' GMT';
header("Pragma: Cache");
header("Cache-Control: max-age=0, must-revalidate");
header("Last-Modified: " . $gmt_mtime);
header("Expires: " . $gmt_etime);
if (!empty($_SERVER['HTTP_IF_MODIFIED_SINCE']) &&
     $_SERVER['HTTP_IF_MODIFIED_SINCE'] == $gmt_mtime) {
    http_response_code(304);
    exit();
}

// Output the rest of the response
```

这将允许浏览器仅在重新验证后返回 HTTP 状态 304 时使用响应的缓存版本。

## 配方 5-6：处理周数

### 问题

在世界上的某些行业，使用周数相当普遍。不幸的是，关于如何计算周数，存在多种不同的标准。例如，包含一年中第一个星期四的那一周，包含 1 月 4 日的那一周，包含四天或更多天的第一周，或者从 12 月 29 日到 1 月 4 日之间的星期一开始的那一周。

### 解决方案

PHP 中的日期函数支持 ISO-8601 周数标准，该标准将包含一年中第一个星期四的那一周定为第一周。另外值得一提的是，周数的切换发生在星期日和星期一之间，而不是像世界某些地区定义的那样发生在星期六和星期日之间。

### 工作原理

为了获取给定时间戳的周数，我们使用 `'W'` 格式化字符来获取符合 ISO-8601 标准的周数。周数通常与年份一起给出，因此格式化字符串将是 `'Y-W'` 或 `'W-Y'`。

```
<?php

// 5_12.php

$ts = time();
```



`echo date("Y-W", $ts) . "\n";`

# 配方 5-7. `DateTime` 类

### 问题

当使用整数值来表示自参考时间点以来的秒数作为时间戳时，可处理的日期范围由整型变量允许的值决定。在整数为 4 字节值的 32 位系统上，时间戳的范围是从 UTC 时间的 1901-12-13 20:45:51 到 2038-01-19 03:14:08（大约正负 68 年）。那么在 PHP 中，如何表示超出此范围的日期和时间值呢？

### 解决方案

在整数为 8 字节值的 64 位系统上，时间戳范围介于 UTC 时间的 -292277022657-01-28 08:29:52 和 292277026596-12-04 15:30:07 之间（大约正负 290 亿年）。

将操作系统从 32 位更改为 64 位可能并非总是可行，因此，PHP 提供了一个名为 `DateTime()` 的内置类，它可以支持与 64 位系统相同的日期范围。`DateTime()` 类在 PHP 5.1 版本中作为实验性功能引入，并从 5.2 版本开始被包含在内。

### 工作原理

实例化 `DateTime()` 类的对象可以不带参数。这将创建一个包含当前时间和当前时区的对象。可以使用与 `date()` 函数相同的格式化字符串来打印日期和时间。

```php
// 5_13.php
$d = new DateTime();
echo $d->format("M d Y h:i:s a") . "\n";
```

这将生成类似如下的输出：

`Mar 29 2015 11:18:36 am`

如果你希望日期/时间对象包含特定的日期和时间值，而非当前时间，可以在实例化对象时将该值作为第一个参数传入。日期/时间值的格式必须遵循特定格式。支持的格式完整列表可在此处找到：[`php.net/manual/en/datetime.formats.php`](http://php.net/manual/en/datetime.formats.php)。

使用特定日期创建日期/时间对象的代码如下所示：

```php
// 5_14.php
$d = new DateTime("2010-10-31 13:15:00");
echo $d->format("M d Y h:i:s a P") . "\n";
```

第一个参数同时包含日期部分和时间部分，输出将根据提供的格式字符串显示日期和时间：

`Oct 31 2010 01:15:00 pm -07:00`

如果你传入的日期字符串来自不同的时区，可以使用第二个可选参数在不同时区创建日期/时间对象。当使用时区参数时，日期/时间对象会记住该值，并将其用于所有计算，包括显示。如果我们修改代码以包含一个时区，输出将保持不变，唯一的区别是 GMT 偏移量。

```php
// 5_15.php
$tz = new DateTimeZone("Europe/Copenhagen");
$d = new DateTime("2010-10-31 13:15:00", $tz);
echo $d->format("M d Y h:i:s a P") . "\n";
```

请注意，时区是以从 `DateTimeZone()` 类实例化的对象形式传入的。输出将如下所示：

`Oct 31 2010 01:15:00 pm +01:00`

作为第一个参数传入的日期/时间字符串的解析方式与 `strtotime()` 函数相同。这意味着你可以在实例化对象时解析相对日期，如下一个示例所示，其中创建了两个对象。第一个使用当前日期和时间，第二个计算未来两个月后的日期/时间，同时将确切时间指定为 12:00。

```php
// 5_16.php
$d = new DateTime();
echo $d->format("M d Y h:i:s a P") . "\n";

// 2 个月后的中午
$d = new DateTime("+2 months 12:00");
echo $d->format("M d Y h:i:s a P") . "\n";
```

脚本执行时，当地时间刚过中午，因此这两个时间很接近，但日期相隔两个月。

`Mar 29 2015 11:39:46 am -07:00`

`May 29 2015 12:00:00 pm -07:00`

在创建对象之后，可以使用 `DateTime()` 对象上的多个方法来更改日期、时间或时区。

设置特定日期或时间可以通过 `DateTime::modify()` 完成。此方法接受一个字符串参数，其工作方式与构造函数中使用的参数相同。你可以将日期/时间更改为新的显式值，也可以相对地修改它。

在下一个示例中，日期/时间值被初始化为当前时间，然后使用 `modify()` 方法更改为特定日期，接着相对更改为两个月后的日期，最后更改为 2000 年前的日期。

```php
// 5_17.php
date_default_timezone_set('America/Los_Angeles');
$d = new DateTime();
echo $d->format("M d Y h:i:s a P") . "\n";

$d->modify("2010-10-31");
echo $d->format("M d Y h:i:s a P") . "\n";

// 2 个月后的中午
$d->modify("+2 months 12:00");
echo $d->format("M d Y h:i:s a P") . "\n";

$d->modify("-2000 years");
echo $d->format("M d Y h:i:s a P") . "\n";
```

最终输出显示了对应于不存在夏令时的日期的 GMT 偏移量变化。第一个日期是因为它落在冬季，而第二个是因为 2000 年前不存在夏令时。

`Mar 29 2015 11:56:05 am -07:00`

`Oct 31 2010 11:56:05 am -07:00`

`Dec 31 2010 12:00:00 pm -08:00`

`Dec 31 0010 12:00:00 pm -08:00`

尽管此代码是在 64 位系统上执行的，但它在 32 位平台上也能产生相同的结果。

另一种处理相对时间的方法是使用 `DateTime()` 对象的 `add()` 或 `sub()` 方法。这些方法接受一个参数，该参数是 `DateInterval()` 类的对象。`DateInterval()` 构造函数使用一个字符串来定义时间段长度，通过指定年数（`Y`）、月数（`M`）、周数（`W`）、天数（`D`）、小时数（`H`）、分钟数（`M`）和秒数（`S`）。请注意，`M` 同时用于月和分。这是通过两个额外的字母 `P` 和 `T` 实现的，它们分别表示时间段（`P`）和时间（`T`）部分。要创建一个 2 年 3 个月 5 天 10 小时的时间间隔，你可以使用字符串 `'P2Y3M5DT10H'`。

```php
// 5_18.php
date_default_timezone_set('America/Los_Angeles');
$d = new DateTime("Oct 10 1980");
echo $d->format("M d Y h:i:s a P") . "\n";

$i = new DateInterval('P2Y3M5DT10H');
$d->add($i);
echo $d->format("M d Y h:i:s a P") . "\n";
```

输出将如下所示，其中 GMT 偏移量发生变化，因为日期从夏令时期间移动到了非夏令时期间：

`Oct 10 1980 12:00:00 am -07:00`

`Jan 15 1983 10:00:00 am -08:00`

在比较日期之间的差异时，也会用到 `DateInterval()` 类。

```php
// 5_19.php
// 用户 1 创建了一个日期
$d1 = new DateTime("Oct 10 1980");
$d2 = new DateTime("March 15 1990");
$i = $d1->diff($d2);
echo $i->format("%Y %M %D %H:%I:%S") . "\n";
```

在这个例子中，我们有两个日期和时间对象，它们大约相隔 10 年。脚本的输出显示 `09 05 05 00:00:00`，根据格式，这表示 9 年 5 个月 5 天。没有时间差异，因此两个对象的时间相同。

在前面那个例子中，一个用户根据本地时区输入日期，另一个用户根据其自己的时区查看日期，这可以通过更改日期/时间对象上的时区来处理。

```php
// 5_20.php
// 用户 1 创建了一个日期
$tz = new DateTimeZone("Europe/Copenhagen");
$d = new DateTime("Oct 10 1980", $tz);
echo $d->format("M d Y h:i:s a P") . "\n";

// 洛杉矶的用户 2
$tz = new DateTimeZone("America/Los_Angeles");
$d->setTimezone($tz);
echo $d->format("M d Y h:i:s a P") . "\n";
```

代码显示了与之前相同的缺陷：由于时区偏移量的差异，存储为时间戳的日期看起来会像是两个不同的日期。

`Oct 10 1980 12:00:00 am +01:00`

`Oct 09 1980 04:00:00 pm -07:00`



# 日期与时间

同样，解决方法是像对待生日这类不应包含时间部分的日期一样，无论用户从何处输入或显示这些日期，都将其视为同一时区的日期。为此使用 UTC 时区虽然简单，但实际上只要保证输入和输出时该时区不发生变化，任何时区都可以。

## 第 5 章 ■ 日期与时间

`DateTime` 类也可以通过使用别名函数以更过程化的风格来操作：`date_create_immutable()` 是 `DateTimeImmutable::__construct()` 的别名。我更喜欢面向对象的方式，并且觉得这些命名更清晰。要查看 `DateTime` 函数的完整列表，请访问 [`php.net/manual/en/ref.datetime.php`](http://php.net/manual/en/ref.datetime.php)。

## 技巧 5-8：存储日期与时间值

### 问题
使用整数或 UNIX 时间戳来表示时间和日期值，便于操作和存储。如果我创建数据库模式，我倾向于直接使用整数或长整型列来存储我在 PHP 脚本中使用的确切值。这样，我只需要在向用户显示日期和时间时将其转换为字符串，并在接收用户输入时从字符串进行转换。

如果你使用的数据库模式包含原生的 datetime 或 timestamp 列，你可能能够将值作为整数选取并用 PHP 进行格式化，也可能需要检索该列的文本值并在 PHP 中执行转换。

### 解决方案
MySQL 的 datetime 列期望字符串格式为：4 位年份、2 位月份和日期，以及 2 位小时、分钟和秒。时间格式应为 24 小时制。对应的格式化字符串如下：`date("Y-m-d H:i:s")`。

### 工作原理
如果我们创建一个包含单个 datetime 列的简单表：

```sql
create table date_time (
    ts datetime
);
```

然后我们可以创建一个简单的 PHP 脚本，用 1970 年至今之间的随机日期填充数据库。

```php
<?php
// 5_21.php

$now = time();
$dates = [];

for ($i = 0; $i < 100; $i++) {
    $dates[] = mt_rand(0, $now);
}

$con = mysqli_connect("127.0.0.1", "root", "seecret", "book");
foreach ($dates as $d) {
    $date = date("Y-m-d H:i:s", $d);
    mysqli_query($con, "insert into date_time (ts) values ('{$date}');");
}

mysqli_close($con);
```

### 第 5 章 ■ 日期与时间

如果我们想从数据库中读取相同的值并使用 PHP 将其转换为 UNIX 时间戳，代码会类似这样。

```php
<?php
// 5_22.php

$now = time();
$dates = [];

for ($i = 0; $i < 100; $i++) {
    $dates[] = mt_rand(0, $now);
}

$con = mysqli_connect("127.0.0.1", "root", "secret", "book");
$rs = mysqli_query($con, "select * from date_time");

while ($row = mysqli_fetch_row($rs)) {
    $dates[] = strtotime($row[0]);
}

mysqli_free_result($rs);
mysqli_close($con);

print_r($dates);
```

## 技巧 5-9：计算流逝时间

### 问题
虽然 PHP 有很多将时间戳转换为字符串表示的方法，但它没有直接显示流逝时间的功能。

### 解决方案
要获取两个时间戳之间的流逝时间，只需将它们相减即可。这将得到两个时间戳之间的秒数。使用 `date` 函数，可以用小时、分钟和秒的形式很好地表示两个时间戳之间的间隔。如果流逝时间超过 24 小时，我们还需要考虑天、周、月和年。通过简单的数学计算可以得出这些值。一天有 86400 秒，一周是它的 7 倍，但月份长度并不相同，年份长度也会稍有变化。因此，在计算时，我们只能取平均值，即每月 30 天或每年 365 天。

与其尝试将秒数转换为等效的月、天、小时等，不如使用 `DateTime()` 类来创建两个时间戳对象，然后计算差值，并使用 `DateInterval::format()` 方法来表示流逝时间。

### 工作原理



在本示例中，我们使用`strtotime()`函数生成两个事件的时间戳。

```php
<?php

// 5_23.php

$ts1 = strtotime("2000-06-15 20:30:00");

$ts2 = strtotime("2005-03-18 14:15:30");

```

第 5 章 ■ 日期和时间

```php
$d1 = new DateTime();

$d1->setTimestamp($ts1);

echo $d1->format("Y-m-d H:i:s") . "\n";

$d2 = new DateTime();

$d2->setTimestamp($ts2);

echo $d2->format("Y-m-d H:i:s") . "\n";

$diff = $d1->diff($d2);

echo $diff->format("Years: %y, Months: %m, days: %d %H:%I:%S") . "\n";
```

脚本输出是一个字符串，其格式符合格式字符串的规则，其中所有以`%`开头的格式字符都会被替换为时间差中的对应值。

```
Years: 4, Months: 9, days: 2 17:45:30
```

`strtotime()`函数也可用于计算相对日期/时间偏移。该函数支持 GNU 日期语法（[`php.net/manual/en/datetime.formats.php`](http://php.net/manual/en/datetime.formats.php)）。这是一个非常复杂的系统，允许我们从当前时间或固定时间戳向前或向后移动若干天、周、月。

以下示例展示了它的几种用法：

```php
<?php

// 5_24.php

date_default_timezone_set("America/Los_Angeles");

// 计算当前时间起 3 周后的时间戳

$ts = strtotime("+3 weeks");

echo date("Y/m/d h:i:s a", $ts) . "\n";

// 计算当前时间起 5 天前的时间戳

$ts = strtotime("-5 days");

echo date("Y/m/d h:i:s a", $ts) . "\n";

// 计算给定日期后 3 个月零 11 点 30 分的时间戳

$ref = mktime(0, 0, 0, 12, 15, 2012);

$ts = strtotime("11:30 + 3 month", $ref);

echo date("Y/m/d h:i:s a", $ts) . "\n";
```

此示例使用了`America/Los_Angeles`时区，输出结果如下：

```
2016/04/27 12:25:41 pm

2016/04/01 12:25:41 pm

2013/03/15 11:30:00 am
```

## 第 6 章 字符串

字符串是 PHP 中基本的变量类型之一。它们通常用于构建 PHP 脚本的输出，作为对 HTTP 请求的响应。

字符串是一系列字符，每个字符被定义为单个字节，序数值范围为 0 到 255。单字节的限制使得字符串无法原生支持 Unicode。

`mb_string`扩展使得在 PHP 中处理 Unicode（多字节字符串）成为可能，但这些字符串不能使用原生变量类型和原生字符串操作函数来处理。

在 32 位系统上，PHP 字符串最长可达 2GB。这要求 PHP 配置足够的内存来处理这种大小的变量。从 PHP 7 开始，所有整数都提供真正的 64 位支持，使得字符串（以及文件）的大小可以远超过之前的 2GB 限制。

字符串以内存块和长度形式存储，这保证了二进制安全。二进制安全的字符串可以包含一个或多个空字节，而不会导致字符串部分内容丢失。

### 6-1. 创建字符串

### 问题

字符串可以通过多种方式创建：使用单引号或双引号定义的值进行赋值，或者使用众多返回字符串的函数的返回值。

### 解决方案

`str_repeat()`和`str_pad()`是两种可用于生成字符串的函数。前者通过重复一个或多个字符的模式多次生成字符串。后者则将一个特定字符串作为填充添加到另一个字符串的前面或后面，以创建特定长度的字符串。

对于给定的字符串，可以改变字符顺序，使用相同的字符生成一个随机顺序的新字符串。`str_shuffle()`函数的作用类似于`array_shuffle()`；它会获取所有元素并将其随机排列。

#### 实现原理

在第一个示例中，我们包含时间的字符串。由于两个时间的位数不同，两个字符串的总长度也不同。为了使两个字符串打印时数字对齐，我们需要用空格或零填充这两个字符串。

---

© Frank M. Kromann 2016



# F.M. Kromann，《PHP 与 MySQL 实战》，DOI 10.1007/978-1-4842-0605-8_6

## 第 6 章 ■ 字符串

```php
<?php
// 6_1.php

$t1 = "7:00 am";
$t2 = "11:00 am";

echo str_pad($t1, 8, '0', STR_PAD_LEFT) . "\n";
echo str_pad($t2, 8, '0', STR_PAD_LEFT) . "\n";
```

这两个时间值将具有相同的长度：

```
07:00 am
11:00 am
```

如果将这些字符串用作 Web 请求的响应，可以简单地将 CSS 属性的 `text-align` 设置为 `right`，以将字符串右对齐。

在下一个示例中，我们基于模式 `'abc'` 重复 15 次来创建一个字符串。

```php
<?php
// 6_2.php

$s = str_repeat('abc', 15);
echo $s . "\n";
```

此示例生成以下输出：

```
abcabcabcabcabcabcabcabcabcabcabcabcabcabcabc
```

一个更实用的示例是生成指定长度的、由单个字符组成的字符串，如配方 6-2 所示。

## 配方 6-2：处理字符串中的字符

### 问题

字符串本质上是一个由单个字符组成的长列表。它在许多方面看起来像一个字符数组，那么是否有可能访问或更改字符串中给定位置的字符？

### 解决方案

使用变量名后的方括号，可以像访问数组中的元素一样访问每个字符。字符从索引 0 开始，到索引 length-1 结束。要从字符串 `$a="abcdef"` 中获取第三个字符，可以通过引用 `$a[2]` 来实现。

### 工作原理

在下一个示例中，声明了一个静态值的字符串 `$s`，我们利用数组语法，根据字符 7 在字符串中的位置来替换单个字符。

```php
<?php
// 6_3.php

$s = "The sun is up @ 7am";
$s[16] = '6';
echo $s . "\n";
```

此示例展示了如何替换现有字符串中的单个字符。如果你想添加字符或字符串，可以使用连接运算符（`.` 句点）来构建更长的字符串。

```php
<?php
// 6_4.php

$s = "The sun is up @ 7am";
$s .= " and it sets again at 7pm";
echo $s . "\n";
```

连接可以通过 `$a = $b . $c;` 实现（创建一个新变量来存储结果），也可以使用 `$a .= $b;`，这等价于 `$a = $a . $b;`，但写法更简洁。

## 配方 6-3：替换字符

### 问题

并非总能知道某个字符的确切位置，而且仅将一个字符替换为另一个字符也有限制。能够将多个字符替换为零个或多个字符，可以提供更大的灵活性。

### 解决方案

`str_replace()` 函数是将字符串的所有实例替换为另一个字符串的简单方法。该函数接受三个参数：第一个是要替换的字符串，第二个是用于替换的字符串，第三个是执行替换操作的字符串。

前两个参数可以是字符串数组。在这种情况下，替换函数将对数组中的每个字符串执行操作。如果第一个参数是数组而第二个参数是字符串，则该函数会将第一个数组中定义的所有元素替换为作为第二个参数传递的字符串。

如果第一个数组包含的值多于第二个数组，则第一个数组中任何在第二个数组中没有匹配值的元素都将被替换为空字符串。

如果第一个值是字符串而第二个值是数组，则该字符串将被替换为单词 `'Array'`。

`str_replace()` 有一个变体函数 `str_ireplace()`。该函数在搜索部分不区分大小写。

### 工作原理

考虑存储在字符串中的 HTML 片段 `'<h1>Document Title</h1>'`。如果我们决定将标签从 `<h1>` 改为 `<h2>`，可以使用 `str_replace()` 函数。

```php
<?php
// 6_5.php

$s = "'<h1>Document Title</h1>";
echo str_replace("h1", "h2", $s) . "\n";
```

这仅在确定字符串 `h1` 仅作为标签存在时才有效。如果它作为字符串的一部分被使用，也会在那里被替换，但仅当大小写完全匹配时。为了稍加改进，我们可以包含结尾的 `>`，并使用该函数的不区分大小写版本。

```php
<?php
// 6_6.php
```



# PHP 字符串处理相关食谱

## 食谱 6-4. 使用 Heredoc 和 Newdoc 创建长字符串

### 问题描述

通过拼接字符串元素来构建字符串会需要大量额外的字符来处理换行符、转义字符和引号。

### 解决方案

使用 Heredoc 和 Newdoc 可以在不使用引号的情况下创建长字符串，同时保留换行符。Heredoc 的工作方式类似于双引号字符串，其中包含的任何 PHP 变量都会被解析为其值；而 Newdoc 则类似于单引号字符串，不会进行变量替换。

Heredoc 和 Newdoc 字符串都以操作符`<<<`开头，后跟一个名称。对于 Heredoc，名称只是一组字符序列；对于 Newdoc，则使用单引号括起来的名称。两种类型都会在名称作为行中唯一内容（名称前无空格，名称后有分号终止命令）的位置结束字符串声明。

### 工作原理

不一定要使用`HEREDOC`或`NEWDOC`作为标识符，只要块的开头和结尾使用相同的标识符即可。

```php
<?php
// 6_7.php
// 定义一个 Heredoc 块
$h = <<<HEREDOC
Some string
value
HEREDOC;
```

```php
<?php
// 6_8.php
// 定义一个 Newdoc 块
$n = <<<'NEWDOC'
A different
string value
NEWDOC;
```

这些块既可以作为变量赋值使用，也可以作为数组或函数调用中的内联结构使用。在下面的示例中，我们生成了一个包含两个字符串元素的数组：

```php
<?php
// 6_9.php
$a = [
    <<<HEREDOC
Some string
value
HEREDOC
    ,
    <<<HEREDOC
Second string
HEREDOC
    ,
];
```

这将产生以下输出：

```
Array
(
    [0] => Some string
    value
    [1] => Second string
)
```

注意字符串中换行符是如何被保留的。

```php
<?php
// 6_10.php
printf("String = %s\n", <<<HEREDOC
This is a very long string
HEREDOC
);
```

注意，字符串的结束部分必须是行中唯一的内容，除了用于终止语句的分号外。如果 heredoc 用作数组声明的一部分，则结束名称必须是行中唯一的内容。用于分隔参数或数组元素的逗号必须放在单独的行上。即使是结束名称后面的空白也会导致解析错误。

使用 heredoc 定义的文本块对于构建 HTML 文档的模板非常实用。在接下来的示例中，我们使用数据库查询从数据库中选择两列，遍历它们，并基于两个模板创建 HTML 片段：

```php
<?php
// 6_11.php
$con = mysqli_connect("127.0.0.1", "root", "secret", "book"); 
$rs = mysqli_query($con, "select * from person");

if ($rs) {
    $persons = array();
    while($row = mysqli_fetch_assoc($rs)) {
        $persons[] = $row;
    }
    mysqli_free_result($rs);
}
mysqli_close($con);

$person_tpl = <<<HEREDOC
<div>
    <span>%first_name%</span>
    <span>%last_name%</span>
    <span>%date_of_birth%</span>
</div>
HEREDOC;

$section_tpl = <<<HEREDOC
<div>%persons%</div>
HEREDOC;

$html = "";
$p = array();
foreach ($persons as $person) {
    $r = str_replace('%first_name%', $person['first_name'], $person_tpl);
    $r = str_replace('%last_name%', $person['last_name'], $r);
    $r = str_replace('%date_of_birth%', gmdate("M j Y", $person['date_of_birth']), $r);
    $p[] = $r;
}
echo str_replace('%persons%', implode('', $p), $section_tpl);
```

内部模板（`$person_tpl`）定义了每个人的布局，外部模板定义了注入所有记录的 HTML 部分。

---

## 食谱 6-5. 字符串转义

### 问题描述

当字符串用作数据库查询的输入时，单引号用于开始和结束字符串值，因此需要指示字符串值中的单引号并非字符串的结束。如果没有正确处理，黑客很容易将恶意代码注入数据库。转义字符串是防止这种情况的最低限度措施。通常，程序员永远不应该信任用户的输入，尤其不应该将任何用户输入（查询字符串、表单提交等）直接注入数据库查询。

### 解决方案

大多数数据库使用反斜杠（`\`）形式的转义字符串，或者在某些情况下在单引号前添加一个额外的单引号，以指示该引号应该是字符串的一部分。使用`str_replace()`函数将单引号替换为两个单引号是准备用于更新和插入语句的字符串的一种方法。PHP 中的许多数据库扩展都支持转义字符串的函数。

使用数据库引擎提供的函数的好处是，它允许数据库根据选定的字符集来处理转义。

有两个函数用于添加和删除字符串中的反斜杠，分别是`addslashes()`和`stripslashes()`。这些函数不应用于转义插入数据库的字符串，而是用于向将要被求值的代码添加反斜杠，或者当输出嵌入在 JavaScript 代码中的内容时。`addslashes()`函数会转义 NULL 字符、反斜杠、单引号和双引号。

### 工作原理

在接下来的示例中，我们有一个包含带撇号名称的字符串变量。如果直接使用这个变量，SQL 服务器会报错；但如果在插入数据库之前正确转义了字符串变量，则会成功。

```php
<?php
// 6_12.php
$con = mysqli_connect("127.0.0.1", "root", "secret", "book");

// create table strings (
//   val varchar(250);
// );

$name = "Mark O'Brian";

// 这会失败
mysqli_query($con, "insert into strings (val) values ('{$name}');"); 
$name = mysqli_escape_string($con, $name);

// 这会成功
mysqli_query($con, "insert into strings (val) values ('{$name}');");
```

请记住，你永远不应该信任任何输入。始终验证和转义所有输入到你代码中的内容。

考虑以下代码，其中字符串`$name`可能是用户填写 HTML 表单中的名字值并提交到服务器的结果：

```php
<?php
// 6_13.php
$con = mysqli_connect("127.0.0.1", "root", "secret", "book");

// create table strings (
//   val varchar(250);
// );

$name = "My Name";

// 这会失败
mysqli_query($con, "insert into strings (val) values ('{$name}');");
```

这段代码中没有任何内容阻止用户向`$name`变量中输入 SQL 语句。用户不是输入`My Name`，而是可以输入`My Name'; delete from strings; --`。这将导致代码执行两条语句：第一条从`strings`表中获取所有匹配的行，第二条删除该表中的所有行。

---

## 食谱 6-6. 字符串格式化

### 问题描述

字符串有很多用途，通常需要处理字符串的部分内容。这可能涉及重新排序各部分、添加或删除字符串的部分内容，或者添加 HTML 标签来格式化字符串。

### 解决方案

可以使用`explode()`函数将字符串转换为数组，该函数接受两个参数。第一个参数是要查找并执行分割的模式，可以是空格字符、换行符或任何其他字符串模式。第二个参数是要转换为数组的字符串。返回值将是一个字符串数组。



# PHP 字符串处理函数

## 字符串拆分与合并

反向功能由`implode()`函数处理，该函数将数组转换为字符串，在每个数组元素之间插入分隔符字符串（第一个参数）。源数组不必是字符串数组，但每个元素在拼接前都会被转换为字符串。`join()`函数是`implode()`的别名。

这两个函数都不会修改参数中的字符串或数组，因此系统最终会拥有包含数据的两个变量。如果字符串或数组很大，可能没有足够的内存来同时保存两个值。在这种情况下，最好使用分词器函数`strtok()`，它可以一次拆分一个令牌，从而在任何时候只保留原始字符串和当前令牌在内存中。

### 工作原理

`explode()`和`implode()`有很多用途。假设你有一个文件想要逐行读取和处理，你可以读取文件内容并将其转换为可在循环中使用的数组。

```
<?php

// 6_14.php

$content = file_get_contents('file.txt');

$a = explode("\n", $content);

foreach ($a as $line) {

// Do something

}
```

如果你不需要将内容作为字符串使用，可以使用`file()`函数。该函数将读取文件并将各行作为数组返回。

同样，我们可以将一个整数数组转换为逗号分隔的整数字符串，用于 SQL 查询。

```
<?php

// 6_15.php

$a = [1, 5, 7, 9, 3, 4];

$s = implode(',', $a);

$sql = "select * form mytable where id in({$s})"
```

## 去除空白字符

### 问题

当用户在 HTML 表单中输入字符串值时，他们可能会不小心添加前导或尾随空格。如果这些值是名字和姓氏，并且你希望将它们拼接成全名，则可能会产生多余的空格。如果该字符串用作搜索查询的输入且包含前导或尾随空格，除非先移除这些字符，否则搜索可能不会产生任何结果。

### 解决方案

可以使用`ltrim()`、`rtrim()`或`trim()`函数之一移除开头、结尾或两端的空白字符。这些函数不会移除字符串中间的空白字符。空白字符包括空格字符、换行符（`\n`）或回车符（`\r`）。

### 工作原理

如果你使用 HTML 表单的输入作为数据库查询中的搜索参数，可以使用`trim()`函数去除字符串开头和结尾的空白字符。

```
<?php

// 6_16.php

$con = mysqli_connect('127.0.0.1', 'root', 'secret', 'book');

$q = mysqli_escape_string($con, trim($_POST['name']));

$sql = "select * form person where name like '%{$q}%'";
```

## 在字符串中查找字符串

### 问题

检查一个字符串是否包含在另一个字符串中、检查一个字符串在另一个字符串中出现的次数、或者查找特定字符串在另一个字符串中的位置，是处理字符串时常见的问题。

### 解决方案

在 PHP 中，有函数可以查找一个字符串（针）在另一个字符串（草堆）中的第一个或最后一个位置。这些函数称为`strpos()`或`strrpos()`，它们将返回一个表示位置的整数值，如果未找到字符串则返回`NULL`。`strpos()`系列函数区分大小写且二进制安全。

函数名中的额外`r`表示反向或从字符串末尾而不是开头开始。许多字符串函数都有一个从开头开始的版本和一个从字符串末尾开始的版本。所有这些函数在名称中都有一个额外的`r`。同样，名称中可以有一个额外的`i`表示不区分大小写。

两个`strpos()`函数都接受一个可选参数，用于指示搜索应从字符串中的哪个位置开始。这可用于重复搜索以检查字符串是否多次存在。这些函数区分大小写，但如果你不关心大小写，可以使用不区分大小写的版本，称为`stripos()`和`strripos()`。

同样，有函数可以查找字符或字符串在字符串中的第一次和最后一次出现。这些函数称为`strchr()`、`strrchr()`、`strstr()`和`strrstr()`。在内部，`strchr()`是`strstr()`的别名。返回值是从搜索字符串的第一次或最后一次出现开始以及字符串其余部分组成的第一个字符串。即使你搜索单个字符，这些函数也会返回从找到该字符的位置开始的字符串的其余部分。

### 工作原理

在下一个示例中，我们有一个长字符串，我们将查找字符串`'at'`的第一次和最后一次出现。

```
<?php

// 6_17.php

$s = "Lorem ipsum dolor sit amet, has dicit eleifend no, legimus " .

"voluptatibus ad mei. Ornatus sententiae vituperatoribus mel ea, " .

"at veri maiorum quaerendum vel, vis at deleniti vulputate. An vis " .

"quis percipitur reformidans. Eu agam tation vel, " .

"pri nemore discere no.";

$p_first = strpos($s, 'at');

$p_last = strrpos($s, 'at');

echo "'at' is first found at positionn {$p_first}\n";

echo "'at' is last found at positionn {$p_last}\n";

$p = strchr($s, 'at');

echo "{$p}\n";

$p = strrchr($s, 'at');

echo "{$p}\n";
```

首先，我们使用`strpos()`和`strrpos()`获取`'at'`第一次和最后一次出现的位置。这些函数返回一个整数，表示字符串中从 0 开始的索引位置。

接下来，我们使用`strchr()`和`strrchr()`获取从这些位置开始的字符串。示例的输出如下：

```
'at' is first found at positionn 65
'at' is last found at positionn 227
atibus ad mei. Ornatus sententiae vituperatoribus mel ea, at veri maiorum quaerendum vel, vis at deleniti vulputate. An vis quis percipitur reformidans. Eu agam tation vel, pri nemore discere no.
ation vel, pri nemore discere no.
```

请注意，这些函数会精确匹配正在搜索的字符串，即使该字符串是单词的一部分。

## 将字符串分割成子字符串

### 问题

字符串可用于存储不同类型的数据，当代码逻辑需要知道字符串的特定部分时，我们需要能够将字符串切割成小元素的函数。

### 解决方案

这由`substr()`函数处理。该函数接受一个字符串和一个或两个整数作为参数。第一个整数是起始位置（从 0 偏移量开始），第二个整数是可选的长度。如果省略长度参数，该函数将返回从起始位置开始的所有字符。

起始参数可以是负数，导致函数从字符串末尾开始。如果未提供长度，将`start`设置为`-3`将返回最后三个字符。如果起始位置是大于字符串长度的数字，`substr()`函数将返回`NULL`。如果起始值为负且长度值为小于或等于起始值的负值，则该函数将返回一个空字符串。

### 工作原理

在前面的示例中，我们看到了如何查找字符和字符串在另一个字符串中的位置。现在我们将研究如何使用它来获取字符串的特定部分。

如果我们采用相同的长字符串，并希望获取`'at'`第一次和最后一次出现之间的字符串内容，代码如下：

```
<?php

// 6_18.php

$s = "Lorem ipsum dolor sit amet, has dicit eleifend no, legimus " .

"voluptatibus ad mei. Ornatus sententiae vituperatoribus mel ea, " .

"at veri maiorum quaerendum vel, vis at deleniti vulputate. An vis " .

"quis percipitur reformidans. Eu agam tation vel, " .

"pri nemore discere no.";

$p_first = strpos($s, 'at');

$p_last = strrpos($s, 'at');

echo substr($s, $p_first + 2, $p_last - $p_first - 2) . "\n";
```



# PHP 字符串处理函数示例

由于我们只想要两个位置之间的内容，我们计算从第一个位置加上字符串长度开始的子串，并将子串的长度计为两个位置之间的差值，这里我们减去 2。

获取存储在字符串中的文件名的扩展名，以确定文件可能包含的内容类型，可以通过类似的方式实现。

```
<?php

// 6_19.php

$filename = "myfile.txt";

$ext = substr($filename, -3);

echo "Extension = {$ext}\n";
```

在这种情况下，子串的起始位置设置为`-3`。这意味着从字符串末尾开始数 3 个字符，并且没有提供长度参数，因此该函数将返回字符串的其余部分。这个示例只适用于长度为 3 个字符的扩展名。如果我们希望能够获取不同文件类型的扩展名，我们可以使用`strrpos()`来获取最后一个`.`出现的位置，并将其作为子串的起始点。我们需要将起始点加 1 来调整，因为句点不应包含在扩展名字符串中。

```
<?php

// 6_20.php

$filename = "myfile.txt";

$p = strrpos($filename, '.');

$ext = substr($filename, $p + 1);

echo "Extension = {$ext}\n";
```

此示例计算了文件名扩展名部分的起始点，但并非所有文件名都有扩展名部分。在这种情况下，`strrpos()`将返回`null`。为了解决这个问题，我们可以将其转换成一个函数，同时检查`.`的位置是否有效。

```
<?php

// 6_21.php

function GetExtension($filename) {

$ext = null;

$p = strrpos($filename, '.');

if ($p !== false) {

$ext = substr($filename, $p + 1);

}

return $ext;

}

$filename = "myfile.txt";

echo "Extension of {$filename} is " . GetExtension($filename) . "\n"; $filename = "myfile.info";

echo "Extension of {$filename} is " . GetExtension($filename) . "\n"; 136

CHAPTER 6 ■ STRINGS

$filename = "myfile";

echo "Extension of {$filename} is " . GetExtension($filename) . "\n"; $filename = ".cvsignore";

echo "Extension of {$filename} is " . GetExtension($filename) . "\n";
```

请注意，检查有效位置是通过`$p !== false`完成的。这是因为当搜索到的字符出现在字符串的第一个位置时，其值为`0`，简单的比较会得到`0 == false`，但在比较类型和值时，`0 !== false`。

该示例的输出如下所示：

```
Extension of myfile.txt is txt
Extension of myfile.info is info
Extension of myfile is
Extension of .cvsignore is cvsignore
```

## Recipe 6-10. 显示 HTML 实体

### 问题

当字符串用于存储 HTML 代码，并且您希望显示代码而不是让浏览器渲染代码时，您需要将某些字符转换为 HTML 实体。

### 解决方案

处理此问题的简单方法是使用`str_replace()`将`<`替换为`&lt;`，将`>`替换为`&gt;`。这需要两次函数调用。相反，我们可以使用`htmlspecialchars()`。此函数会将`&`、`'`、`"`、`<`和`>`转换为对应的 HTML 实体。字符串可能包含其他具有关联命名实体的子串。为了转换所有这些实体，您必须使用`htmlentities()`函数。

这两个函数都将要转换的字符串作为第一个参数，后面跟着三个可选参数，用于定义如何进行替换。有关这些参数的详细信息，请参阅 PHP 手册：

[`php.net/manual/en/function.htmlentities.php`](http://php.net/manual/en/function.htmlentities.php)

### 工作原理

每当您的网站允许用户输入文本值，这些值存储在数据库中，并随后呈现给同一用户或其他用户时，您可能希望阻止用户输入 HTML 内容，这些内容会被浏览器解析并渲染为 HTML，而不是用户实际输入的内容。如果您在博客页面上有一个评论部分，用户可能会在评论中输入一个图像标签。如果您不对此输入进行处理，当向用户显示时，该图像将作为评论的一部分显示出来，因为图像标签可以引用服务器上或任何连接到互联网的其他服务器上的任何 URL。

您可以在将值插入数据库之前或向用户显示之前调用`htmlentities()`，但不要两次都调用它。

```
<?php

// 6_22.php

$input = 'This is my comment <img src="https://example.com/image.jpg" />'; echo htmlentities($input);
```

此脚本的原始输出将显示以下内容：

```
This is my comment &lt;img src=&quot;https://example.com/image.jpg&quot; /&gt;
```

当发送到浏览器时，这将显示为文本，而不会被解释为图像标签。将字符串转换回有效的 HTML 字符串可以使用`html_entity_decode()`函数完成。

如果您在命令行上运行此脚本，输出将如下所示：

```
This is my comment &lt;img src=&quot;https://example.com/image.jpg&quot; /&gt;
```

如果您在浏览器中查看此输出，它将完全按照用户输入的样子显示，并且图像不会显示出来。

如果不采取任何预防措施，用户可以插入可能执行 JavaScript 或从其他站点加载脚本文件的`<script>`标签。在下一个示例中，我们使用`strip_tags()`函数从输入中删除所有标签。

```
<?php

// 6_23.php

$input = <<<HEREDOC

Comment

<script src="https://example.com/script.js"></script>

<script>

MyFunction();

</script>

<img src="https://example.com/image.jpg" />

This is my opinion about that!

HEREDOC;

$i = strip_tags($input);

echo $i;
```

尽管所有标签都被删除了，但该函数会保留标签之间的内容，浏览器能够将输出渲染为纯文本。输出将如下所示：

```
Comment
MyFunction();
This is my opinion about that!
```

请注意，所有换行符都被保留了，因为只删除了字符串中的标签部分。为了不创建无效的 HTML，在将字符串发送到浏览器之前，您可能仍然需要对字符串运行`htmlentities()`。

## Recipe 6-11. 为文件生成哈希值

### 问题

当创建一个资产管理系统，其中不同类型和大小的文件存储在文件系统上，并且您希望确保所有文件都是唯一的时，您可以将文件名作为键。然而，这将允许用户仅重命名文件并再次存储，从而导致同一文件被保存两次。

### 解决方案

解决方案是生成文件内容的哈希值。有多个不同的函数可以做到这一点。最简单（可能也是最古老）的方法是使用`md5()`函数，它根据字符串的内容返回一个 32 字节的字符串；如果使用`md5_file()`，则根据文件内容返回。更全面的哈希算法可以通过`sha1()`和`sha1_file()`获得。这些函数生成一个 40 字节的字符串。从 PHP 5.1.2 版本开始，哈希扩展默认包含在 PHP 中。此扩展提供了大量哈希算法，包括`md2`、`md5`、`sha1`、`sha256`、`sha512`以及更多。

### 工作原理

在引入哈希扩展之前，PHP 使用独立的函数如`md5()`和`sha1()`来生成哈希值，但通过`hash()`函数，可以使用单个函数生成多种不同的哈希值。`hash()`和`hash_file()`函数接受两个或三个参数。第一个参数是哈希算法的名称，第二个参数是要从中创建哈希的字符串或文件，最后一个可选参数指定函数是返回二进制数据还是返回小写十六进制来表示数据。



# 如果您正在创建用于存储图像和其他文件的服务，可能会遇到同一文件以相同或不同名称被多次上传的情况。但通过查看文件内容的哈希值，可以判断该文件是否已存在于系统中。为此，我们将创建一个包含每个文件 `id`、`filename` 和 `hash` 的数据库表。添加新文件时，我们检查 `hash` 是否已知，若已知则返回现有哈希；若未知则创建新记录并存储文件。

```php
<?php

// 6_24.php

// create table files (
//   id int auto_increment,
//   hash varchar(255),
//   filename varchar(255),
//   primary key(id)
// );
// create index ixHash on files(hash);

$con = mysqli_connect('127.0.0.1', 'root', 'secret', 'book');

function GetFile($hash) {
  global $con;
  $file = null;

  $sql = "select * from files where hash = '{$hash}'";
  $rs = mysqli_query($con, $sql);
  if ($rs) {
    $file = mysqli_fetch_row($rs);
    mysqli_free_result($rs);
  }
  return $file;
}

function SaveFile($filename) {
  global $con;
  $hash = hash_file('sha256', $filename);
  if (!GetFile($hash)) {
    mysqli_query($con, "insert into files (hash, filename) values ('{$hash}', '{$filename}')");
  }
}

SaveFile('6_1.php');
SaveFile('6_12.php');
SaveFile('6_1.php');

$rs = mysqli_query($con, "select * from files");
if ($rs) {
  while ($row = mysqli_fetch_row($rs)) {
    print_r($row);
  }
  mysqli_free_result($rs);
}

mysqli_close($con);
```

函数 `GetFile()` 会在数据库中已存在该哈希值时返回存储的值，若数据库中不存在则返回 `null`。函数 `SaveFile()` 会在创建新记录前调用 `GetFile()` 函数以防止重复记录。这在示例末尾插入三个文件时得到体现。

第一个和最后一个文件是同一个文件，因此只会创建两条记录，输出结果如下：

```
Array
(
    [0] => 1
    [1] => 866b1551930506766708fbb73cfdfc6fd5038cf812cb64983f43d771b8e0ff71
    [2] => 6_1.php
)
Array
(
    [0] => 2
    [1] => 125a71508e064ee34662498fcb040585fd1bf539cf4f09b203a79adffba8fc99
    [2] => 6_12.php
)
```

## 配方 6-12. 存储密码

### 问题

许多网站允许用户注册并创建账户，以便访问额外功能或查看购物订单状态。其中许多网站使用电子邮件地址作为用户 ID 并采用某种形式的密码。在数据库中以明文形式存储密码并不安全。如果网站被入侵或硬盘落入不法分子手中，所有密码都可能被泄露。

### 解决方案

我们可以存储密码的哈希值，而不是存储明文密码。可以使用上一配方中的 `md5()`、`sha1()` 或 `hash()` 函数，如果是 PHP 5.5 或更高版本，还可以使用 `password_hash()` 函数。建议数据库模式为密码列至少允许 60 个字符（255 个字符更佳），因为哈希算法可能会随时间变化以提升安全性并生成更强的哈希值。

### 实现原理

如果您有一个名为 `users` 的表，包含 `login` 和 `password` 两列，那么应在存储用户密码前以及验证登录时提供的登录名和密码是否有效之前使用 `password_hash()` 函数。

```php
<?php

// 6_25.php

//create table myusers (
//  id int auto_increment,
//  login varchar(255),
//  password varchar(255),
//  primary key(id)
//);

$con = mysqli_connect('127.0.0.1', 'root', 'secret', 'book');

function CreateUser($login, $password) {
  global $con;
  $login = mysqli_escape_string($con, $login);
  $password = mysqli_escape_string($con, password_hash($password, PASSWORD_BCRYPT));
  mysqli_query($con, "insert into myusers (login, password) values ('{$login}', '{$password}')");
}

function CheckUser($login, $password) {
  global $con;
  $user = null;
  $login = mysqli_escape_string($con, $login);
  $rs = mysqli_query($con, "select * from myusers where login='{$login}'");
  if ($rs) {
    $user = mysqli_fetch_assoc($rs);
    if ($user && !password_verify($password, $user['password'])) {
      $user = null;
    }
    mysqli_free_result($rs);
  }
  return $user;
}

CreateUser('john@example.com', 'A secret Password');

if (CheckUser('john@example.com', 'A secret Password')) {
  echo "用户存在且密码正确\n";
}
else {
  echo "用户不存在或密码错误\n";
}
```

使用 MySQL 命令行工具显示表中的行，结果类似如下：

```
MariaDB [book]> select password from myusers;
+--------------------------------------------------------------+
| password                                                     |
+--------------------------------------------------------------+
| $2y$10$V4KWuQIzssFRmlQeZNLPeegsyQ0MFp0vjUNl3FP9.7k5E4EiB5INi |
+--------------------------------------------------------------+
1 row in set (0.00 sec)
```

# 第 7 章：文件和目录

在本章中，我们将讨论 PHP 中使用文件的两种基本方式。第一种是包含 PHP 代码的脚本，形式为主脚本文件以及脚本可能使用的任何包含文件。

第二种形式是从本地或远程文件系统读取更多内容或数据，并由脚本进行处理。

## 配方 7-1. Include 和 Require

### 问题

当 PHP 用于执行或解释脚本时，它总是从一个文件开始。该文件可以是 PHP 命令行版本（`php-cli`）的输入参数，也可以是从 Web 服务器请求间接传递给 PHP 的文件。

让单个文件包含每个请求所需的所有代码会导致脚本文件庞大且代码重复。此外，将代码拆分成服务于特定目的的小单元也更有意义。

这通常通过在每个文件中定义单个类来实现。

### 解决方案

这正是 `include` 和 `require` 发挥作用的地方。每当多个脚本共享代码时，您可以将其放入单独的文件中，并在需要的地方包含该代码。`include` 和 `require` 的基本区别在于，当请求的文件在文件系统中不存在时，这些语句的反应不同。`Include` 会生成一个警告，而 `require` 会生成一个致命错误并停止脚本执行。

关于如何组织包含文件，有两种不同的思路。第一种是将它们放在与主脚本相同的目录结构中。这通常迫使开发人员使用与主脚本相同的扩展名，以避免用户直接请求这些文件时暴露其内容。如果 Web 服务器配置为将带有 `.php` 扩展名的文件传递给 PHP 解释器，那么带有 `.inc` 等其他扩展名的文件可能会被直接传递，如果文件包含数据库凭据和其他秘密信息，最终用户可能能够访问这些信息。简单的解决方法是确保 Web 服务器配置为将所有 PHP 文件交给解释器处理。用于配置数据库凭据的包含文件很可能不会生成任何输出，因此用户看到的内容将是一个空白文档。

另一种思路是将所有包含文件放在用于定义网站文档根的目录结构之外。在这种情况下，文件可以使用任何扩展名，并且无法直接暴露给用户，因为 Web 服务器不知道如何访问这些文件。在我看来，这是更可取的方法。

© Frank M. Kromann 2016

F.M. Kromann, *PHP and MySQL Recipes*, DOI 10.1007/978-1-4842-0605-8_7


```markdown
