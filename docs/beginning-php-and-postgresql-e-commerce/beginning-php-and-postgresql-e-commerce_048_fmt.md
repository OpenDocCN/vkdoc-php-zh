# 第三章 创建产品目录：第一部分

与`UPDATE`命令一样，使用`DELETE`命令时也需要小心，因为如果忘记指定`WHERE`子句，将删除表中的所有行。下面的查询将删除`department`表中的所有记录。`DELETE`命令本身不会删除表。

```
DELETE FROM department;
```

## PostgreSQL 函数和类型

数据库函数是存储用 PostgreSQL 能理解的语言编写的程序的对象。PostgreSQL 能够处理用多种语言（如 Perl 或 Python）编写的函数，但我们将使用标准的基于 SQL 的语言——`PL/pgSQL`。你可以在 http://www.postgresql.org/docs/current/interactive/plpgsql.html#PLPGSQL-OVERVIEW 找到该语言的介绍。

如果你只想执行数据库操作，并不一定需要使用数据库函数。你可以直接从外部应用程序（如 HatShop 应用的 PHP 脚本）向 PostgreSQL 数据库发送 SQL 命令。使用函数时，你无需传递要执行的 SQL 代码，只需调用函数并传入其可能需要的参数值即可。使用函数进行数据操作有以下优点：

- 性能可能更好，因为 PostgreSQL 在函数首次执行时会生成并缓存其执行计划。

- 使用函数可以更好地维护数据访问和操作代码，这些代码存储在一个中心位置，并且更容易实现三层架构（数据库函数构成数据层）。

- 安全性可以更好地控制，因为 PostgreSQL 允许为每个单独的函数设置不同的安全权限。

- 在 PHP 代码中临时创建的 SQL 查询更容易受到 SQL 注入攻击，这是一个主要的安全威胁。（许多网络资源都涵盖了这个安全主题，例如 http://www.sitepoint.com/article/sql-injection-attacks-safe 上的文章。）

- 这可能是个人偏好问题，但将 SQL 逻辑与 PHP 代码分离可以使 PHP 代码更清晰、更易于管理；调用函数看起来比拼接字符串来构建传递给数据库的 SQL 查询要好得多。

在开发 HatShop 时，我们将把所有数据访问代码保存为`hatshop`数据库中的 PostgreSQL **函数**。这些函数，像任何主流语言中的函数一样，具有输入参数和返回类型。在某些示例中，我们将定义自定义类型用于返回结果。

创建函数的语法是：

```
CREATE FUNCTION <name>(<param1 type>, <param2 type> ... ) RETURNS [SETOF] <return type> LANGUAGE plpgsql AS $$
<code>
$$
```

或者，你也可以在函数代码的末尾（在关闭的`$$`之后）指定语言（`LANGUAGE plpgsql`）。

请注意，如果数据库中已经存在具有相同名称、参数数量和参数类型的函数，则无法创建该函数。但是，你可以拥有多个同名但参数不同的函数（这种方法在面向对象编程（OOP）术语中称为*重载*）。关键在于，在调用函数时，PostgreSQL 必须知道要调用哪个版本的函数，如果参数不同，它可以做到这一点。

要修改现有函数，应使用`CREATE OR REPLACE FUNCTION`而不是`CREATE FUNCTION`，如果函数不存在，它会创建函数；如果函数存在，则会更新函数。

我们将使用类型来指定函数返回的数据类型。类型的定义如下：

```
CREATE TYPE name AS
( attribute_name data_type [, ... ] )
```

对于部门列表的数据层，你需要创建一个名为`department_list`的类型和一个名为`catalog_get_departments_list`的函数。让我们在下面的练习中完成这个任务。

## 练习：创建 PostgreSQL 类型和函数

1. 启动 pgAdmin III，并使用`hatshopadmin`用户名连接到`hatshop`数据库。

2. pgAdmin III 具有可用于创建类型的界面元素，无需编写代码（通过右键单击“类型”，然后选择“新建类型”即可查看）。但是，由于界面不太友好，我们更倾向于编写并执行执行相同操作的代码。选择“工具”➤“查询工具”，并编写以下代码：

   ```
   CREATE TYPE department_list AS
   (
       department_id INTEGER,
       name VARCHAR(50)
   );
   ```

3. 按`F5`执行命令。输出应为类似于“查询成功返回，在 30 毫秒内无结果。”

4. 选择“编辑”➤“清除窗口”以清除当前内容，并编写以下创建`catalog_get_departments_list`函数的代码：

   ```
   CREATE FUNCTION catalog_get_departments_list()
   RETURNS SETOF department_list LANGUAGE plpgsql AS $$
   DECLARE
       outDepartmentListRow department_list;
   BEGIN
       FOR outDepartmentListRow IN
           SELECT department_id, name
           FROM department
           ORDER BY department_id
       LOOP
           RETURN NEXT outDepartmentListRow;
       END LOOP;
   END;
   $$;
   ```

5. 按`F5`执行命令。输出应再次提示命令已成功执行，无结果。

6. 要测试新函数是否返回预期结果，再次清除窗口内容，并输入以下查询。应该会检索到部门列表（参见图 3-13）。

   ```
   SELECT * FROM catalog_get_departments_list();
   ```

**图 3-13.** *使用 pgAdmin III 执行函数*

7. 关闭查询工具窗口。如果系统询问是否保存更改，请点击“否”。

## 工作原理：PostgreSQL 类型和函数

让我们逐步分解`catalog_get_departments_list`函数。第一行是定义函数名称的行。请记住，如果你已经创建了函数并想要修改它，可以使用`CREATE OR REPLACE FUNCTION`。

```
CREATE FUNCTION catalog_get_departments_list()
```

下一行定义了返回类型和函数中使用的语言。我们在本书中用于此函数以及所有其他函数的语言是 PL/pgSQL（`LANGUAGE plpgsql`）。

```
RETURNS SETOF department_list LANGUAGE plpgsql AS $$
```

返回类型是`SETOF department_list`，这意味着该函数应返回一个或多个其结构由`department_list`类型定义的记录。`department_list`类型是一个由`DepartmentID`和`Description`组成的简单类型，定义如下：

```
CREATE TYPE department_list AS
(
    department_id INTEGER,
    name VARCHAR(50)
);
```

函数体位于开始和结束的`$$`之间。以下代码片段代表了我们编写返回数据的函数的典型方式。粗体行执行我们感兴趣的查询，其余部分是返回该查询结果所需的辅助代码。

```
DECLARE
    outDepartmentListRow department_list;
BEGIN
    FOR outDepartmentListRow IN
        SELECT department_id, name FROM department
    LOOP
        RETURN NEXT outDepartmentListRow;
    END LOOP;
END;
```

那么这里发生了什么？函数体以`DECLARE`部分开始，该部分声明了函数将使用的变量。与其它语言中的函数不同，PL/pgSQL 函数有一个专门声明变量的特殊位置。在本例中，变量名是`outDepartmentListRow`，其类型是`department_list`。