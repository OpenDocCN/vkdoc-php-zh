# PostgreSQL 触发器

在上一章中，我们探讨了 PostgreSQL 中的用户自定义函数，特别是 PL/pgSQL 函数。本章将深入探讨用户自定义函数最常见的用途之一——驱动触发器，并涵盖以下内容：

- 不同类型触发器的定义及其触发时机
- 创建、修改和删除触发器的语法
- 普通函数与触发器函数之间的区别

触发器是非常强大的工具，可用于在数据库内部执行一系列任务，例如审计数据集、记录事件发生情况，或限制对特定数据的访问。

### 什么是触发器？

从广义上讲，*触发器*定义了基于数据库中特定事件发生的特定操作。在 PostgreSQL 中，这指的是基于对特定表执行的某个操作而执行的存储过程。所有触发器都由六个特性定义：

- 触发器的名称
- 触发器应触发的时机
- 触发器应响应的事件
- 触发器应作用的数据表
- 执行的频率
- 应调用的函数

通过组合这六个特性，PostgreSQL 允许你通过其触发器系统创建相当广泛的功能。

### 添加触发器

你可以使用 `CREATE TRIGGER` 命令向 PostgreSQL 添加触发器。发出此命令时，你将定义触发器的六个不同特性。语法如下：

```
CREATE TRIGGER 名称
{ BEFORE | AFTER }
{ 事件[ OR ... ] }
ON 表名
[ FOR [ EACH ] { ROW | STATEMENT } ]
EXECUTE PROCEDURE 函数名 ( 参数 )
```

如你所见，每个部分对应六个特性之一。第一部分指定触发器的名称。触发器名称对于任何给定的表必须是唯一的，但可以在不同的表中重复。触发器名称的另一个重要方面是，指定在同一时刻对同一表执行的触发器将按字母顺序执行。

触发器定义的第二部分设置了触发器应在事务中的哪个时间点执行。`BEFORE` 触发器会在触发查询所做的更改应用于事务*之前*执行其过程。`AFTER` 触发器会在触发查询所做的更改在事务中生效*之后*执行其过程。

事件部分指定了触发器将执行的查询类型，即 `DELETE`、`INSERT` 或 `UPDATE`，但不能是 `SELECT`。如果需要，你可以让触发器响应多个事件，例如同时响应插入和更新。

接下来，你需要定义触发器将作用于哪个表。一个触发器只能分配给一个表，但如果过程编写得足够通用，则不同表上的多个触发器可以调用同一个过程。

下一步是定义所执行函数的调用频率：每条语句执行一次，或者受语句影响的每一行执行一次。当触发器按行执行时，被调用的函数可以通过适当地使用 `NEW` 或 `OLD` 结构来访问所影响每一行中的数据，从而使你能够逐行操作每个被修改行的数据。语句级触发器仅为整个查询调用一次，并且无法访问被操作的数据。我们稍后会进一步讨论 `NEW` 和 `OLD` 结构。

最后一步是定义将由触发器执行的过程。并非所有过程都能被触发器调用；要符合条件，该过程必须返回 `trigger` 数据类型，并且该过程的语言处理程序必须能够处理触发器信息。目前大多数语言都支持触发器，包括 PL/C、PL/pgSQL、PL/Perl 和 PL/PHP，但在使用特定语言之前，请务必确认其支持情况。

### 修改触发器

触发器创建后，你可以使用 `ALTER TRIGGER` 命令对其进行修改。目前，你只能修改给定触发器的名称，但这种修改不仅仅是外观上的改变——它还会影响触发器的触发顺序，因为为同一表上同一操作同时定义的触发器会按字母顺序执行。修改触发器的语法如下：

```
ALTER TRIGGER 名称 ON 表名 RENAME TO 新名称;
```

要修改触发器，你必须拥有触发器定义中所指定表的所有权。

### 删除触发器

你也可以使用 `DROP TRIGGER` 命令从表中删除触发器。该命令的语法如下：

```
DROP TRIGGER 名称 ON 表名 [CASCADE | RESTRICT]
```

要删除触发器，你必须拥有触发器定义中所指定表的所有权。`CASCADE` 或 `RESTRICT` 选项控制删除触发器时是否同时删除任何依赖对象，或者在处理完所有依赖对象之前是否拒绝删除。默认行为是 `RESTRICT`。

### 编写触发器函数

当然，在 PostgreSQL 中使用触发器的另一半是这些触发器所执行的函数。触发器函数在大多数方面与其他函数相似；它们以相同的方式定义，并且触发器函数内部的语法操作方式也与其他任何函数相同。但是，你应该注意触发器函数的一些不同之处：

- 触发器函数不能带参数。
- 触发器函数可以访问特殊结构 `NEW` 和 `OLD`，它们分别表示要插入行的新数据（来自插入或更新操作）或行中先前包含的旧数据（来自更新或删除操作）。
- 触发器函数的返回类型必须是 `trigger` 类型。

除了这三个主要区别之外，在处理触发器函数时，你还需要了解一些事项。在函数体内，被按语句调用的触发器函数应始终返回 `NULL`。如果你希望跳过对当前行的操作，可以在查询执行前触发的行级触发器中返回 `NULL`。你还可以为插入和更新的行级 `BEFORE` 触发器函数返回 `NEW`，这将用触发器函数中的数据替换要放入表中的数据。对于 `AFTER` 触发器函数，返回值会被忽略，因此通常建议直接返回 `NULL`。

当你试图记住哪些触发器何时触发时，关于前置和后置以及语句级与行级触发器的讨论有时会令人困惑。表 34-1 显示了给定表上不同触发器的操作顺序。

**表 34-1.** *触发器操作顺序*

| 时机 | 频率     | 顺序 |
|------|----------|------|
| Before | 语句级  | 1    |
| Before | 行级    | 2    |
| After  | 行级    | 3    |
| After  | 语句级  | 4    |



之前

语句级触发器

按触发器名称字母顺序排列

之前

行级触发器

按触发器名称字母顺序排列

**正在执行触发查询**  
（`Insert`、`Update` 或 `Delete`）

之后

行级触发器

按触发器名称字母顺序排列

之后

语句级触发器

按触发器名称字母顺序排列

一旦触发器被触发，PostgreSQL 将按这些步骤执行，执行所有找到的触发器函数。如果某个触发器函数恰好触发了其他触发器（例如通过向另一个表插入数据），PostgreSQL 会先在该表上执行这些步骤，直到处理完毕，再返回原始表继续处理剩余的触发器。

**注意** 你经常会看到这种一个触发器触发另一个触发器的过程被称为*级联触发器*。

### 示例触发器函数

虽然编写触发器函数并不困难，但我们希望向你展示一个解决最常见触发器函数操作问题的示例：持续更新时间戳字段。在此场景中，你将修改第 32 章中使用的 `employee` 表，添加一个用于标记表中信息最后更新时间的字段。然后，你将使用一个触发器来确保只要表中的数据被修改，该信息就会自动更新。首先，添加新列：

```
company=# ALTER TABLE employee ADD COLUMN last_updated timestamptz; ALTER TABLE
```

清单 34-1 展示了触发器函数的语法。

**清单 34-1.** *一个简单的触发器函数*

```
CREATE OR REPLACE FUNCTION last_updated() RETURNS trigger AS
$$
BEGIN
    NEW.last_updated = now();
    RETURN NEW;
END
$$ LANGUAGE 'plpgsql';
```

虽然这个函数很简单，但你可以看到它如何整合了触发器函数所需的各种变更：它不接受参数，并返回 `trigger` 类型。在函数内部，你使用了 `NEW` 结构，该结构包含将要插入到表中的数据，通过设置 `NEW` 结构的 `last_updated` 列的值，并返回修改后的 `NEW` 行。随后，该行将被插入到数据库中，其中包含新设置的 `last_updated` 列。由于 `INSERT` 和 `UPDATE` 都会向表中输入新数据，因此该函数适用于这两种查询类型。

当然，下一步是将触发器添加到表中。由于你想在数据被插入表之前对其进行操作，因此你将创建一个 `BEFORE` 触发器。同时，由于你想确保每一行都被更新，你将选择频率 `FOR EACH row`。语法如下：

```
CREATE TRIGGER last_updated
BEFORE insert OR update
ON employee
FOR EACH row
EXECUTE PROCEDURE last_updated();
```

创建此触发器后，任何后续的插入或更新操作都会更新这个新字段。让我们双重检查以确保一切按预期工作：

```
company=# UPDATE employee SET fname = 'Emilia' WHERE employee_id = 3; UPDATE 1
company=# SELECT * FROM employee WHERE employee_id = 3;
employee_id | fname | lname | last_updated
-------------+--------+-------+-------------------------------
3 | Emilia | Jane | 2005-12-20 12:11:24.86753-09
(1 row)
```

如你所见，`last_updated` 字段已按预期更新。值得注意的是，即使你为 `last_updated` 列指定了显式值，触发器也会触发。在这种情况下，指定的值将在 `NEW` 结构中被替换，并使用系统生成的时间。

**提示** 如果函数编写得足够通用，同一个函数可以被多个触发器使用。在前面的例子中，任何包含名为 `last_updated` 列的表都可以使用这个触发器函数。

在下一个例子中，你将进一步了解触发器函数。这次你将同时使用 `OLD` 和 `NEW` 结构，并且你将更新另一个表，而不是触发器所在的表。首先，让我们在 company 数据库中添加另一个示例表：

```
CREATE TABLE email (
    employee_id INTEGER PRIMARY KEY REFERENCES employee(employee_id),
    address TEXT
);
```

然后继续向表中添加一条记录：

```
INSERT INTO email (employee_id, address)
VALUES (3, 'EmiliaJ@example.com');
```

现在，在继续之前，先看看这些数据：

```
company=# SELECT employee_id, fname, lname, address
company-# FROM employee JOIN email
company-# USING (employee_id) WHERE employee_id = 3 ;
employee_id | fname | lname | address
-------------+--------+-------+---------------------
3 | Emilia | Jane | EmiliaJ@example.com
```

现在你已经有了可以处理的数据，接下来继续设置触发器。在这个例子中，假设公司决定希望员工的电子邮件地址遵循“名字首字母+姓氏首字母”的格式。为了实现这一点，你将设置一个触发器函数，以便在员工姓名发生更改时更新 `email` 表。第一步是创建触发器函数：

```
CREATE OR REPLACE FUNCTION email_address_change() RETURNS TRIGGER AS
$$
DECLARE
    lastinitial TEXT;
    domain TEXT := '@example.com';
    fulladdress TEXT;
BEGIN
    IF TG_OP = 'UPDATE' THEN
        IF NEW.fname <> OLD.fname OR NEW.lname <> OLD.lname THEN
            SELECT substr(NEW.lname,1,1) INTO lastinitial;
            fulladdress := NEW.fname || lastinitial || domain;
            UPDATE email SET address = fulladdress
            WHERE employee_id = NEW.employee_id;
        END IF;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';
```

由于这个函数有点复杂，在继续创建触发器之前，我们先暂停一下。函数的第一行现在应该看起来相当熟悉了；你创建了函数，为其命名并指定了特殊的 `TRIGGER` 返回类型。接下来，你声明了一些变量，用于在函数内部操作数据。

至此，你已经准备好开始实现函数的真正逻辑。在这个例子中，我们使用了一对嵌套的 `IF` 语句。第一个 `IF` 语句验证触发器是通过 `UPDATE` 语句调用的，第二个 `IF` 语句判断员工的名或姓是否发生了变化。虽然这是一个展示 PL/pgSQL 如何允许你嵌套控制结构的好例子，但为了函数能正确运行，这也是必需的。在 PL/pgSQL 中，条件判断不会“短路”，这意味着在确定条件判断的最终结果之前，条件判断的所有部分都会被评估。如果我们将这些嵌套的 `IF` 语句写成一个单一的 `IF`，例如：

```
IF TG_OP = 'UPDATE' AND NEW.fname <> OLD.fname OR NEW.lname <> OLD.lname THEN
```

那么当从 `INSERT` 语句调用时，该函数会返回错误，因为 PL/pgSQL 会尝试确定 `NEW.fname <> OLD.fname` 以及 `TG_OP = 'UPDATE'`，但对于 `INSERT` 语句来说，并没有 `OLD` 结构可用。这种行为与许多编程语言略有不同，因此需要小心，即使是经验丰富的程序员有时也会因此出错。

另一个值得仔细研究的项目是 `TG_OP` 变量。`TG_OP` 表示一个文本字符串，告诉你函数是被 `INSERT`、`UPDATE` 还是 `DELETE` 触发的。实际上，PostgreSQL 在触发器函数中提供了相当多的这类特殊变量。表 34-2 提供了这些特殊变量的更详细列表。

**表 34-2.** *触发器函数的特殊变量*

| 变量名 | 定义 |
|--------|------|
| `NEW` | 在行级触发器中包含 `INSERT` 和 `UPDATE` 操作的新数据库行；在语句级触发器中为 `NULL`。数据类型为 `RECORD`。 |
| `OLD` | |



`TG_OLD` 包含行级触发器中 `UPDATE` 和 `DELETE` 操作的旧数据库行；在语句级触发器中为 `NULL`。数据类型为 `RECORD`。

`TG_LEVEL` 包含基于触发器定义的字符串 `ROW` 或 `STATEMENT`。数据类型为 `TEXT`。

`TG_NAME` 包含实际触发的触发器名称。数据类型为 `NAME`。

`TG_NARGS` 包含 `CREATE TRIGGER` 语句中传递给触发器过程的参数数量。数据类型为 `INTEGER`。

`TG_OP` 包含基于触发触发器的操作对应的字符串 `INSERT`、`UPDATE` 或 `DELETE`。数据类型为 `TEXT`。

`TG_RELID` 包含导致触发器被触发的表的对象 ID。数据类型为 `OID`。

`TG_RELNAME` 包含导致触发器被触发的表的名称。数据类型为 `NAME`。

`TG_WHEN` 包含基于触发器定义的字符串 `BEFORE` 或 `AFTER`。数据类型为 `TEXT`。

`TG_ARGV[]` 包含来自 `CREATE TRIGGER` 语句的参数，从 0 开始。数据类型为 `TEXT`。

---

回到我们的函数，一旦确定满足适当条件，就开始构建新的电子邮件地址。首先确定姓氏的首字母，然后（使用 SQL 的 `||` 运算符）将名字、姓氏首字母和域名拼接在一起。获得新的电子邮件地址后，更新 `address` 表中的相应记录，然后发出 `RETURN` 语句。

现在函数已经完成，我们来创建触发器：
```
CREATE TRIGGER maintain_email
AFTER update OR insert
ON employee
FOR EACH row
EXECUTE PROCEDURE email_address_change();
```
如您所见，此触发器定义看起来与第一个示例非常相似，唯一显著的区别在于这里选择将其设为 `AFTER` 触发器。这样做的原因是无需修改即将插入到 `employee` 表中的任何数据，因此无需事先触发触发器。这些部分就位后，让我们看看实际变更效果：
```
company=# UPDATE employee SET fname='Emma' WHERE employee_id = 3;
UPDATE 1

company=# SELECT employee_id, fname, lname, address
company-# FROM employee JOIN email
company-# USING (employee_id) WHERE employee_id = 3 ;

employee_id | fname | lname | address
-------------+--------+-------+---------------------
3           | Emma  | Jane  | EmmaJ@example.com
```
如您所见，触发器在 `employee` 表更新后成功更新了 `address` 表，完全符合预期。使用这些技术，您应该能够开始看到如何直接在数据库层面强制执行最复杂的业务规则集，并且以这种方式强制执行有助于保持数据规范，无论是从命令行、通过 Web 应用程序，甚至是通过 shell 脚本进行更新。

### 查看现有触发器

有时在处理数据库时，需要查看是否有任何触发器参与修改给定表。大多数管理工具不会查询数据库中的完整触发器列表，而是根据触发器依赖的表对它们进行分组。例如，在 `psql` 中，当描述一个表时，您会看到以下输出：
```
company=# \d employee

Table "public.employee"
    Column    |           Type           | Modifiers
--------------+--------------------------+-----------
employee_id   | integer                  | not null
fname         | text                     |
lname         | text                     |
last_updated  | timestamp with time zone |
Indexes:
    "employee_pkey" PRIMARY KEY, btree (employee_id)
Triggers:
    last_updated BEFORE INSERT OR UPDATE ON employee FOR EACH
        ROW EXECUTE PROCEDURE last_updated()
    maintain_email AFTER UPDATE OR INSERT ON employee FOR EACH
        ROW EXECUTE PROCEDURE email_address_change()
```
在此输出中，显示了完整的触发器定义，包括触发器名称及其调用的函数。

### 规则 vs. 触发器

一个常见问题涉及到何时使用触发器而非规则。在许多情况下，两者的使用可能可以互换。然而，有时一种方法优于另一种方法。您可能想坚持使用规则的一个原因是，如果您不想涉及编写数据库函数（实现触发器需要编写函数）。规则胜出的另一个场景是处理针对视图的数据修改。在这种情况下，您唯一的选择是使用规则，因为视图中没有数据供触发器操作。

当然，触发器也有其优势。一方面，触发器概念在数据库系统中更为常见，因此触发器可能更容易被大多数人理解。触发器还可以利用 PostgreSQL 提供的各种函数语言中的过程能力或其他高级功能，比规则强大得多。

### 总结

在本章中，我们研究了在 PostgreSQL 中使用触发器所涉及的基本概念。我们介绍了有助于定义触发器的六个不同特征，并探讨了这些特征如何影响触发器操作。我们还讨论了编写触发器函数与常规函数之间的区别，并展示了一个示例函数以及一个使用该函数的触发器。虽然触发器不直接交互，但它们的功能和实用性可以大大增强您的应用程序，并使您的应用程序代码更简单、更有效。

---

