# SQL 不区分大小写

从 `images` 表中检索所有记录的查询如下所示：

`SELECT * FROM images`

大写单词是 SQL 关键字。这纯粹是一种约定。以下写法都是同样正确的：

`SELECT * FROM images`

`select * from images`

`SeLEcT * fRoM images`

尽管 SQL 关键字不区分大小写，但数据库列名并非如此。使用大写关键字的优点是能让 SQL 查询更易于阅读。您可以自由选择最适合自己的风格，但最后一个例子中的“勒索信”风格最好避免。