# 22. 魔术方法

PHP 引擎内部会调用类中实现的若干方法，这些方法被称为**魔术方法**，识别它们很容易，因为其名称均以两个下划线开头。表 22-1 列出了目前已讨论过的魔术方法。

表 22-1. 魔术方法

| 名称 | 描述 |
| --- | --- |
| `__construct(...)` | 创建新实例时调用。 |
| `__destruct()` | 对象不再被引用时调用。 |
| `__call($name, $array)` | 在对象上下文中调用不可访问的方法时调用。 |
| `__callStatic($name, $array)` | 在静态上下文中调用不可访问的方法时调用。 |
| `__get($name)` | 读取不可访问属性数据时调用。 |
| `__set($name, $value)` | 写入不可访问属性数据时调用。 |
| `__isset($string)` | 对不可访问属性使用 `isset` 或 `empty` 时调用。 |
| `__unset($string)` | 对不可访问属性使用 `unset` 时调用。 |

除此之外还有六个魔术方法，与其他方法一样，它们可以在类中实现以提供特定功能。

表 22-2. 更多魔术方法

| 名称 | 描述 |
| --- | --- |
| `__toString()` | 对象转换为字符串时调用。 |
| `__invoke(...)` | 对象转换为函数时调用。 |
| `__sleep()` | 由 `serialize` 调用。执行清理任务并返回一个待序列化的变量数组。 |
| `__wakeup()` | 由 `unserialize` 调用，用于重建对象。 |
| `__set_state($array)` | 由 `var_export` 调用。该方法必须为静态方法，其参数包含导出的属性。 |
| `__clone()` | 对象被克隆后调用。 |

## `__toString`

当对象在需要字符串的上下文中使用时，PHP 引擎会查找名为 `__toString` 的方法，以获取对象的字符串表示形式。

```php
class MyClass
{
    public function __toString()
    {
        return 'Instance of ' . __CLASS__;
    }
}
$obj = new MyClass();
echo $obj; // 输出 "Instance of MyClass"
```

除字符串类型外，无法定义对象在评估为其他类型时的行为。

## `__invoke`

`__invoke` 方法允许将对象当作函数来使用。调用对象时提供的参数会作为 `__invoke` 函数的参数。

```php
class MyClass
{
    public function __invoke($arg)
    {
        echo $arg;
    }
}
$obj = new MyClass();
$obj('Test'); // 输出 "Test"
```

### 对象序列化

序列化是将数据转换为字符串格式的过程。这对于在数据库或文件中存储对象非常有用。在 PHP 中，内置的 `serialize` 函数执行此对象到字符串的转换，而 `unserialize` 则将字符串还原为原始对象。`serialize` 函数可处理所有类型，但不包括资源类型（用于保存数据库连接和文件句柄等）。请考虑以下简单的数据库类。

```php
class MyConnection
{
    public $link, $server, $user, $pass;
    public function connect()
    {
        $this->link = mysql_connect($this->server,
            $this->user,
            $this->pass);
    }
}
```

当此类被序列化时，数据库连接会丢失，并且保存连接的 `$link` 资源类型变量会存储为 `null`。

```php
$obj = new MyConnection();
// ...
$bin = serialize($obj);   // 序列化对象
$obj = unserialize($bin); // 还原对象
```

为了更精细地控制对象数据的序列化和反序列化方式，该类可以实现 `__sleep` 和 `__wakeup` 方法。

### `__sleep`

`__sleep` 方法由 `serialize` 调用，需要返回一个包含将被序列化的属性的数组。此数组不得包含私有或受保护的属性，因为 `serialize` 无法访问它们。该方法还可以在序列化发生前执行清理任务，例如将待处理数据提交到存储介质。

```php
public function __sleep()
{
    return array('server', 'user', 'pass');
}
```

请注意，这些属性以字符串形式返回给 `serialize`。其中 `$link` 资源类型的指针未包含在数组中，因为它无法被序列化。为了重新建立数据库连接，可以使用 `__wakeup` 方法。

### Wakeup

对序列化后的对象调用 `unserialize` 会调用 `__wakeup` 方法来恢复该对象。该方法不接受参数，也无需返回值。它用于重新建立资源类型的变量，以及执行对象反序列化后可能需要的其他初始化任务。在本例中，它重新建立了 MySQL 数据库连接。

```php
public function __wakeup()
{
    if(isset($this->server, $this->user, $this->$pass))
        $this->connect();
}
```

请注意，这里用多个参数调用了 `isset` 结构，在这种情况下，只有当所有参数都已设置时，它才会返回 `true`。

### Set State

`var_export` 函数会检索可用作有效 PHP 代码的变量信息。下面这个例子展示了该函数用于一个对象时的用法。

```php
class Fruit
{
    public $name = 'Lemon';
}
$export = var_export(new Fruit(), true);
```

由于对象是一种复杂类型，没有通用的语法来构造它及其成员。相反，`var_export` 会生成以下字符串。

```php
Fruit::__set_state(array( 'name' => 'Lemon', ))
```

为了构造对象，该字符串依赖于为该对象定义的静态 `__set_state` 方法。如所示，`__set_state` 方法接受一个关联数组，其中包含对象每个属性的键值对，包括私有和受保护的成员。

```php
static function __set_state(array $array)
{
    $tmp = new Fruit();
    $tmp->name = $array['name'];
    return $tmp;
}
```

在 `Fruit` 类中定义此方法后，现在可以使用 `eval` 结构解析导出的字符串，以创建一个相同的对象。

```php
eval('$MyFruit = ' . $export . ';');
```

### Object Cloning

将对象赋值给变量只会创建对该对象的新引用。要复制对象，可以使用 `clone` 运算符。

```php
class Fruit {}
$f1 = new Fruit();
$f2 = $f1;       // 复制对象引用
$f3 = clone $f1; // 复制对象
```

克隆对象时，其属性会被复制到新对象。然而，它可能包含的任何子对象不会被克隆，因此它们在副本之间是共享的。这就是 `__clone` 方法的用途。该方法在克隆完成后，于克隆出的副本上被调用。它可用于克隆任何子对象。

```php
class Apple {}
class FruitBasket
{
    public $apple;
    function __construct() { $apple = new Apple(); }
    function __clone()
    {
        $this->apple = clone $this->apple;
    }
}
```

# 23. 用户输入

当 HTML 表单提交到 PHP 页面时，这些数据对该脚本来说就变得可用。

## HTML 表单

HTML 表单有两个必需的属性：`action` 和 `method`。`action` 属性指定表单数据提交到的脚本。例如，以下表单将名为 `myString` 的输入属性提交到 `mypage.php` 脚本文件。

表单元素的另一个必需属性指定发送方法，可以是 GET 或 POST。

## 使用 POST 发送

如果使用 POST 方法发送表单，数据可以通过 `$_POST` 数组获取。属性的名称是该关联数组中的键。通过 POST 方法发送的数据在页面的 URL 上不可见，但这同时也意味着页面状态无法通过例如收藏页面等方式来保存。

```php
echo $_POST['myString'];
```

### 使用 GET 发送

POST 的替代方法是使用 GET 方法发送表单数据，并通过`$_GET`数组检索。变量随后会显示在地址栏中，这实际上会在页面被收藏并重新访问时保持页面状态。

```php
echo $_GET['myString'];
```

由于数据包含在地址栏中，变量不仅可以通过 HTML 表单传递，还可以通过 HTML 链接传递。然后可以使用`$_GET`数组相应地更改页面状态。这提供了一种将变量从一个页面传递到另一个页面的方法。

```
<a href="mypage.php?myString=Foo+Bar">链接</a>
```

### 请求数组

如果不在乎数据是使用 POST 还是 GET 方法发送的，可以使用`$_REQUEST`数组。该数组通常包含`$_GET`和`$_POST`数组，但也可能包含`$_COOKIE`数组。

```php
echo $_REQUEST['myString']; // "Foo Bar"
```

`$_REQUEST`数组的内容可以在 PHP 配置文件中设置¹，并且因 PHP 发行版而异。出于安全考虑，通常不包含`$_COOKIE`数组。

### 安全问题

任何用户提供的数据都可能被篡改；因此，在使用前应对其进行验证和清理。验证意味着要确保数据在数据类型、范围和内容方面符合预期形式。例如，以下代码验证电子邮件地址。

```php
if(!filter_var($_POST['email'], FILTER_VALIDATE_EMAIL))
    echo "无效的电子邮件地址";
```

清理是指禁用用户输入中潜在的恶意代码。这是通过根据将要使用输入的语言规则对代码进行转义来完成的。例如，如果数据要发送到数据库，则需要使用`mysql_real_escape_string`函数进行清理，以禁用任何嵌入的 SQL 代码。

```php
// 为数据库使用进行清理
$name = mysql_real_escape_string($_POST['name']);
// 执行 SQL 命令
$sql = "SELECT * FROM users WHERE user='" . $name . "'";
$result = mysql_query($sql);
```

当用户提供的数据作为文本输出到网页时，应使用`htmlspecialchars`函数。它会禁用任何 HTML 标记，以便用户输入得以显示但不会被解释执行。

```php
// 为网页使用进行清理
echo htmlspecialchars($_POST['comment']);
```

### 提交数组

通过在表单中的变量名后添加数组方括号，可以将表单数据分组到数组中。这适用于所有表单输入元素，包括`<input>`、`<select>`和`<textarea>`。

这些元素也可以被赋予它们自己的数组键。

```html
<input type="text" name="myArr[0]" />
<input type="text" name="myArr[1]" />
<input type="text" name="myArr[name]" />
```

提交后，该数组即可在脚本中使用。

```php
$val1 = $_POST['myArr'][0];
$val2 = $_POST['myArr'][1];
$name = $_POST['myArr']['name'];
```

表单的`<select>`元素具有一个属性，允许从列表中选择多个项目。

```html
<select name="myArr[]" multiple>
    <option value="apple">苹果</option>
    <option value="orange">橘子</option>
    <option value="pear">梨</option>
</select>
```

当在表单中包含此多选元素时，数组方括号对于在脚本中检索所选值来说变得必不可少。

```php
foreach ($_POST['myArr'] as $item)
    echo $item . ' '; // 例如 "苹果 橘子 梨"
```

### 文件上传

HTML 表单提供了一种文件输入类型，允许将文件上传到服务器。要使文件上传起作用，表单的可选`enctype`属性必须设置为`"multipart/form-data"`，如下例所示。

关于上传文件的信息存储在`$_FILES`数组中。该关联数组的键见表 23-1。

**表 23-1. `$_FILES`数组的键**

| 名称        | 描述                   |
|-------------|----------------------|
| `name`      | 上传文件的原始名称。     |
| `tmp_name`  | 服务器上临时副本的路径。 |
| `type`      | 文件的 MIME 类型。      |
| `size`      | 文件大小（以字节为单位）。|
| `error`     | 错误代码。             |

接收到的文件仅临时存储在服务器上。如果脚本未保存它，它将被删除。以下展示了一个如何保存文件的简单示例。该示例检查错误代码以确保文件成功接收，如果是，则将文件移出临时文件夹进行保存。在实践中，你可能还需要检查文件大小和类型以决定是否保留该文件。

```php
$dest = 'upload\\' . basename($_FILES['myfile']['name']);
$file = $_FILES['myfile']['tmp_name'];
$err  = $_FILES['myfile']['error'];
if($err == 0 && move_uploaded_file($file, $dest))
echo '文件成功上传';
```

这个例子中出现了两个新函数。`move_uploaded_file`函数会检查第一个参数是否为有效的上传文件，如果是，则将其移动到第二个参数指定的路径并重命名为该文件名。指定的文件夹必须已存在，若函数成功移动文件，则返回`true`。另一个新函数是`basename`。它返回路径中的文件名部分，包含文件扩展名。

## 超全局变量（Superglobals）

如本章所示，PHP 内置了若干关联数组，用于为 PHP 脚本提供外部数据。这些数组被称为超全局变量，因为它们可在所有作用域中自动生效。PHP 共有九个超全局变量，表 23-2 对每个变量进行了简要说明。

**表 23-2.** 超全局变量

| 名称 | 描述 |
| --- | --- |
| `$GLOBALS` | 包含所有全局变量，包括其他超全局变量。 |
| `$_GET` | 包含通过 HTTP GET 请求发送的变量。 |
| `$_POST` | 包含通过 HTTP POST 请求发送的变量。 |
| `$_FILES` | 包含通过 HTTP POST 文件上传发送的变量。 |
| `$_COOKIE` | 包含通过 HTTP Cookie 发送的变量。 |
| `$_SESSION` | 包含存储在用户会话中的变量。 |
| `$_REQUEST` | 包含`$_GET`、`$_POST`以及可能包含的`$_COOKIE`变量。 |
| `$_SERVER` | 包含关于 Web 服务器及其收到的请求的信息。 |
| `$_ENV` | 包含由 Web 服务器设置的所有环境变量。 |

变量`$_GET`、`$_POST`、`$_COOKIE`、`$_SERVER`和`$_ENV`的内容包含在`phpinfo`函数生成的输出中。该函数还会显示 PHP 配置文件`php.ini`的常规设置以及关于 PHP 的其他信息。

```php
phpinfo(); // 显示 PHP 信息
```

## 脚注

[`http://www.php.net/manual/en/ini.core.php#ini.request-order`](http://www.php.net/manual/en/ini.core.php#ini.request-order)

## 24. Cookie

Cookie 是保存在客户端计算机上的一个小文件，可用于存储与该用户相关的数据。

### 创建 Cookie

要创建 Cookie，可使用`setcookie`函数。该函数必须在向浏览器发送任何输出之前调用。它有三个必选参数，分别包含 Cookie 的名称、值和过期时间。

```php
setcookie("lastvisit", date("H:i:s"), time() + 60*60);
```

这里的值由`date`函数设置，该函数根据指定的格式字符串返回格式化后的时间字符串。过期时间以秒为单位，通常相对于通过`time`函数获取的当前时间（秒数）进行设置。在这个例子中，Cookie 在一小时后过期。

### Cookie 数组

一旦为用户设置了 Cookie，该 Cookie 将在用户下次浏览页面时发送过来；随后可通过 `$_COOKIE` 数组访问它。

```php
if (isset($_COOKIE['lastvisit']))
echo "上次访问时间：" . $_COOKIE['lastvisit'];
```

### 删除 Cookie

可以通过重新创建相同的 Cookie 并设置一个旧的过期时间来手动删除 Cookie。当浏览器关闭时，该 Cookie 会被移除。

```php
setcookie("lastvisit", 0, 0);
```

## 25. 会话（Sessions）

会话提供了一种让变量在多个网页间保持可访问性的方法。与 Cookie 不同，会话数据存储在服务器上。

### 启动会话

要开始一个会话，可使用 `session_start` 函数。该函数必须在向网页发送任何输出之前出现。

`session_start` 函数会在客户端计算机上设置一个 Cookie，其中包含一个用于将客户端与会话关联的 ID。如果客户端已有进行中的会话，该函数会恢复该会话，而不是启动新会话。

### 会话数组

会话启动后，可使用 `$_SESSION` 数组来存储和检索会话数据。例如，以下代码用于存储页面浏览次数。第一次浏览页面时，会话元素被初始化为 1。

```php
if(isset($_SESSION['views']))
$_SESSION['views'] += 1;
else
$_SESSION['views'] = 1;
```

只要在页面的顶部调用了 `session_start`，就可以从该域中的任何页面检索此元素。

```php
echo '浏览次数：' . $_SESSION['views'];
```

### 删除会话

会话会一直持续到用户离开网站；之后，垃圾回收器可以自由删除该会话。若要手动删除会话变量，可使用 `unset` 函数。若要删除所有会话变量，可使用 `session_destroy` 函数。

```php
unset($_SESSION['views']); // 销毁会话变量
session_destroy();         // 销毁会话
```

## 26. 命名空间（Namespaces）

命名空间提供了一种避免命名冲突并将命名空间成员组织成层级结构的方法。任何代码都可以包含在命名空间中，但只有四种代码结构会受到影响：类、接口、函数和常量。

### 创建命名空间

未包含在命名空间中的代码结构属于全局命名空间。

```php
// 全局代码/命名空间
class MyClass {}
```

要将代码结构分配到另一个命名空间，需定义一个 `namespace` 指令。该指令下方的代码结构都属于该命名空间。命名空间的命名约定是全部使用小写。

```php
namespace my;
// 属于 my 命名空间
class MyClass {}
```

包含命名空间代码的脚本文件必须在文件最顶部、其他任何代码、标记或空白之前声明命名空间。`declare` 语句是此规则的例外，因为它必须放在命名空间声明之前。

### 嵌套命名空间

命名空间可以嵌套任意多层，以进一步定义命名空间的层级结构。类似于 Windows 中的目录和文件，命名空间及其成员之间用反斜杠字符分隔。

```php
namespace my\sub; class MyClass {} // my\sub\MyClass
```

### 替代语法

另外，命名空间也可以使用其他编程语言中常用的大括号语法来定义。与常规语法一样，命名空间外部不能存在任何文本或代码。

可以在同一个文件中声明多个命名空间，但这不被视为良好的编码实践。如果全局代码要与命名空间代码结合使用，则必须使用大括号语法。此时，全局代码被包裹在一个未命名的命名空间块中。

```php
// 命名空间代码
namespace my
{
const PI = 3.14;
}
// 全局代码
namespace
{
echo my\PI; // "3.14"
}
```

与其他 PHP 结构不同，同一个命名空间可以在多个文件中定义。这允许将命名空间成员分散到多个文件中。

### 引用命名空间

命名空间成员可以通过三种方式引用：完全限定名称、限定名称和未限定名称。完全限定名称始终可以使用。它由全局前缀运算符（`\`）开头，后跟命名空间路径和成员名称。全局前缀运算符表示该路径相对于全局命名空间。

```php
namespace my
{
class MyClass {}
}
namespace other
{
// 完全限定名称
$obj = new \my\MyClass();
}
```

限定名称包含命名空间路径，但不包含全局前缀运算符。因此，只有当所需成员在当前命名空间层级结构的下级命名空间中定义时，才能使用限定名称。

```php
namespace my
{
class MyClass {}
}
namespace
{
// 限定名称
$obj = new my\MyClass();
}
```

仅使用成员名称，即未限定名称，只能在定义该成员的命名空间内部使用。

```php
namespace my
{
class MyClass {}
// 未限定名称
$obj = new MyClass();
}
```