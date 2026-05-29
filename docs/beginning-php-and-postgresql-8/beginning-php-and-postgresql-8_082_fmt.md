# 第 37 章 导入和导出数据

### 导出表 OID

如果目标表是使用 OID（对象标识符）创建的，您可以通过使用`OIDS`子句指定 OID 与表数据的其余部分一起输出。例如：

```
psql>COPY employee TO STDOUT OIDS;
```

这将产生：

```
24627 1 GM100011 Jason Gilmore jason@example.com
24628 2 RT435234 Robert Treat rob@example.com
24629 3 GS998909 Greg Sabino Mullane greg@example.com
24630 4 MW777983 Matt Wade matt@example.com
```

### 更改默认分隔符

请注意，上一个示例输出中每列之间出现的空白实际上是制表符（`\t`），这是默认的分隔符。通过使用`DELIMITER`子句，您可以将默认分隔符更改为，例如，竖线（`|`）：

```
psql>COPY employee TO STDOUT DELIMITER '|';
```

这将以以下格式返回前述输出（不含 OID）：

```
1|GM100011|Jason Gilmore|jason@example.com
2|RT435234|Robert Treat|rob@example.com
3|GS998909|Greg Sabino Mullane|greg@example.com
4|MW777983|Matt Wade|matt@example.com
```

同样，如果您要导入的文本文件不使用制表符分隔字段，请像前面的命令一样指定它：

```
psql>COPY employeenew FROM '/home/jason/sqldata/employee.sql' DELIMITER |;
```

### 仅复制特定列

如果您只想将员工的姓名和电子邮件地址复制到标准输出，请像这样指定列名：

```
psql>COPY employee (name,email) TO STDOUT;
```

这将产生以下结果：

```
Jason Gilmore jason@example.com
Robert Treat rob@example.com
Greg Sabino Mullane greg@example.com
Matt Wade matt@example.com
```

同样，如果文本文件只包含要插入到表中的部分字段，您可以像之前那样指定它们。但请记住，其他表列需要要么可为空，要么具有默认值。

### 处理 NULL 值

虽然电子邮件对于在办公环境中工作的个人来说是一个重要的通信工具，但假设有些员工只在仓库工作，因此不需要电子邮件。因此，`employee`表中的某些电子邮件值可能为`NULL`。使用`COPY`导出数据时，`NULL`值的默认值是`\N`，而在使用 CSV 模式（将在本节后面讨论）时，它是一个空字符串。但是，如果您想为此类实例声明自定义字符串，例如`no email`，该怎么办？您应该使用`NULL`子句，如下所示：

```
psql>COPY employee TO STDOUT NULL 'no email';
```

这会产生类似于以下的输出（假设某些员工的电子邮件地址已设置为`NULL`）：

```
Jason Gilmore no email
Robert Treat rob@example.com
Greg Sabino Mullane greg@example.com
Matt Wade no email
```

同样，如果您从文本文件导入数据并且指定了`NULL`值，则无论何时找到该值，相应的列都将被设置为`NULL`。

### 使用 CSV 文件

逗号分隔值（CSV）文件是当今几乎所有主流关系数据库都接受的格式，更不用说各种各样的产品，如 Microsoft Excel。您可以通过使用带有`CSV`子句的`COPY`轻松地从 PostgreSQL 表创建 CSV 文件。例如，要创建一个可以在 Microsoft Excel 或 OpenOffice.org Calc 中立即查看的文件，请执行以下命令：

```
psql>COPY employee (name, email) TO '/home/jason/sqldata/employee.csv' CSV HEADER;
```

如上所述指定`HEADER`子句会导致检索到的列名在第一行中作为列标题列出。例如，执行此命令并在 Microsoft Excel 中打开`employee.csv`文件会产生类似于图 37-1 所示的输出。

**图 37-1.** *在 Microsoft Excel 中查看 employee.csv 文件*