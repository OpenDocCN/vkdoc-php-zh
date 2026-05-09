# 创建`customer`和`shipping_region`表

现在，你可以按照下一步练习中的步骤来构建`customer`和`shipping_region`表。

## 练习：创建数据库表

1. 加载`pgAdmin III`，并连接到`hatshop`数据库。
2. 点击“工具”→“查询工具”（或点击工具栏上的“SQL”按钮）。一个新的查询窗口将出现。
3. 使用查询工具执行以下代码，该代码在`hatshop`数据库中创建`shipping_region`表：

```sql
-- Create shipping_region table
CREATE TABLE shipping_region
(
shipping_region_id SERIAL NOT NULL,
shipping_region VARCHAR(100) NOT NULL,
CONSTRAINT pk_shipping_region_id PRIMARY KEY (shipping_region_id)
);
```

4. 现在将值`'Please Select'`、`'US / Canada'`、`'Europe'`和`'Rest of the World'`添加到`shipping_region`表中。`'Please Select'`的`shipping_region_id`值必须始终为 1——这很重要！使用查询工具执行以下 SQL 代码，将上述值添加到`shipping_region`表中：

```sql
-- Populate shipping_region table
INSERT INTO shipping_region (shipping_region_id, shipping_region) VALUES (1, 'Please Select');
INSERT INTO shipping_region (shipping_region_id, shipping_region) VALUES (2, 'US / Canada');
INSERT INTO shipping_region (shipping_region_id, shipping_region) VALUES (3, 'Europe');
INSERT INTO shipping_region (shipping_region_id, shipping_region) VALUES (4, 'Rest of World');

-- Update the sequence
ALTER SEQUENCE shipping_region_shipping_region_id_seq RESTART WITH 5;
```

5. 使用查询工具执行以下代码，该代码在`hatshop`数据库中创建`customer`表：

```sql
-- Create customer table
CREATE TABLE customer
(
customer_id SERIAL NOT NULL,
name VARCHAR(50) NOT NULL,
email VARCHAR(100) NOT NULL,
password VARCHAR(50) NOT NULL,
credit_card TEXT,
address_1 VARCHAR(100),
address_2 VARCHAR(100),
);
```



`city` `VARCHAR(100)`,,

`region` `VARCHAR(100)`,,

`postal_code` `VARCHAR(100)`,,

`country` `VARCHAR(100)`,,

`shipping_region_id` `INTEGER` `NOT NULL` `DEFAULT 1`,,

`day_phone` `VARCHAR(100)`,,

`eve_phone` `VARCHAR(100)`,,

`mob_phone` `VARCHAR(100)`,,

`CONSTRAINT pk_customer_id PRIMARY KEY (customer_id)`, `CONSTRAINT fk_shipping_region_id FOREIGN KEY (shipping_region_id) REFERENCES shipping_region (shipping_region_id)`

`ON UPDATE RESTRICT ON DELETE RESTRICT`,

`CONSTRAINT uk_email UNIQUE (email)`

);

客户的信用卡信息将以加密格式存储，确保无人能够访问这些信息。但与密码不同，当订单处理流程需要时，你必须能够检索到信用卡信息，因此不能简单地使用哈希算法（哈希算法是单向的）。你将通过多个业务层类来实现信用卡数据加密功能，接下来将会看到这些类。

## 实现安全类

到目前为止，以下两个领域需要安全功能：

- 密码哈希
- 信用卡加密

这两项任务均由业务层类完成，这些类将保存在 `business` 目录下的以下文件中：

- `password_hasher.php`：包含 `PasswordHasher` 类，其中包含静态方法 `Hash()`，用于返回所提供密码的哈希值。
- `secure_card.php`：包含 `SecureCard` 类，代表一张信用卡。该类可以接收信用卡信息，并以加密格式提供访问。该类也可以接收加密的信用卡数据，并提供解密后的信息访问。
- `symmetric_crypt.php`：此文件中的 `SymmetricCrypt` 类被 `SecureCard` 用于加密和解密数据。这意味着，如果你需要更改加密方法，只需修改此处的代码，而无需改动 `SecureCard` 类。

我们先来看哈希的代码，然后再看加密。

## 在业务层中实现哈希功能

哈希是一种获取表示对象的唯一值的方法。用于将源字节数组转换为哈希字节数组的算法各不相同。最常用的哈希算法是 MD5（消息摘要，所生成哈希代码的另一种称呼），它生成 128 位的哈希值。遗憾的是，许多攻击都是基于针对 MD5 哈希构建的词典。另一种流行的哈希算法是 SHA1（安全哈希算法），它生成 160 位的哈希值。普遍认为 SHA1 比 MD5 更安全（尽管速度较慢）。

在 HatShop 的实现中，你将使用 SHA1，不过如果需要其他安全类型，也很容易更改。现在，你将在以下练习中实现 `PasswordHasher` 类。

> **注意：** PHP 默认不支持 `mhash` 和 `mcrypt`，这是本章用于哈希和加密的库。请参阅附录 A 了解如何启用对 `mhash` 和 `mcrypt` 的支持。

### 练习：实现 `PasswordHasher` 类

要实现 `PasswordHasher` 类，请遵循以下步骤：

**1.** 在 `include/config.php` 文件末尾添加以下行。这定义了一个随机值（你可以随意更改），用于在哈希之前添加到密码中。

```php
// 用于哈希的随机值
define('HASH_PREFIX', 'K1-');
```

**2.** 在 `business` 目录中创建一个名为 `password_hasher.php` 的新文件，并编写 `PasswordHasher` 类：

```php
<?php
class PasswordHasher
{
    public static function Hash($password, $withPrefix = true)
    {
        if ($withPrefix)
            $hashed_password = sha1(HASH_PREFIX . $password);
        else
            $hashed_password = sha1($password);
        return $hashed_password;
    }
}
?>
```

**3.** 接下来，编写一个简单的测试页面来测试 `PasswordHasher` 类。在 `hatshop` 文件夹中创建一个名为 `test_hasher.php` 的新文件，并在其中包含以下代码：

```php
<?php
if (isset ($_GET['to_be_hashed']))
{
    require_once 'include/config.php';
    require_once BUSINESS_DIR . 'password_hasher.php';
    $original_string = $_GET['to_be_hashed'];
    echo '"' . $original_string . '" 的哈希值是 ' .
        PasswordHasher::Hash($original_string, false);
    echo '<br />';
    echo '... 而 "' . HASH_PREFIX . $original_string .
        '"（秘密前缀与密码连接）的哈希值是 ' .
        PasswordHasher::Hash($original_string, true);
}
?>
<br /><br />
<form action="test_hasher.php">
请输入您的密码：
<input type="text" name="to_be_hashed" /><br />
<input type="submit" value="计算哈希" />
</form>
```

**4.** 在你喜欢的浏览器中加载 `test_hasher.php` 文件，输入要哈希的密码，并查看如图 11-1 所示的结果。

**图 11-1.** *测试密码哈希功能*

### 工作原理：哈希功能

`PasswordHasher` 类中的代码非常简单。默认情况下，静态 `Hash()` 方法返回一个字符串的哈希值，该字符串由秘密前缀与密码连接而成。

你可能会好奇秘密前缀的作用。正如你可能已经猜到的，这与安全性有关。如果你的数据库被盗，攻击者可能会尝试将哈希后的密码值与一个大型哈希值字典进行匹配，该字典看起来像这样：

```
word1 .... sha1(word1)
word2 .... sha1(word2)
...
word10000 .... sha1(word10000)
```

如果两个哈希值匹配，则意味着原始字符串（在本例中为客户密码）也匹配。

在哈希之前将秘密前缀附加到密码中，可以降低对哈希密码数据库进行字典攻击的风险，因为被哈希的字符串（秘密前缀 + 密码）不太可能出现在大型的“密码-哈希值”对字典中。

`test_hasher.php` 页面测试了你新创建的 `PasswordHasher` 类。

> **注意：** 你也可以通过使用 PostgreSQL 加密函数在数据库级别处理哈希。
> 首先，你需要将加密函数添加到 PostgreSQL。Unix 用户应查看 PostgreSQL 源码中的 `contrib/pgcrypto` 目录并按照说明操作（详细说明请参阅附录 A）。然后，例如，你可以执行以下 PostgreSQL 语句来查看 PostgreSQL SHA1 的实际效果：
> ```sql
> SELECT ENCODE(DIGEST('freedom', 'sha1'), 'hex');
> ```
> 当然，当依赖 PostgreSQL 的哈希功能时，密码会以“明文格式”传输到你的 PostgreSQL 服务器，因此如果 PostgreSQL 服务器位于另一个网络（尽管这种情况不太常见），你必须通过使用 SSL 连接来保护 Web 服务器和 PostgreSQL 服务器之间的连接。
> 这可以通过在 PHP 代码中处理哈希来避免，这也提供了更好的可移植性，因为它不依赖于 PostgreSQL 特有的函数。请记住，出于同样的可移植性原因，我们选择使用 PDO 而不是 PHP 的 PostgreSQL 特有函数。

## 在业务层中实现加密功能

加密以多种形式和样式存在，并且一直是一个热门话题。尽管关于这个主题有大量建议，但并没有一种确定的数据加密解决方案。一般来说，加密有两种形式：

**对称加密：** 使用单个密钥来加密和解密数据。


