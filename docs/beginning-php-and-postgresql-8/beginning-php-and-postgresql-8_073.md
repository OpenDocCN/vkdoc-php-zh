# 如何在 PostgreSQL 中创建和操作视图

- 如何在 PostgreSQL 中创建和操作视图
- PostgreSQL 的规则系统，包括规则系统中可以使用的命令类型
- 可更新视图以及如何利用 PostgreSQL 的规则系统实现这种经典关系型概念的强大版本

### 使用视图

在处理大型数据模型时，您经常需要使用复杂查询从多个连接表中检索信息，这些查询通常带有很长的`WHERE`条件列表。在应用程序代码的不同部分重复这些复杂查询往往很麻烦，尤其是当您的数据库有多个接口时。这些接口之间的细微差异可能导致最终结果不匹配，从而引发问题。

这里一个方便的做法是为复杂查询命名，这样它就可以存储在数据库中，并由外部应用程序以统一的方式访问。这正是视图的作用。在 PostgreSQL 中，*视图*被定义为给定查询的存储表示形式。一旦定义，视图在许多方面可以被视为一个虚拟表。虽然视图本身不存储数据，但它可以像任何其他表一样被查询，您甚至可以基于其他视图创建视图。

#### 创建视图

您可以使用`CREATE VIEW`语句创建视图。在使用`CREATE VIEW`语句时，需要为视图指定名称以及定义视图结构的 SQL 查询：

```sql
CREATE VIEW database_books AS SELECT * FROM books WHERE subject = 'PostgreSQL';
```

这将创建一个名为`database_books`的视图，该视图在列名及其类型方面与`books`表具有等效的结构。在旧版本的 PostgreSQL（7.2 及更早版本）中，如果需要更改视图的定义，必须先删除视图，然后重新创建它。但在当前版本中，我们可以利用`CREATE OR REPLACE VIEW`命令：

```sql
CREATE OR REPLACE VIEW database_books AS SELECT * FROM books
WHERE subject = 'PostgreSQL' OR subject = 'Sqlite';
```

> **注意**：`CREATE OR REPLACE VIEW`命令仅在您不更改列名布局或其类型时有效。如果您的查询产生不同的列布局，您将需要删除视图然后重新创建。

您可以使用非常复杂的 SQL 语句创建视图，并且不需要将视图限制为现有表中的列；派生值和常量也是可接受的，如下例所示：

```sql
CREATE VIEW featured_technical_books AS SELECT 'Best Sellers', title, (copies_sold / months_in_print) AS average_sales FROM books WHERE
current_stock >= 1000 AND genre = 'technical';
```

#### 删除视图

删除视图通过使用`DROP VIEW`命令完成。该命令接受可选的关键字`RESTRICT`或`CASCADE`，以确定在处理依赖对象（例如查询正在删除的视图的其他视图或函数）时的行为。

- `RESTRICT`关键字防止视图在其存在依赖对象时被删除。
- `CASCADE`关键字（如下例所示）会级联删除依赖对象：

```sql
DROP VIEW database_books CASCADE;
```

### PostgreSQL 规则系统

许多数据库在数据库内部使用基于函数和触发器组合的主动规则来强制执行数据约束和外键等操作。PostgreSQL 也提供这些功能，但它还提供了一种替代系统用于即时重写查询。

使用规则系统，您可以执行诸如强制数据完整性、创建只读表以及使视图可交互等操作。这些“查询重写”规则可以分为四种类型：`SELECT`、`INSERT`、`UPDATE`和`DELETE`。在本节中，我们将介绍创建规则的语法，并介绍这四种不同类型的规则。

#### 使用规则

本节介绍用于创建和删除规则的基本语法。



[www.it-ebooks.info](http://www.it-ebooks.info/)

