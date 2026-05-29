# PostgreSQL 函数

尽管 PostgreSQL 长期以来以其对自定义函数语言的支持而闻名，但许多人并未充分利用其丰富的内置函数和大量的内置运算符。在本章中，我们将同时介绍运算符和函数，并花些时间探讨自定义函数。到本章结束时，你将熟悉以下内容：

•   什么是运算符以及 PostgreSQL 中最常用的运算符

•   不同类型的 PostgreSQL 内部函数以及每种类型最常见的示例

•   PostgreSQL 的内部过程语言，包括如何用这些语言编写你自己的函数

•   如何通过额外的自定义过程语言扩展 PostgreSQL

你应该明白，我们的目标并非提供 PostgreSQL 中每个运算符和函数的全面参考——坦白说，这些内容实在太多了，而且许多仅在非常狭窄的领域中使用。相反，我们将聚焦于其中最常用和最有用的部分，这样你在开始开发基于 PostgreSQL 的应用程序时，就能打下坚实的基础。

## 运算符

PostgreSQL 提供了大量的内置运算符（截至最近一次统计，超过 600 个！），用于执行各种比较和数据转换。在本节中，我们将研究最常用的运算符，其中许多可能你已经很熟悉了。

### 逻辑运算符

PostgreSQL 中的逻辑运算符与大多数编程语言类似，具体如下：

- `AND`

- `OR`

- `NOT`

关于 `AND` 和 `OR` 逻辑运算符，一个常被误解的方面是它们如何与 `NULL` 值交互。SQL 使用三态布尔逻辑，其中 `NULL` 值代表一个“未知”的值。表格 33-1 详细说明了各种逻辑运算符的效果。

**表格 33-1.** *逻辑运算符*

| `foo` | `bar` | `foo AND bar` | `foo OR bar` |
|-------|-------|---------------|--------------|
| TRUE | TRUE | TRUE | TRUE |
| TRUE | FALSE | FALSE | TRUE |
| TRUE | NULL | NULL | TRUE |
| NULL | NULL | NULL | NULL |
| FALSE | NULL | FALSE | NULL |
| FALSE | FALSE | FALSE | FALSE |

### 比较运算符

比较运算符，如表格 33-2 所示，用于比较两个值并返回一个布尔值。

**表格 33-2.** *常用比较运算符*

| 运算符 | 示例 | 说明 |
|----------|---------|-------------|
| `<` | `12 < 21` | 小于 |
| `<=` | `12 <= 21` | 小于或等于 |
| `>` | `21 > 12` | 大于 |
| `>=` | `21 >= 12` | 大于或等于 |
| `<>`, `!=` | `21 <> 12` | 不等于 |
| `=` | `21 = 12 + 9` | 等于 |
| `BETWEEN` | `12 BETWEEN 9 and 21` | 构造用于 `21 >= 12 AND 12 <= 9` |
| `NOT BETWEEN` | `21 NOT BETWEEN 12 and 9` | 构造用于 `21 < 12 OR 21 > 9` |

PostgreSQL 还提供了一些额外的构造用于比较布尔值或 `NULL` 值，如表格 33-3 所示。这些值可以是一个特定列，也可以是一个表达式的结果。如果表达式符合所测试的构造，则返回 true。

**表格 33-3.** *常用比较运算符构造*

| 比较运算符构造 |
|--------------------------------|
| `expression IS NULL` |
| `expression IS NOT NULL` |
| `expression IS TRUE` |
| `expression IS NOT TRUE` |
| `expression IS FALSE` |
| `expression IS NOT FALSE` |

### 数学运算符

PostgreSQL 中的数学运算符与大多数编程语言中的相同。最常用的数学运算符列在表格 33-4 中。

**表格 33-4.** *常用数学运算符*

| 运算符 | 示例 | 说明 |
|----------|---------|-------------|
| `+` | `9 + 6 = 15` | 加法 |
| `-` | `9 – 6 = 3` | 减法 |
| `*` | `9 * 6 = 54` | 乘法 |
| `/` | `9 / 6 = 1` | 除法（返回 1，因为是整数除法） |
| `%` | `9 % 6 = 3` | 取模（余数） |

### 字符串运算符

字符串运算符通常在 PostgreSQL 中用于字符串操作或字符串匹配。最常用的列在表格 33-5 中。

**表格 33-5.** *常用字符串运算符*

| 运算符 | 示例 | 说明 |
|----------|---------|-------------|
| `||` | `'foo' \|\| 'bar'` 结果为 `'foobar'` | 字符串连接 |
| `~` | `'foobar' ~ 'oo.*r'` | 正则表达式匹配。`^` 和 `$` 分别将搜索锚定到字符串的开头和结尾 |
| `~*` | `'foobar' ~* 'OO.*R'` | 不区分大小写的正则表达式 |
| `!~` | `'foobar' !~ 'rr.*o'` | 不匹配正则表达式 |
| `!~*` | `'foobar' !~* 'RR.*O'` | 不匹配不区分大小写的正则表达式 |
| `~~` | `'foobar' ~~ '%oo%'` | `LIKE` 的同义词 |
| `!~~` | `'foobar' !~~ '%RR%'` | `NOT LIKE` 的同义词 |

### 运算符优先级

在 PostgreSQL 中使用运算符与在任何编程语言中使用运算符非常相似。所有运算符都具有优先级，该优先级决定了运算符的处理顺序，并且可以使用括号对某些操作进行分组来更改该优先级。为了说明我们的意思，请看以下两个查询：

```
phppg=# SELECT 3*2+1;
?column?
(1 row)

phppg=# SELECT 3*(2+1);
?column?
(1 row)
```

如果你曾使用过像 PHP 这样的语言进行编程，那么这些结果应该不会让你感到意外，但在 SQL 层面操作时，有关于运算符优先级的几点特殊情况你需要了解。

其中一种特殊情况是，PostgreSQL 运算符还具有与运算符左侧或右侧值的结合性，这决定了具有相同优先级的运算符的处理顺序。例如，加法和减法的算术运算符是左结合的，因此像 `3 – 2 + 1` 这样的表达式会被计算为 `(3 – 2) + 1`。

然而，相等运算符是右结合的，因此 `a = b = c` 会被计算为 `a = (b = c)`。

**表 33-6** 按降序列出了运算符优先级，并附带了运算符的结合性。不过，请始终牢记，如果你需要处理复杂的运算符组合，使用括号可以帮助更改和明确运算符的优先级。

**表 33-6.** *PostgreSQL 中的运算符优先级*

| **运算符** | **结合性** | **说明** |
| --- | --- | --- |
| `.` | 左 | 模式/表/列名称分隔符 |
| `::` | 左 | PostgreSQL 特定的类型转换 |
| `[]` | 左 | 数组选择 |
| `-` | 右 | 整数取负（一元减号） |
| `^` | 左 | 指数运算 |
| `*` / `%` | 左 | 乘法、除法、取模 |
| `+` `-` | 左 | 加法、减法 |
| `IS` | | 测试是否为 `TRUE`、`FALSE`、`UNKNOWN` 或 `NULL` |
| `ISNULL` | | 测试是否为 `NULL` |
| `NOTNULL` | | 测试是否不为 `NULL` |
| (所有其他) | 左 | 所有其他内置和用户定义的运算符 |
| `IN` | | 测试集合成员资格 |
| `BETWEEN` | | 测试是否在某个范围内 |
| `OVERLAPS` | | 测试时间段是否存在重叠 |
| `LIKE` `ILIKE` `SIMILAR` | | 测试字符串模式匹配 |
| `>` `<` | | 大于、小于 |
| `=` | 右 | 测试相等性 |
| `NOT` | 右 | 逻辑非 |
| `AND` | 左 | 逻辑与 |
| `OR` | 左 | 逻辑或 |

## 内部函数

与运算符的数量一样，PostgreSQL 内置函数的数量也相当惊人。在本节中，我们将介绍一些你最有可能遇到的最常用且最有用的内置函数。我们可以将这些函数分为以下几类：

-   用于处理日期和时间值的函数

-   用于处理字符串值的函数

-   用于格式化字符串和时间输出的函数
-   聚合函数
-   各种条件表达式
-   子查询表达式

大多数 PostgreSQL 内置函数可以根据需要，在查询的 `SELECT` 部分或 `WHERE` 部分中调用。

### 日期和时间函数

PostgreSQL 提供了许多与日期和时间相关的函数。对于可以接受时间或时间戳参数的函数，带或不带时区的时间和时间戳都是可以接受的。

**表 33-7** 展示了一些常见的日期和时间函数。

**表 33-7.** *常见日期与时间相关函数*

| **函数** | **说明** |
| --- | --- |
| `current_date` | 返回今天的日期。 |
| `current_time` | 返回当前时间（不返回日期信息）。 |
| `current_timestamp` | 返回当前时间的时间戳（日期和时间）。 |
| `date_part(text, timestamp)` | 返回给定时间戳中文本指定的字段。 |
| `now()` | 返回当前时间的时间戳，该时间戳在事务开始时固定。 |
| `timeofday()` | 返回当前时间的文本输出。该值在事务期间会递增。 |

### 字符串函数

PostgreSQL 中的字符串函数可以用于以多种方式操作字符串值。

对于这些函数，任何字符串输入，包括`text`、`varchar`和`char`类型的字符串，都将被视为函数的有效输入。表 33-8 列出了一些常见的字符串函数。

**表 33-8.** *常见字符串函数*

| 函数 | 说明 |
| --- | --- |
| `lower(string)` | 返回字符串的小写形式。 |
| `position(substring in string)` | 返回子串在字符串中的整数位置。 |
| `split_part(string,delimiter,field)` | 使用分隔符拆分字符串，并返回指定的字段。 |
| `substring(string, from,[for])` | 从字符串中提取一个子串，从`from`位置开始，长度为指定的`for`位数。 |
| `replace(string,from,to)` | 在给定字符串中，将`from`文本替换为`to`文本。 |
| `upper(string)` | 返回字符串的大写形式。 |

### 聚合函数

与大多数其他数据库系统一样，PostgreSQL 提供了许多函数，允许你进行计数、平均计算以及其他聚合操作。表 33-9 列出了一些更常见的聚合函数。

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 33 章 • PostgreSQL 函数

**725**

**表 33-9.** *常见聚合函数*

| 函数 | 说明 |
| --- | --- |
| `avg(expression)` | 所有输入值的平均值。输入值必须为整数类型之一。 |
| `count(*)` | 输入值的数量。 |
| `count(expression)` | 非空输入值的数量。 |
| `max(expression)` | 所有输入值中`expression`的最大值。 |
| `min(expression)` | 所有输入值中`expression`的最小值。 |
| `sum(expression)` | 所有非空输入值的`expression`之和。 |

在 PostgreSQL 中使用聚合函数时，请注意，PostgreSQL 通常需要对表进行全表扫描来满足聚合函数，尤其是在查询中未指定`WHERE`子句的情况下，而不是使用索引扫描。其主要原因是 PostgreSQL 将元组可见性信息存储在表内，而非索引中，因此它必须查看表以确定哪些元组与当前事务相关。（想象一下，尝试为两个并发查询提供准确的`count(*)`，其中一个刚执行了大量删除操作，而另一个则执行了大量插入操作。）如果不小心，这可能导致性能不佳。在 PostgreSQL 8.1 版本中，`min()`和`max()`函数已解决了此问题，PostgreSQL 现在会尝试利用可用的索引来确定此信息；希望未来我们能看到其他聚合函数得到改进。

### 条件表达式

条件表达式更像是结构而非函数，但其工作方式与函数非常相似，并且在处理复杂 SQL 时非常有用。PostgreSQL 中有三个主要的条件表达式：`CASE`、`COALESCE`和`NULLIF`。

#### `CASE`

`CASE`函数根据多个匹配条件中的一个，返回多个指定值之一。`CASE`表达式的语法如下：

```
CASE
    WHEN condition THEN result
    [WHEN condition THEN result]
    [ELSE result]
END
```

一个`CASE`表达式可以有任意多个`WHEN`条件，但它只返回第一个评估为真的条件对应的`result`。如果没有`WHEN`条件评估为真，则如果指定了`ELSE result`则返回该结果；否则返回`NULL`。考虑以下示例：

[www.it-ebooks.info](http://www.it-ebooks.info/)

**726**

第 33 章 • PostgreSQL 函数

```
company=# SELECT name, price,
CASE WHEN price < 10.00 THEN 'Hot Deal'
     ELSE 'Exceptional Value' END as offer FROM product;
      name       | price |      offer
-----------------+-------+-----------------
 Linux Hat       |  8.99 | Hot Deal
 PostgreSQL Hat  |  3.99 | Hot Deal
 PHP Hat         | 16.99 | Exceptional Value
(3 rows)
```

#### `COALESCE`

`COALESCE`函数是`CASE`语句的一种特殊格式，它返回其输入中第一个非空值。`COALESCE`表达式的语法如下：

```
COALESCE(VALUE [,VALUE])
```

与 `CASE` 类似，`COALESCE` 可以按需接收任意多个值。如果在列表中未找到任何非空值，`COALESCE` 将返回 `NULL`。为简单起见，我们在这里使用一个较为直接的例子：

```
company=# SELECT name, COALESCE(NULL,price) as price from product; name | price
----------------+-------
Linux Hat       | 8.99
PostgreSQL Hat  | 3.99
PHP Hat         | 16.99
(3 rows)
```

#### `NULLIF`

`NULLIF` 函数有时被认为是 `COALESCE` 的逆操作。`NULLIF` 接受两个参数，如果两个参数相等，则返回 `NULL`；如果不相等，则返回第一个参数。当其中一个或两个参数为 `NULL` 时，结果将被判定为参数不相等的情况。`NULLIF` 的语法如下：

`NULLIF(VALUE1, VALUE2)`

我们可以这样查看 `NULLIF` 的实际作用：

```
company=# select NULLIF(1,2) as different, NULLIF(1,1) as same;
different | same
-----------+--------
        1 |
(1 row)
```

### 更多函数

正如本章开头所述，这里介绍的内置函数和运算符并非完整列表，而只是对您可能遇到的最常见项的简要介绍。PostgreSQL 还提供了更专门的函数，例如几何函数和网络地址函数。您会发现 PostgreSQL 还提供了与内置运算符完全等效的函数，以及用于在各种数据类型之间转换数据的函数。要更深入地了解内置函数，您应该查阅 [`www.postgresql.org/docs/`](http://www.postgresql.org/docs/) 上的在线文档。

### 用户自定义函数

尽管 PostgreSQL 提供了大量内置函数，但有时这些选项可能并不完全适用。为解决此问题，PostgreSQL 提供了一套非常强大的工具，允许开发者编写用户自定义函数。

实际上，PostgreSQL 更进一步，允许用户编写自己的自定义过程语言。这催生了十多种不同的过程语言，有些包含在 PostgreSQL 中，有些则可在外部获取。在本节中，我们将介绍 PostgreSQL 用户自定义函数的一些基本方面，包括以下内容：

-   如何使用 SQL 编写基本函数
-   如何使用 PL/pgSQL 语言创建更高级的函数
-   在哪里可以找到有关外部过程语言包的更多信息

#### 创建函数语法

在深入探讨用户自定义函数的具体细节之前，我们先来了解一下创建用户自定义函数的语法。所有函数都使用此相同的语法创建。

```
CREATE [ OR REPLACE ] FUNCTION name ( [ [ argmode ] [ argname ]
argtype [, ...] ] )
[ RETURNS returntype ]
{ AS 'definition'
| LANGUAGE langname
| IMMUTABLE | STABLE | VOLATILE
| CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT
| [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER
} ...
[ WITH ( attribute [, ...] ) ]
```

只要不存在另一个同名且参数相同的函数，使用 `CREATE FUNCTION` 命令将会创建一个新函数。使用 `CREATE OR REPLACE` 版本的函数则会创建新函数或覆盖现有定义。命令的下一部分指定了一个 `name`（名称）和可选数量的表示不同数据类型的可选命名参数。接下来指定 `return type`（返回类型），该类型将定义函数在其结果中提供的数据类型。函数 `definition`（定义）通过带引号的字符串提供，该字符串可以用单引号引用，也可以用美元符号（有时称为*美元引号*）引用。`language`（语言）指定将使用哪个过程语言处理程序来执行该函数——例如，对于基于 SQL 的函数使用 `'sql'`，对于基于 PL/pgSQL 的函数使用 `'plpgsql'`。

### PostgreSQL 函数选项详解

`CREATE FUNCTION` 命令的其他选项都与性能或安全相关。三个选项 `IMMUTABLE`、`STABLE` 和 `VOLATILE` 向计划器提供关于函数性质的指示。`IMMUTABLE` 函数是指输出在相同输入下不会改变的函数——例如，如果函数返回两个整数相加的结果。`STABLE` 函数是指那些在给定查询执行期间保持不变的函数，例如在查询中选择 `CURRENT_TIME`，它在事务期间保持静态。`VOLATILE` 函数是指每次执行时输出都不同的函数——例如，将新数据插入表的函数。请注意，如果省略上述级别之一，默认将为 `VOLATILE`。

下一组选项具有类似的性能影响。当程序被标记为 `RETURNS NULL ON NULL INPUT`（也称为 `STRICT`）时，当给定 `NULL` 输入时，它实际上不会被执行；相反，PostgreSQL 将简单地返回 `NULL`。与此相反的是默认操作 `CALLED ON NULL INPUT`，意味着函数会被执行。请注意，如果函数代码中实现了返回 `NULL` 的结果，函数仍可能返回 `NULL`，但代码在任何情况下都会被执行。接下来的两个选项定义函数应以哪个用户的身份执行，要么使用调用用户的默认行为（如 `SECURITY INVOKER`），要么使用函数的原始创建者（如 `SECURITY DEFINER`）。`CREATE FUNCTION` 中的 `WITH *attribute*` 子句是为向后兼容而保留的，但并非必需。

如果这一切看起来有些令人不知所措，别担心。当我们通过几个不同类型函数的示例来逐步讲解时，事情会变得更清晰。此外，所有性能和安全选项最初都可以省略，因为它们都有合理的默认值，在大多数情况下都能正常工作。重要的是熟悉所使用的语法，因为它适用于所有不同类型的函数语言。

### SQL 函数

PostgreSQL 函数最简单的形式是基于 SQL 的函数。SQL 函数无需外部库即可运行，因此内置于每个 PostgreSQL 安装中。SQL 函数本质上不是过程性的；相反，它们只是在其内部封装一个或多个 SQL 查询。即使如此简单，它们仍然可以非常强大，实际上您会发现很多人使用这类函数来简化标准例程。为了更好地理解它们，让我们来看一个简单的 SQL 函数。

```sql
CREATE OR REPLACE FUNCTION firstfunc () RETURNS text AS
$$ SELECT 'hello world'::text;
$$ LANGUAGE 'sql' IMMUTABLE;
```

如您所见，此函数遵循前面章节中介绍的约定。我们从 `CREATE OR REPLACE` 子句开始，然后将我们的函数命名为 `firstfunc`。参数列表留空，因为该函数不接受任何参数。然后我们指定返回类型为 `text`，因为计划返回一个文本字符串。接下来是函数定义，即一个简单的 `SELECT` 语句。接着指定函数要使用的语言，这里是 `SQL`。最后，指定此函数为 `IMMUTABLE`，意味着无论发生什么，它都会产生相同的输出。在 `psql` 中执行此函数的结果如下：

```
phppg=# SELECT firstfunc();

 firstfunc
------------
 hello world
(1 row)
```

如果您还记得上一章，我们创建了一个包含一组员工信息的数据库。在下一个示例中，我们将创建一个函数，将新用户及其薪资信息插入到相应的表中：

```sql
CREATE OR REPLACE FUNCTION newemployee(integer,text,text,integer) RETURNS boolean AS $$
INSERT INTO employee (employee_id,fname,lname)
VALUES ($1,$2,$3);
INSERT INTO salary (employee_id, salary)
VALUES ($1,$4);
SELECT TRUE;
$$ LANGUAGE 'sql' RETURNS NULL ON NULL INPUT;
```

如你所见，此函数遵循与之前示例相同的语法，仅存在少量差异。其中一个差异在于，该函数现在将接收一系列参数，分别对应员工 ID、名、姓和薪资。若查看函数体，你会发现我们使用了一系列 SQL 语句来添加新员工及其薪资。我们还最终选择了 `TRUE` 值，以匹配返回类型并在成功完成时给出响应。最后，我们指明：如果任何输入参数为 `NULL`，则直接返回 `NULL`，因为我们的 `INSERT` 语句将失效。

接下来，我们再次通过 `psql` 程序来查看此函数被调用时的效果。

```
rob=# SELECT newemployee(4,'Amber','Lee',33000);

newemployee
------------
t
(1 row)
```

与之前一样，函数已执行，并返回了 `t`，表示它已运行完成。我们还可以查询相关表，以确保数值已正确插入。

```
rob=# SELECT * FROM employee WHERE employee_id = 4;

employee_id | fname | lname
-------------+-------+-------
4 | Amber | Lee
(1 row)
```

员工已插入，但员工的薪资呢？

```
rob=# SELECT * FROM salary WHERE employee_id = 4;

employee_id | salary
-------------+--------
4 | 33000
(1 row)
```

是的，薪资也正确插入了，因此我们的函数按预期工作。

至此，你应该开始看到使用函数带来的一些好处了。即便是像上述函数这样简单的例子，我们也能设定一个定义明确的流程来添加新员工，确保他们都能获得薪资，并消除诸如使用错误的 `employee_id` 号插入薪资这类小问题。

从更宏观的角度看，拥有用于插入新员工的函数，还能让我们更改表定义。例如，无需更改应用程序代码中插入用户的方式，只要更新函数，就能合并 `employee` 和 `salary` 表。这是一个强大的理念，尤其是在结合更复杂的 SQL 语句或使用诸如 `PL/pgSQL` 等语言实现复杂过程逻辑时。

### 基于 PL/pgSQL 的函数

使用 `PL/pgSQL` 与使用常规 SQL 函数类似。它们都使用相同的语法创建函数，并包含相同的基本部分：函数名、参数列表、返回类型和函数体。然而，在深入学习之前，有几个重要差异需要留意。首先，PostgreSQL 默认未安装 `PL/pgSQL`，因此在使用 `PL/pgSQL` 函数前，需确保你的数据库已安装该扩展。另一个差异在于，`PL/pgSQL` 有自己独特的语法结构来实现过程化任务。对于熟悉其他编程语言的人来说，这种语法并不复杂；但对于习惯于仅使用纯 SQL 的用户而言，则需适应。

在本节中，我们将涵盖这两点差异，并通过几个 `PL/pgSQL` 函数示例进行讲解。

#### 安装 PL/pgSQL

与其他过程语言一样，`PL/pgSQL` 在 PostgreSQL 中并没有内置的解释器。相反，这些语言通过一个 C 语言函数来处理，该函数充当解析、语法分析及执行函数体内代码的胶水代码。基于这种架构，`PL/pgSQL` 在 PostgreSQL 中默认不启用，而必须由数据库管理员安装。在某些平台上，这可以通过操作系统自带的包管理系统完成；若不可行，则可以使用 PostgreSQL 提供的 `createlang` 命令行程序。`createlang` 命令接受语言名称和数据库名称作为参数，例如：`createlang plpgsql template1`。