# 11. 数据

到目前为止，你已经使用 MySQL 存储了一个简单的用户表，其中包含几个列。这对于一些快速示例来说很有用，但如果是具有多重依赖关系的更复杂查询呢？在本章中，让我们规划一些可用于营地注册/管理数据库的数据。

本章包含以下部分。

- 规划新数据库

- 创建新数据库

## 规划新数据库

当数据库在充分考虑数据和表格的情况下组织良好时，其效果最佳。以下是规划数据库时需要考虑的几点：

1. **始终使用正确的数据类型。**

   MySQL 的最佳实践之一是根据信息的性质或内在特性来使用数据类型。使用不必要的数据类型可能会占用更多空间或导致错误。

   例如，使用 `VARCHAR(20)` 而不是 `DATETIME` 数据类型来存储日期时间值，会导致日期时间相关计算出现错误。此外，无效信息也可能混入其中，最终引发错误。

2. **优先使用 `CHAR(1)` 而非 `VARCHAR(1)`。**

   `VARCHAR(1)` 需要额外字节来存储数据，因此如果你的字符串是单个字符，最好使用 `CHAR(1)`。

3. **仅使用 `CHAR` 数据类型存储固定长度的信息。**

   例如，如果信息长度小于 1,000，使用 `CHAR(1000)` 而非 `VARCHAR(1000)` 会占用更多空间。

4. **尽量避免使用地区性日期格式。**

   当使用 `DATETIME` 或 `DATE` 数据类型时，始终使用 `YYYY-MM-DD` 日期格式或适用于你 SQL 引擎的 ISO 日期格式。像 `DD-MM-YYYY` 或 `MM-DD-YYYY` 这样的地区性格式将无法正确存储，导致错误和困扰。

5. **为关键列创建索引。**

   为了使查询快速返回结果，建议为 `JOIN` 条件中使用的列创建索引。

   如果你使用涉及多个表的 `UPDATE` 语句，请为所有用于连接表的列创建索引。

6. **不要对索引列使用函数。**

   这正是索引的目的。试图通过使用函数来复制索引过程，会使情况过于复杂，从而拖慢整个过程。

   例如，假设你需要获取营员名字开头两个字符为 `GE` 的信息。请使用以下语句：

   ```sql
   SELECT firstname FROM campers WHERE firstname like 'GE%'
   ```

   并且，不要编写：

   ```sql
   SELECT firstname FROM campers WHERE left (firstname,2)='GE'
   ```

   第一个示例利用了索引，因此响应时间更快。

7. **仅在必要时使用 `ORDER BY` 子句。**

   让 PHP 来排序你的数据，而不是 MySQL。使用 MySQL，你可以为返回的数据设置排序方式，例如 `ASC` 表示升序，`DESC` 表示降序。这可能会导致查询花费额外的时间，而这些工作完全可以由 PHP 甚至前端的 JavaScript 来完成。

8. **选择合适的数据引擎。**

   如果你开发的是一个读取数据频率高于写入数据频率的应用程序（例如搜索引擎），请选择 `MyISAM` 存储引擎。

   选择错误的存储引擎会影响性能。可供你选择的存储引擎包括 `MyISAM`（MySQL 默认的存储引擎）和 `InnoDB`（MySQL 内置的另一种引擎，专为高性能数据库设计）。这两种引擎的主要区别之一是表锁定与行级锁定。表锁定是指当表中的某个或多个单元格需要更新或删除时，锁定整个表的技术。表锁定是默认存储引擎 `MyISAM` 采用的默认方法。行级锁定是指在修改或删除表中某个有效范围内的单元格时，锁定该范围内行数据的技术。行级锁定是 `InnoDB` 存储引擎使用的方法，专为高性能数据库设计。

9. **在必要时使用 `EXISTS` 子句。**

   当你只需要检查数据是否存在时，请使用 MySQL 的 `EXISTS` 函数，而不是发起整个查询来评估返回数据。例如，使用：

   ```sql
   IF EXISTS(SELECT * from Table WHERE col='foo')
   ```

   不要使用：

10. **使用 `EXPLAIN` 分析你的 `SELECT` 查询。**

    ```sql
    IF (SELECT count(*) from Table WHERE col='foo')>0
    ```

    MySQL 提供了 `EXPLAIN` 功能，用于显示 MySQL 如何执行查询的过程，例如

    ```sql
    mysql> EXPLAIN ANALYZE SELECT * FROM SALES;
    +----------------------------------------------------+
    | EXPLAIN                                            |
    +----------------------------------------------------+
    | –> Table scan on SALES (cost=0.35 rows=1) (actual time=0.070..0.070 rows=0 loops=1)                    |
    +----------------------------------------------------+
    1 row in set (4.15 sec)
    ```

## 创建新数据库

结合目前学到的所有知识，让我们创建具有以下结构的新数据库：

- **表名：** `Campers`

- **列：** `ID`、`First Name`、`Last Name`、`Age`、`Camp ID`、`Created`

- **表名：** `Camps`

- **列：** `ID`、`Name`、`Size`、`Created`

- **表名：** `Registered`

- **列：** `ID`、`Camper ID`、`Camp ID`、`Registered`、`Paid`、`Created`

这是一个供营地用于跟踪露营者、营地和注册信息的基本数据库设计。为了在你的项目中正确使用它，你需要设置并创建这些表，用数据填充它们，并通过表关系使用 PHP 进行管理。虽然有几种管理 MySQL 的前端 GUI 方法，但目前你不会使用其中任何一种。我们将带你通过 MySQL 的命令行界面 (CLI) 完成后续步骤。首先，打开命令提示符（Windows）或终端（Mac OS, Linux）并运行：

```bash
docker ps
```

还记得这个命令吗？此命令显示所有正在运行的容器。如果没有显示任何内容，那么你可能没有为本书运行 Docker。请回到本书的第一章，确保 Docker 正在运行并且 `docker-compose up` 已经执行。

如果一切运行正常，你应该会看到类似于以下的内容：

```bash
docker ps
CONTAINER ID   IMAGE                          COMMAND     CREATED         STATUS      PORTS                                                  NAMES
d5d98b7de503   beginning-php8-and-mysql_app   "docker-php-entrypoi…"   2 days ago      Up 2 days   9000/tcp                                               php-app
63715c3c4f52   nginx:alpine                   "/docker-entrypoint.…"   2 days ago      Up 2 days   0.0.0.0:80->80/tcp, :::80->80/tcp                      php-nginx
21f2a4b87b7b   mysql:8.0                      "docker-entrypoint.s…"   2 days ago      Up 2 days   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql-db
```

你可以看到你的 MySQL 容器名为 `mysql-db`，使用 Docker 你可以像连接服务器一样连接到这个容器。

```bash
docker exec -ti mysql-db bash
```

该命令告诉 Docker 你想执行一个名为 `-ti` 的命令来创建一个伪 TTY。这基本上允许你将终端用作此容器的接口，而 `i` 代表交互模式，意味着你想将这个容器当作一个实时系统来使用。下一个属性是 `mysql-db`，这是你要连接的容器名称，最后你想运行 bash。Bash 是 Unix/Linux 系统的一个 shell，允许你访问文件系统和运行脚本。按下回车后，你就“进入”了 MySQL 容器。连接后，你需要运行以下命令：

# MySQL 数据库操作入门

## 连接数据库

使用以下命令连接到 MySQL：

```
mysql -uroot -ppass
```

这将使用用户名 `root` 和密码 `pass` 连接到 MySQL。通常不鼓励这样做，但在封闭网络环境和开发环境中可以允许这种随意性。

连接到 MySQL 后，你需要了解并做的第一件事是列出数据库。

```
show databases;
```

注意，所有 MySQL 命令都以分号结尾。如果你输入一个命令并按回车但没有加分号，命令行只会换行并等待你输入更多内容或输入分号。你可以只输入一个分号然后按回车来继续执行你的命令。

## 使用数据库

在这个列表中，你应该会看到 `beginningPHP`。输入：

```
use beginningPHP;
```

然后输入：

```
show tables;
```

这个命令会显示当前数据库中可用的表。应该有一个 `users` 表。这没问题，我们暂时先把它放一边。让我们开始为你的露营数据创建表。

## 创建表

在 `chapter5` 目录下有一个名为 `campers.sql` 的文件。

```
create table IF NOT EXISTS campers(
id INT NOT NULL AUTO_INCREMENT,
first_name VARCHAR(100) NOT NULL,
last_name VARCHAR(40) NOT NULL,
age INT NOT NULL,
campId INT default 0,
created DATETIME NOT NULL ON UPDATE CURRENT_TIMESTAMP default current_timestamp,
PRIMARY KEY ( id )
);
```

复制这段代码，粘贴到 MySQL 命令行中并按下回车。然后再次输入 `show tables;`。

```
mysql> show tables;
+------------------------+
| Tables_in_beginningPHP |
+------------------------+
| campers                |
| users                  |
+------------------------+
2 rows in set (0.00 sec)
```

现在数据库中有了一个 `campers` 表。要查看表结构，请输入：

```
desc campers;
```

`Desc` 是 `Describe` 的缩写，它会显示表的布局。现在让我们创建用于存储营地信息的表。查看 `chapter5` 目录中的 `camps.sql` 文件。

```
create table IF NOT EXISTS camps(
id INT NOT NULL AUTO_INCREMENT,
camp_name VARCHAR(100) NOT NULL,
size INT NOT NULL,
created DATETIME NOT NULL ON UPDATE CURRENT_TIMESTAMP default current_timestamp,
PRIMARY KEY ( id )
);
```

复制这段代码，粘贴到 MySQL 命令行中并按下回车。

现在输入 `show tables;` 并查看结果：

```
show tables;
+------------------------+
| Tables_in_beginningPHP |
+------------------------+
| campers                |
| camps                  |
+------------------------+
2 rows in set (0.03 sec)
```

最后，让我们创建一个用于存储已注册露营者的表。按照上述步骤处理 `registered.sql`。打开 `registered.sql`。复制代码，粘贴到 MySQL 命令行中并按下回车。输入 `show tables;` 并查看结果。现在你已经有了数据，让我们接下来看看如何利用关系查询为你的应用创建简单和复杂的查询。

## 总结

在本章中，你学习了哪些数据类型可以在 MySQL 数据库表中使用，例如 `CHAR` 或 `VARCHAR`，以及如何在查询中定义多个依赖关系。在下一章中，你将把所有学到的知识整合到一个示例网站中，以创建、读取、更新和删除数据（也称为 CRUD）。你将学习一个基本的 CRUD 网站如何成为企业或组织内管理信息的标准方式。