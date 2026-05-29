# 第 32 章 ■ 视图与规则

**709**

## 创建规则

你可以使用`CREATE RULE`命令创建规则，其完整语法如下：

```
CREATE [ OR REPLACE ] RULE rule_name AS ON event_type
TO object_name [ WHERE conditional ] DO [ ALSO | INSTEAD ] command
```

以下列表描述了该语法的各个部分：

- `CREATE [ OR REPLACE ] RULE rule_name`：指定规则的名称，并可选用`OR REPLACE`子句，该子句告知 PostgreSQL 替换同表或同视图上已存在的同名规则。

- `AS ON event_type`：确定规则将针对何种事件类型执行。`event_type`可以是`SELECT`、`INSERT`、`UPDATE`或`DELETE`之一，每种类型将在后续的“规则类型”一节中详细描述。

- `TO object_name [ WHERE conditional ]`：指定规则所应用的表或视图的名称。如果你希望规则仅在特定情况下执行，可以指定可选的`conditional`条件。可以使用任何 SQL 条件表达式，只要它返回布尔值、不包含聚合函数，并且除可选的`NEW`和`OLD`伪关系（在适用情况下）外，不引用其他表或视图。

- `DO [ ALSO | INSTEAD ]`：描述规则是应在针对表的原始 SQL 语句操作之外执行，还是应替代原始 SQL 语句执行。如果既未指定`ALSO`也未指定`INSTEAD`，则默认使用`ALSO`。

- `command`：指定要为规则运行的所需查询。命令的形式为`SELECT`、`INSERT`、`UPDATE`、`DELETE`或`NOTIFY`语句。规则中`COMMAND`部分使用的 SQL 查询可以访问`NEW`和`OLD`伪关系（在适用情况下）。如果您不希望对规则执行任何操作，`COMMAND`部分也可以使用`NOTHING`关键字。

**提示** `COMMAND`部分中指定的操作不必与`event_type`中指定的操作相匹配。例如，你可以创建一个规则，每当有人试图更新视图时，该规则就向另一个表插入数据。

### 删除规则

要删除规则，可以使用`DROP RULE`命令。与`CREATE RULE`命令相比，其语法要简单得多：

```
DROP RULE rule_name ON object_name [ CASCADE | RESTRICT ]
```

`DROP RULE`命令的关键元素包括：规则的名称、规则所应用到的视图或表的名称，以及可选的`CASCADE`或`RESTRICT`关键字。如果选择`CASCADE`，则所有依赖于该规则的对象都将被删除；如果指定了`RESTRICT`，则如果该规则有任何依赖对象，PostgreSQL 将拒绝删除该规则；默认值为`RESTRICT`。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**710**

## 规则类型

如前所述，PostgreSQL 有四种基本规则类型：选择、插入、更新和删除。每种规则都有一些独特的特征，尽管它们都遵循相同的基本模式。

### 选择规则

最基本的规则类型是选择规则。选择规则被定义为`ON SELECT`，其操作必须是一个无条件的`SELECT`操作，并标记为替代原始查询运行。通过这种方式，表上的规则模拟了视图的功能，以至于以下两个示例在功能上是等效的：

```
CREATE VIEW ourbooks AS SELECT * FROM books;
```

和

```
CREATE TABLE ourbooks AS SELECT * FROM books;
CREATE RULE "_RETURN" AS ON SELECT TO ourbooks DO INSTEAD SELECT * FROM books;
```

对于`ON SELECT`规则，PostgreSQL 要求使用名称`_RETURN`，以帮助向内部查询重写器发出信号，表明被查询的关系是一个视图。PostgreSQL 中的视图会自动使用选择规则来处理选择调用并从其基表中检索数据，但在大多数情况下，你不需要直接处理选择规则。

### 插入规则

PostgreSQL 中使用的下一种规则类型是插入规则。插入规则可以具有`ALSO`或`INSTEAD`的动作，并且可以根据需要包含多个动作或没有动作。插入规则中定义的动作可以包含条件，并且还可以使用`NEW`伪关系。插入规则的语法如下所示：

```
CREATE RULE database_book_insert AS ON INSERT TO database_books DO INSTEAD INSERT
INTO books (title, copies_on_hand, genre) VALUES (NEW.title, 1, 'technical');
```

有了这条规则，任何针对`database_books`视图的插入操作，都会改为将指定的值插入到原始的`books`表中。

### 更新规则

下一种规则类型是更新规则。与插入规则类似，更新规则可以具有`ALSO`或`INSTEAD`的动作，并且可以根据需要包含多个动作或没有动作。更新规则中定义的动作可以包含条件，并且可以使用`NEW`和`OLD`伪关系。一个更新规则的示例如下所示：

```
CREATE RULE database_books_update AS ON UPDATE TO database_books WHERE
NEW.title <> OLD.title DO INSTEAD UPDATE books SET title = NEW.title;
```

有了这条规则，针对我们的`database_books`视图的更新操作（其中标题已更改），会改为使用新标题更新原始的`books`表。

### 删除规则

最后一种规则类型是删除规则。它可以具有`ALSO`或`INSTEAD`的动作，并且可以根据需要包含多个动作或没有动作。删除规则中定义的动作可以包含条件，并且可以使用`OLD`伪关系。一个删除规则的示例如下所示：

```
CREATE RULE database_books_delete AS ON DELETE TO database_books
DO INSTEAD NOTHING;
```

在这里，我们使用`NOTHING`关键字来防止当有人尝试从`database_books`视图中删除时发生任何删除操作。这会产生两种效果：我们阻止了对`database_books`视图有访问权限的人从我们的主`books`表中删除数据，同时我们允许对`database_books`表执行`DELETE`语句而不会报错。不要轻易忽略第二种效果。在大多数正常情况下，从视图中删除（以及插入和更新）会引发错误，但通过使用规则，如果我们的应用程序需要，我们可以处理这些错误。

## 让视图可交互

如今，许多数据库都提供某种形式的可更新视图，允许你对视图运行`UPDATE`语句，并让它们更新底层表。然而，你经常会发现，这种视图的定义受到严格限制，例如不允许连接查询或不允许在视图定义中对条目进行特殊格式化。

PostgreSQL 通过其规则系统，允许你绕过这些限制，从而在创建可更新视图的类型方面提供更大的灵活性。下一节将引导你完成几个使用 PostgreSQL 规则系统的示例。

### 可更新、可插入、可删除的视图

我们示例的第一部分创建了一组我们将使用的简单表：

```
CREATE TABLE employee (
    employee_id INTEGER PRIMARY KEY,
    fname TEXT NOT NULL,
    lname TEXT NOT NULL
);

CREATE TABLE phone (
    employee_id INTEGER REFERENCES employee (employee_id) ON DELETE CASCADE,
    npa INTEGER NOT NULL,
    nxx INTEGER NOT NULL,
    xxxx INTEGER NOT NULL
);
```

这建立了两个表，一个用于存储员工姓名，另一个用于存储他们的电话信息。当然，虽然使用像`npa`、`nxx`和`xxxx`这样的列名可能符合北美编号计划的官方格式，但它们使用起来有点笨拙，所以让我们继续创建我们的视图：

```
CREATE VIEW directory AS (SELECT employee.employee_id,
       fname || ' ' || lname AS name,
       npa || '-' || nxx || '-' || xxxx AS number
FROM employee JOIN phone USING (employee_id));
```

这将创建一个三列视图：一列显示员工 ID，一列显示员工全名，一列显示经过格式化的员工电话号码。有了这个结构之后，我们继续添加一些数据：

```sql
INSERT INTO employee(employee_id,fname,lname) VALUES
(1,'Amber','Lee');

INSERT INTO phone(employee_id, npa, nxx, xxxx) VALUES
(1,607,555,5210);
```

并通过我们的视图查看结果：

```
rob=# SELECT * FROM directory;
 employee_id |   name    |   number
-------------+-----------+--------------
           1 | Amber Lee | 607-555-5210
(1 row)
```

这看起来很不错，但假设我们要向通讯录中添加一个新成员。为此，我们必须了解底层表的结构，并向这两个表中插入数据：

```sql
INSERT INTO employee(employee_id, fname, lname)
VALUES (2,'Dylan','Jairus');

INSERT INTO phone(employee_id, npa, nxx, xxxx)
VALUES (2,813,555,5040);
```

正如你所见，要向系统中添加信息，我们必须向两个表中插入数据。如果我们能找到一种方法，直接以我们习惯的格式向通讯录视图输入数据，那该多好。这时，我们的第一个规则就派上用场了。我们将创建一个规则，用于处理对通讯录视图的直接插入操作，将数据拆分到合适的表中，并转换为基表所需的合适类型：

```sql
CREATE RULE directory_addition AS ON INSERT TO directory DO INSTEAD
(
INSERT INTO employee VALUES
(NEW.employee_id,
 split_part(NEW.name,' ', 1),
 split_part(NEW.name,' ', 2) );

INSERT INTO phone VALUES
(NEW.employee_id,
 split_part(NEW.number,'-', 1)::INTEGER,
 split_part(NEW.number,'-', 2)::INTEGER,
 split_part(NEW.number,'-', 3)::INTEGER );
) ;
```

由于通讯录视图将基表各列的数据合并成了单个列，我们必须将这些数据拆分后才能插回到底层表中；在本例中，我们使用了内置的数据库函数`split_part`（自 PostgreSQL 7.3 起新增），该函数根据给定的分隔符拆分文本字符串。这一点非常重要，需要牢记：只要你能找到分解视图中数据组合规则的方法，就可以利用规则系统将数据推回到底层表中。有了这个规则，我们现在可以直接向视图中插入数据了：

```
rob=# INSERT INTO directory VALUES (3,'Emma Jane','352-555-6120');
INSERT 107999 1
```

`INSERT` 提示表明我们的 `INSERT` 语句执行成功，但我们知道数据并没有直接存入通讯录视图。让我们查看一下基表的情况：

```
rob=# SELECT * FROM employee;
 employee_id | fname | lname
-------------+-------+--------
           1 | Amber | Lee
           2 | Dylan | Jairus
           3 | Emma  | Jane
(3 rows)
```

`employee` 表里有我们的新员工，这很好，但他们的联系方式呢？

```
rob=# SELECT * FROM phone;
 employee_id | npa | nxx | xxxx
-------------+-----+-----+------
           1 | 607 | 555 | 5210
           2 | 813 | 555 | 5040
           3 | 352 | 555 | 6120
(3 rows)
```

成功了！我们已经成功地向视图中插入了数据，并且该规则按照需要将数据拆分并插入了相应的表中。当然，既然你能向视图中插入数据，你肯定也希望从中删除数据，我们同样可以使用规则来实现这一点：

```sql
CREATE RULE youre_fired AS ON DELETE TO directory DO INSTEAD
DELETE FROM employee WHERE fname=split_part(OLD.name,' ', 1) AND
                           lname=split_part(OLD.name,' ', 2);
```

如果你不想的话，连接表上的规则不必引用视图中的所有表。请注意，在这个规则中，我们只引用了 `employee` 表，这对我们的示例来说运行良好，因为 `phone` 表中的相关条目会由于我们在原始表中定义的级联引用而被自动删除。让我们解雇一名员工：

```
rob=# DELETE FROM directory WHERE name = 'Amber Lee';
DELETE 1
```

同样，数据库指示我们的删除操作已成功，接下来我们查看基本表以验证数据：

```
rob=# SELECT * FROM employee;
```

```
employee_id | fname | lname
-------------+-------+--------
2 | Dylan  | Jairus
3 | Emma   | Jane
(2 行)
```

我们想要删除的员工已不在 `employee` 表中，接下来检查一下他们的联系方式：

```
rob=# SELECT * FROM phone;
```

```
employee_id | npa | nxx | xxxx
-------------+-----+-----+------
2 | 813 | 555 | 5040
3 | 352 | 555 | 6120
(2 行)
```

再次看到，规则系统能够正确地将我们的请求传递到相应的表，并且该员工及其电话信息的条目已被删除。

既然我们现在可以通过视图进行插入和删除操作，那么自然也应该能够更新已有数据。在本例中，我们将创建一个规则，允许修改电话号码信息，但不能修改姓名信息：

```sql
CREATE RULE modify_employee AS
ON UPDATE TO directory DO INSTEAD
UPDATE phone SET
npa=split_part(NEW.number,'-', 1)::INTEGER,
nxx=split_part(NEW.number,'-', 2)::INTEGER,
xxxx=split_part(NEW.number,'-', 3)::INTEGER
WHERE employee_id = NEW.employee_id;
```

这个规则结合了我们之前讨论过的多个要点：它仅与连接中的一个基本表交互；它逆转了视图定义中使用的复杂公式；并且它在运行时转换数据类型，以正确匹配基本表中定义的类型。

现在我们来更新 `directory`：

```
rob=# UPDATE directory SET number='352-555-7120'
WHERE employee_id = 3;
```

```
UPDATE 1
```

像往常一样，我们从 PostgreSQL 收到关于语句执行成功的返回码，但让我们再次检查基本表，看看具体发生了什么：

```
rob=# SELECT * FROM phone;
```

```
employee_id | npa | nxx | xxxx
-------------+-----+-----+------
2 | 813 | 555 | 5040
3 | 352 | 555 | 7120
(2 行)
```

如你所见，新的号码现在存储在 `phone` 表中。由于我们不需要更新 `employee` 表，让我们再次查看视图，看看更改是否已反映出来：

```
rob=# SELECT * FROM directory;
```

```
employee_id | name          | number
-------------+--------------+--------------
2           | Dylan Jairus | 813-555-5040
3           | Emma Jane    | 352-555-7120
(2 行)
```

当然，规则系统的用途并不仅限于制作交互式视图：你也可以在表上使用它，而且能做的事情也不仅限于直接引用表。在下一个例子中，我们创建一个名为 `salary` 的新表，并向其中插入一些信息：

```
CREATE TABLE salary (employee_id INTEGER REFERENCES
employee(employee_id) ON DELETE CASCADE, salary INTEGER);
INSERT INTO salary VALUES (2,400000);
INSERT INTO salary VALUES (3,200000);
```

我们决定做的第一件事是防止任何人删除员工的薪资。通常可以通过使用`REVOKE DELETE`命令来实现，但`REVOKE DELETE`并不能阻止超级用户意外删除数据，因此我们想要更进一步：

```
CREATE RULE always_pay AS ON DELETE TO salary DO INSTEAD NOTHING;
```

这是规则的`INSTEAD`形式，它会导致查询被重写，从而根本不执行。现在，即使超级用户试图从表中删除数据，删除操作也会被阻止：

```
rob=# DELETE FROM salary WHERE employee_id = 2;
```

```
DELETE 0
```

另一件可能需要的事情是记录员工薪资可能发生的任何变更。我们通过结合一个日志表和一个更新规则来实现：

```
CREATE TABLE salary_log (employee_id INTEGER REFERENCES
employee(employee_id) ON DELETE CASCADE, salary_change INTEGER,
changed_by TEXT, log_time TIMESTAMP DEFAULT now());
CREATE RULE log_salary_changes AS ON UPDATE TO salary DO ALSO INSERT
INTO salary_log VALUES (NEW.employee_id, NEW.salary - OLD.salary, CURRENT_USER);
```

请注意，此规则属于“同时也执行”类型，意味着原始查询将与规则中指定的操作一同执行。它使用原始查询中的数据、一个数学运算以及内部函数`CURRENT_USER`（用于获取当前登录数据库的用户）：

```
rob=# UPDATE salary SET salary = 250000 WHERE employee_id = 3;

UPDATE 1

rob=# SELECT * FROM salary_log;

employee_id | salary_change | changed_by | log_time
-------------+---------------+------------+--------------------------
          3 |         50000 | rob        |2005-07-11 15:19:33.703885
(1 row)
```

我们尚未提及的一点是，规则会针对初始查询所涉及的所有行触发，而不仅仅是一行。这一点有时会让人措手不及，但也能带来极大便利。例如，如果我们需要将公司所有人的薪资削减 15%，我们可以无需额外工作就追踪这些变更：

```
rob=# UPDATE salary SET salary = (salary – salary*.15) ;

UPDATE 2

rob=# select * from salary_log;

employee_id | salary_change | changed_by | log_time
-------------+---------------+------------+---------------------------
          3 |         50000 | rob        | 2005-07-11 15:19:33.703885
          2 |        -60000 | rob        | 2005-07-11 15:30:41.008909
          3 |        -37500 | rob        | 2005-07-11 15:30:41.008909
(3 rows)
```

由于我们更新了表中的两行，因此日志表中插入了两条额外记录，这正是我们期望的结果。

### 在 PHP 中操作视图

尽管视图在数据库层面与表有一些不同的特性，但在 PHP 中，从视图查询数据与从表查询数据并无区别。清单 32-1 展示了一个简单的 PHP 页面，它从我们的`directory`视图查询数据并在屏幕上显示结果。你会注意到，它与第 31 章中查询表的示例非常相似。

**清单 32-1.** *使用 PHP 查询视图*

```php
<?php
include "pgsql.class.php";

// 创建新的 pgsql 对象
$pgsqldb = new pgsql("localhost","rob","rob","secret");

// 连接到数据库服务器并选择数据库
$pgsqldb->connect();

// 查询数据库
$pgsqldb->query("SELECT name, number FROM directory
                 ORDER BY name");

// 输出数据
while ($row = $pgsqldb->fetchObject())
  echo "$row->name, $row->number<br />";
?>
```

在浏览器中执行此代码时，将返回以下结果：`Dylan Jairus, 813-555-5040` `Emma Jane, 352-555-7120`

你可以看到，无论对于 PHP 应用开发者还是最终用户，查询视图都与查询表毫无差别。因此，作为经验法则，在构建应用程序时，你应该将视图和表视为无区别对待。如果你使用 PostgreSQL 规则来使视图完全可交互，这一点尤为重要。

### 总结

在本章中，我们深入探讨了如何利用 PostgreSQL 的一些高级功能在数据库内部创建强大且交互的关系。我们首先了解了视图，包括它们在数据库中的工作原理以及添加和删除视图的命令。接着，我们通过探讨不同类型的规则并概述如何定义这些规则，详细解释了 PostgreSQL 的规则系统。最后，我们给出了一些示例，说明如何将视图、规则和表结合使用，以创建高度交互的数据库模式。这些示例虽然简单，但应能为你提供一些关于如何利用这些强大功能的好思路。