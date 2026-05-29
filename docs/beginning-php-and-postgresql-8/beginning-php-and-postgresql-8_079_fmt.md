# 索引与搜索

### 文本搜索

借助`PostgreSQL`中的`tsearch2`模块，用户现在能够针对已建立索引的任意字段中的大量文本进行搜索优化。

### 索引类型

索引可分为四大类别：主键索引、唯一索引、普通索引和全文索引。本节将逐一介绍每种类型。

#### 主键索引

主键索引是关系数据库中最常见的索引类型。通常做法是，行的主键值由特定于该键列的自动递增整数值决定。这确保了无论后续是否删除了已有的行，每一行都将拥有唯一的主键条目。例如，假设你要为公司的 IT 团队创建一个实用网站数据库。这个表可能如下所示：

```sql
CREATE TABLE webresource (
    row_id SERIAL NOT NULL,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    description TEXT NOT NULL,
    PRIMARY KEY (row_id)
);
```

这种形式的主键通常被称为代理键，它是一种常用的唯一标识一行的方法。该方法的主要优点在于它不依赖于行内保存的数据，因此无论你如何更改底层数据，标识符都无需修改。

#### 唯一索引

与主键索引类似，唯一索引可防止创建重复值。然而，区别在于每个表只允许有一个主键索引，但却支持多个唯一索引。考虑到这一点，或许值得重新审视上一节中的`webresource`表。虽然两个网站可能同名（例如"优秀 PHP 资源"），但重复的 URL 则毫无意义。这看起来是一个理想的唯一索引场景：

```sql
CREATE TABLE webresource (
    row_id SERIAL NOT NULL,
    name TEXT NOT NULL,
    url TEXT NOT NULL UNIQUE,
    description TEXT NOT NULL,
    PRIMARY KEY (row_id)
);
```

我们也可以事后使用`CREATE INDEX`命令来添加唯一索引。这在功能上等同于在创建表时指定唯一性。但是，它不会出现在诸如`psql`等工具的表约束列表中。

```sql
CREATE UNIQUE INDEX webresource_url_unique_idx ON webresource (url);
```

如前所述，可以在一个表中将多个字段指定为唯一。考虑以下示例。假设你希望防止仓库贡献者在插入新网站时重复使用无意义的名称（例如"酷站"）。重新审视原始的`webresource`表，将`name`列定义为唯一：

```sql
CREATE TABLE webresource (
    row_id SERIAL NOT NULL,
    name TEXT NOT NULL UNIQUE,
    url TEXT NOT NULL UNIQUE,
    description TEXT NOT NULL,
    PRIMARY KEY (row_id)
);
```

你也可以指定多列唯一索引。例如，假设你允许贡献者插入重复的`name`值，甚至重复的`url`值，但不希望出现重复的`name`和`url`组合。你可以通过创建多列唯一索引来强制执行此类限制。重新审视原始的`webresource`表，结果如下：

```sql
CREATE TABLE webresource (
    row_id SERIAL NOT NULL,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    description TEXT NOT NULL,
    UNIQUE (name,url),
    PRIMARY KEY (row_id)
);
```

根据此配置，以下`name`和值对可以同时存在于同一个表中：

- Apress 网站，`http://www.apress.com`
- Apress 网站，`http://blogs.apress.com`
- 博客，`http://www.apress.com`
- Apress 博客，`http://blogs.apress.com`

然而，再次尝试插入上述任何组合都会导致错误，因为重复的`name`和`url`组合是非法的。

#### 普通索引

通常情况下，你希望优化对主键或唯一键字段之外的字段的搜索。由于这种情况非常普遍，因此通过为这些字段建立索引来优化此类搜索自然是合理的。这类索引通常被称为普通索引。

##### 单列普通索引

如果表中的某一特定列将成为大量选择查询的焦点，则应使用单列普通索引。例如，假设员工档案表包含四列：唯一的行 ID、名字、姓氏和电子邮件地址。你知道大部分搜索将针对员工的姓氏或电子邮件地址。你可以为姓氏创建一个普通索引，为电子邮件地址创建一个唯一索引，如下所示：

```sql
CREATE TABLE employee (
    employeeid SERIAL NOT NULL PRIMARY KEY,
    firstname TEXT NOT NULL,
    lastname TEXT NOT NULL,
    email TEXT NOT NULL
);

ALTER TABLE employee CREATE UNIQUE INDEX employee_email_unique_idx ON
employee (email);

ALTER TABLE employee CREATE INDEX employee_last_name_idx ON
employee (lastname);
```

然而，选择查询通常是包含多个列的函数。毕竟，更复杂的表可能需要查询包含多个列才能检索到所需数据。通过建立多列普通索引（下文讨论），可以大大缩短此类查询的运行时间。

##### 多列普通索引

当你知道多个指定列经常一起用于检索查询时，建议使用多列索引。PostgreSQL 的多列索引方法基于最左前缀策略。最左前缀原则指出，任何包含列 A、B 和 C 的多列索引都将提高涉及以下列组合的查询性能：

- A, B, C
- A, B
- A

以下是创建 PostgreSQL 多列索引的方法：

```sql
CREATE TABLE employee (
    employeeid SERIAL NOT NULL PRIMARY KEY,
    firstname TEXT NOT NULL,
    lastname TEXT NOT NULL,
    email TEXT NOT NULL,
    city TEXT NOT NULL
);

ALTER TABLE employee CREATE INDEX employee_lastname_firstname_idx ON
employee (lastname, firstname);
```

像这样创建索引非常有用，因为它将提高涉及以下任何列组合的查询的搜索速度：

- `lastname`、`firstname`
- `lastname`

为了进一步说明，以下查询将受益于这个多列索引：

```sql
SELECT email FROM employee WHERE lastname='Dylan' AND firstname='Robert';
SELECT * FROM employee WHERE lastname='Russell';
```

而以下查询则不会：

```sql
SELECT lastname FROM employee WHERE firstname = 'Amber';
SELECT email FROM employee WHERE city='Breesport';
```

为了提升这两个查询的性能，你需要分别为`firstname`和`city`创建单独的索引。

#### 位图索引

在大多数版本的 PostgreSQL 中，数据库限制每个查询只能使用一个索引。即使你在不同字段上有两个单列索引，PostgreSQL 也只会选择一个索引并使用它，而忽略另一个。假设你向`employee`表添加了一个额外的索引，然后使用多个列进行查询。

```sql
CREATE INDEX employee_city_idx ON employee (city);
```

```sql
SELECT * FROM employee WHERE lastname='Lee' AND city='Horseheads';
```

同样，在这种情况下，服务器会使用`lastname`索引或`city`索引，但不会同时使用两者。如果这种查询会频繁发生，你可以创建一个多列索引。但如果你还希望继续仅根据`lastname`或仅根据`city`字段进行查询呢？使用多列索引时，这两个查询中的一个将无法利用该索引。

为了解决这个问题，PostgreSQL 在 8.1 版本中新增了一种使用索引的方法，称为*位图索引扫描*。其作用是让 PostgreSQL 能够同时使用两个单列索引。用非常通俗的话来说，PostgreSQL 会同时对这两个索引进行查询，然后在内存中合并结果，并返回匹配的行。这通常能让查询比在单个索引上搜索更快，并且通过消除对多列组合索引或重复索引（用于处理针对多列索引中第二列的查询）的需求来节省空间。你需要注意，这并不一定能完全消除对多列索引的需求；如果你计划总是针对表中的两列进行查询，使用一个多列索引进行搜索可能仍然比使用两个独立的单列索引更快。不过，如果你使用的是 8.1 版本，这一点值得了解。

### 部分索引

有时，你会遇到这样的情况：需要对某一列中一小部分值进行重复查询，而不是查询该列的全部值范围。在这些情况下，PostgreSQL 允许你仅针对指定的列值创建部分索引。部分索引通过在 `CREATE INDEX` 命令中添加 `WHERE` 子句来实现。每次你向表中插入或更新一行时，都会评估 `WHERE` 子句，如果该行的值满足 `WHERE` 子句，则该行会被包含在索引中。为了让这一点更清楚，我们来查看一个更具体的例子。假设你在 `employee` 表中添加一列，用于跟踪新员工，这些员工在入职的前 30 天内被视为临时工：

```sql
ALTER TABLE employee ADD COLUMN istemp BOOLEAN;
```

当一名员工刚被雇佣时，其 `istemp` 列会被设置为 `true`，但一旦该员工入职超过 30 天，此字段会被设置为 `false`。在这种场景下，你可能会运行许多查询，比如报告哪些员工是临时工，或者你在哪些城市有临时员工。因为对于大多数已经工作超过 30 天的员工来说，这类报告并不重要，因此这对于创建一个部分索引来说是一个理想的选择：

```sql
CREATE INDEX employee_istemp_idx ON employee (istemp) WHERE istemp IS TRUE;
```

现在，任何指定了 `WHERE istemp IS TRUE` 的查询都可以利用这个索引。除了加快查询速度之外，部分索引还提供以下好处：

- 由于部分索引只包含表中行的一个子集，它们需要的磁盘空间比普通索引少。

- 由于部分索引中包含的行较少，因此部分索引的维护成本更低。

- 当针对部分索引进行查询时，PostgreSQL 需要搜索的行会更少，从而使查询速度更快。

### 函数索引

除了部分索引，PostgreSQL 还提供了一种创建通用*函数索引*的方法。函数索引的工作原理与部分类似：索引通过稍微修改的语法创建，该语法用于评估表中的新条目。关键区别在于，函数索引不存储所讨论列的值；相反，它们存储函数返回的值，当在查询中使用该索引时，返回该值。让我们再看看另一个例子。这次，你将在表中添加一些电话号码信息：

```sql
ALTER TABLE employee ADD COLUMN telephone TEXT;
```

一旦你填充了数据，就可以使用 PostgreSQL 内置的 `substring` 函数，根据电话号码的交换码部分（即前三位数字）进行查询，以查找居住在某个区域附近的人，即使他们可能不住在同一个城市。当然，如果你需要对数千名员工运行此查询，速度可能会有点慢，因此你需要创建一个函数索引来加快速度：

```sql
CREATE INDEX employee_npa_idx ON employee(SUBSTR(telephone,1,3));
```

现在，每当查询在其 `WHERE` 子句中包含函数调用 `SUBSTR(telephone,1,3)` 时，PostgreSQL 就可以利用这个函数索引。

### 全文索引

正如你在前面章节中所看到的，PostgreSQL 为用户提供了许多扩展数据库功能的方法。其中一个很好的例子是 `tsearch2` 模块，它为 PostgreSQL 数据库提供了全文索引能力。

#### 获取 tsearch2

由于 `tsearch2` 默认并未加载到 PostgreSQL 中，你需要额外执行几个步骤才能使用此特定功能。安装 `tsearch2` 的过程因操作系统和数据库安装的具体细节而略有不同，但总的来说，有三种方法可以获取 `tsearch2`：

- 自行下载并编译源代码。

- 通过适用于你操作系统的相应包（rpm, zip）下载并安装 `tsearch2`。

- 购买一个二进制发行版。

覆盖所有这些方法超出了本书的范围。不过，我们将介绍在 Linux 上从源代码进行典型安装的步骤。如果你是从包安装，其中一些或所有步骤可能已经为你执行了。

第一步是获取 `tsearch2` 的源代码；你可以在 PostgreSQL 源代码发行版的 `contrib` 目录下的 `tsearch2` 目录中找到该包。安装好 PostgreSQL 并使其运行后，进入此目录并执行以下命令：

```bash
[rob@ridley tsearch2]$ make;
[rob@ridley tsearch2]$ make install;
```

执行完这些命令后，你会在屏幕上看到大量信息滚动而过，然后你会回到命令提示符。此时，`tsearch2` 已经安装，但你仍然需要配置 PostgreSQL 以使用这个新模块。要配置数据库以使用 `tsearch2`，请将 `tsearch2.sql` 文件（同样位于 `contrib/tsearch2` 目录中）加载到你想在其中进行全文搜索的数据库中：

```bash
[rob@ridley tsearch2]$ psql phppg -f tsearch2.sql
```

再次强调，你会看到大量信息滚动而过，当它们完成后，`tsearch2` 将安装到指定的数据库（上例中的 `phppg`）中。你可以通过查找以下四个 `tsearch2` 表来确认安装：

```sql
phppg=# \dt pg_ts_*
        关系列表
  架构模式 |     名称     | 类型 |  拥有者
----------+--------------+------+----------
 public   | pg_ts_cfg    | 表   | rob
 public   | pg_ts_cfgmap | 表   | rob
 public   | pg_ts_dict   | 表   | rob
 public   | pg_ts_parser | 表   | rob
(4 行)
```

数据库中还将新增几个函数和运算符；完整列表请参阅 `tsearch2` 在线文档。至此，你可以开始使用 `tsearch2` 了。

### 使用 `tsearch2`

由于 PostgreSQL 假设全文本搜索将被用于筛选大量自然语言文本，因此必须建立一种机制来检索数据，并产生最符合用户期望结果的结果输出。更具体地说，如果用户使用类似 "Apache is the world’s most popular Web server" 这样的字符串进行搜索，那么 "is" 和 "the" 这些词在确定结果相关性时应该几乎没有作用。事实上，PostgreSQL 会将可搜索文本拆分为单词，默认情况下会排除停用词列表中的任何单词。你可以修改这个停用词列表，我们稍后会讨论。

创建全文本索引与其他类型的索引略有不同。为了更好地说明这个过程，让我们重新回顾本章前面使用的 `webresource` 表，并使用全文本变体对其 `description` 列进行索引：

```sql
CREATE TABLE webresource (
  webresourceid SERIAL NOT NULL,
  name TEXT NOT NULL,
  url TEXT NOT NULL,
  description TEXT NOT NULL,
  UNIQUE (name,url),
  PRIMARY KEY (webresourceid)
);
```

并添加一些数据以供操作：


```
INSERT INTO webresource(name, url, description) VALUES
('Ruby Home Page','http://www.ruby-lang.org/','The official
Ruby website');
INSERT INTO webresource(name, url, description) VALUES
('Apache Site','http://httpd.apache.org/','Great Apache site,
contains Apache 2 manual');
INSERT INTO webresource(name, url, description) VALUES
('Planet PostgreSQL', 'http://www.planetpostgresql.org/',
'PostgreSQL community bloggers');
INSERT INTO webresource(name, url, description) VALUES
('PHP: Hypertext Preprocessor','http://www.php.net/',
'The official PHP website');
INSERT INTO webresource(name, url, description) VALUES
('Apache Week','http://www.apacheweek.com/',
'Offers a dedicated Apache 2 section');
```

现在，你需要在 `webresource` 表中添加一列，用于存储全文本索引信息：

```
ALTER TABLE webresource ADD COLUMN description_fti_idx tsvector;
```

创建好存储索引的位置后，你可以继续添加新索引：

```
CREATE INDEX webresource_description_fts_idx ON webresource USING
gist(description_fti_idx);
```

最后，在开始使用该索引之前，你需要实际创建索引数据。为此，请执行以下命令：

```
UPDATE webresource SET description_fti_idx = to_tsvector(description);
```

根据数据库的配置方式，你可能会收到类似于 `ERROR: Can't find tsearch config by locale` 的错误信息。这并不意味着你的 `tsearch2` 安装有问题，而是表明你的 `tsearch2` 安装无法识别数据库当前的区域设置。

通常有三种方法可以解决此问题。第一种是重新运行 `initdb` 程序并指定 `--locale=C`。第二种是为 `tsearch2` 创建一个新的区域配置，以匹配数据库中使用的区域设置。这是一个相当复杂的过程，但也是最全面的。有关如何创建自己的区域设置的详细信息，请访问 `http://www.sai.msu.su/~megera/oddmuse/index.cgi/tsearch-v2-intro`。

最后一种方法（也是我们向 `tsearch2` 新手推荐的方法）是通过使用额外的函数参数，强制 `tsearch2` 使用 C 区域设置。为此，你只需将之前的 `UPDATE` 语句替换为以下内容：

```
UPDATE webresource SET description_fti_idx = to_tsvector('default', description);
```

这种语法强制 `tsearch2` 使用 C 区域设置，该设置适用于大多数系统。如果你使用此方法，还需要将额外的参数（`'default'` 部分）添加到本章后面使用的 `to_tsquery()` 和 `headline()` 函数中。
