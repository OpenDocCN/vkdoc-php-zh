# 下一步怎么走？

本书涵盖了海量的知识点。如果你已经掌握了这里介绍的所有技巧，那么你正在迈向中级 PHP 开发者的道路上，再稍加努力，就能进入高级阶段。如果你觉得学起来有些吃力，别担心。重新翻阅前面的章节吧。你练习得越多，它就会变得越简单。

你可能会想：“我怎么可能记住所有这些内容？”其实不必如此。查资料没什么好丢脸的。请把 PHP 在线手册（[`http://php.net/manual/en/`](http://php.net/manual/en/)）加入书签并经常使用。它会持续更新，并且包含大量有用的示例。在每个页面右上角的搜索框中输入函数名称，就能直接跳转到该函数的完整说明。即使你记不住准确的函数名，手册也会带你到一个页面，推荐最可能的候选函数。大多数页面都提供了实际示例，展示该函数或类是如何使用的。

让动态网页设计变得简单的，并非是对 PHP 函数和类百科全书式的记忆，而是扎实掌握条件语句、循环以及其他结构如何控制脚本流程。一旦你能以“如果发生这种情况，接下来应该发生什么？”的思维来构思你的项目，你就成了自己游戏的主宰。我经常查阅 PHP 在线手册。对我来说，它就像一本字典。大多数时候，我只是想确认参数的顺序是否正确，但经常会有某个内容吸引我的注意，为我打开新的视野。我可能不会立刻使用那个知识，但会把它记在脑海深处以备将来之用，并在需要核实细节时再次查阅。

MySQL 在线手册（[`http://dev.mysql.com/doc/refman/5.6/en/index.html`](http://dev.mysql.com/doc/refman/5.6/en/index.html)）同样实用。MariaDB 的文档在 [`https://mariadb.com/kb/en/`](https://mariadb.com/kb/en/)。让 PHP 和数据库在线手册成为你的朋友，你的知识就会突飞猛进。

### 索引

#### A

- `AES_DECRYPT()` 函数
- `AES_ENCRYPT()` 函数
- Apache 网页服务器
- `authenticate_mysqli.php`
- `authenticate_pdo.php`

#### B

- `basename()` 函数
- `bindParam()` 方法
- 布尔值

#### C

- 层叠样式表 (CSS)
- 类
    - `allowAllTypes()` 方法
    - 避免命名冲突
    - 类型转换运算符
    - `checkFile()` 函数
    - 条件语句
    - 构造函数
    - 危险文件
        - `allowAllTypes()` 方法
        - `checkFile()` 方法
        - `checkName()` 方法
        - `moveFile()` 方法
        - `pathinfo()` 函数
        - `str_replace()` 函数
        - `Upload.php` 及测试文件上传
    - 定义
    - `$deleteOriginal` 属性
    - 目标文件夹
    - `DOCTYPE` 声明
    - `extends` 关键字
    - `file_upload.php`
    - `$loader` 对象
    - `$messages`
    - `moveFile()` 方法
    - `move_uploaded_file()` 函数
    - `namespace` 关键字
    - `$permitted` 属性
        - `allowAllTypes()` 方法
        - `checkFile()` 方法
        - `checkType()` 方法
        - `is_numeric()` 函数
    - `$max` 属性
    - `setMaxSize()` 方法
    - `upload_test` 文件夹
    - `PhpSolutions/File` 文件夹
    - 属性
        - `protected` 关键字
        - `public` 关键字
    - 公有方法
    - 重命名重复文件
    - 创建 setter 方法
    - `$thumbDestination` 属性
    - 创建 `ThumbnailUpload.php`
    - `try/catch` 代码块
    - 上传错误
        - `checkFile()` 方法
        - `$_FILES` 数组
        - `getErrorMessage()` 方法
        - `getMaxSize()` 方法
        - `MIME` 类型
        - `number_format()` 函数
        - `upload()` 方法
- 逗号分隔值 (CSV) 文件
    - `array_combine()` 函数
    - 条件语句
    - `fgetcsv()` 函数
    - `print_r()`
    - 私有文件夹
- 内容管理系统
    - blog 表
        - 创建内容
        - 插入和更新记录
        - 删除记录
        - “编辑”和“删除”链接
        - 插入新记录
            - 使用 `MySQLi`
            - 使用 `PDO`
        - 更新记录
            - 使用 `MySQLi`
            - 使用 `PDO`
        - SQL `UPDATE` 命令
- `count()` 函数
- `count()` 方法
- `createThumbnail()` 方法

#### D

- 数据库设计
    - 兼容性
    - MySQL 优势
    - 备份数据
    - 二进制数据
    - 大小写敏感性
    - 数据类型
    - 日期和时间
    - 插入记录
    - 命名规则
    - 数字列类型
    - 预定义列表
    - 创建表
    - 创建用户账户
        - `phpMyAdmin`
    - 存储信息
    - 分解信息
    - 检查点
    - 主键
- 数据源名称 (DSN)
- `date()` 函数
- 日期和时间格式化
    - `add()` 和 `sub()` 方法
    - 32 位处理器
    - `createFromDateString()` 方法
    - `createFromFormat()` 方法
    - `DateInterval` 和 `DatePeriod` 类
    - `DateInterval` 静态方法
    - `DateTime` 方法
    - 创建 `DateTime` 对象
    - `DateTimeZone` 方法
    - `DateTimeZone` 对象
    - `diff()` 方法
    - `format()` 方法
    - `getTimezone()` 方法
    - `.htaccess`/`user.ini` 文件
    - `modify()` 方法
    - MySQL 优势
        - `blog_list_mysqli.php` 或 `blog_list_pdo.php`
    - `checkdate()` 函数
    - 条件语句
    - `convertDateToISO()` 函数
    - `DATE_ADD()` 和 `DATE_SUB()` 方法
    - `date_converter.php`
    - 下拉菜单
    - `explode()` 函数
    - 月、日、年
    - `NOW()` 函数
    - `<option>` 标签
    - `SELECT` 查询
    - `sprintf()` 函数
    - 时间戳
    - `utility_funcs.php`
    - MySQL `date()` 函数
    - 期间指示符
    - `setDate()` 和 `setTime()` 方法
    - `setTimezone()` 方法
    - 静态方法
    - `strftime()` 方法
    - 时区指令
    - `DateTimeImmutable` 类
- `define()` 函数
- `DELETE` 查询
- `doubleIt()` 函数
- 数据导出
- 动态元素
    - 创建多列表格
    - 持久数据库连接

- 递增运算符 (`++`)
- `MySQLi` 版本
- 缩略图查询字符串
- 记录子集
- 条件语句
- `fetch_row()` 方法
- `$_GET` 数组
- `LIMIT` 子句
- 导航
- 创建导航链接
- 选择
- `SHOWMAX`
- `$startRow`

#### E

- `echo` 命令
- 电子邮件
    - 邮件头注入
    - 邮件头
    - 连接运算符 (`.=`)
    - `contact.php`
    - `Content-Type`
    - `filter_input()` 函数
    - `From`/`Reply-To`
    - `mail()` 函数
    - 消息体编码实现
    - `header()` 函数
    - `htmlentities()` 函数
    - `implode()` 函数
    - `processmail.php`
    - `str_replace()` 函数
    - 验证
    - `wordwrap()` 函数
    - `processmail.php` 故障排除
- 加密方法
    - 单向加密 (参见 单向加密)
    - 双向加密 (参见 双向加密)
- `errorCode()` 方法
- `errorInfo()` 方法
- `exec()` 方法
- `execute()` 方法

#### F

- `fclose()` 函数
- PHP 反馈表单
    - 复选框组
        - `count()` 函数
        - `in_array()` 函数
        - `name` 属性的使用
    - 编码实现
    - 下拉选项菜单
    - 邮件头 (参见 邮件头)
    - `get` 方法
    - `isset()` 函数
    - Japan Journey 网站
    - 多选列表
    - `post` 方法
    - `$_POST` 超全局数组
    - 处理和验证用户输入 (参见 处理和验证用户输入)
    - 单选按钮组
    - 单个复选框
- `fetch_assoc()` 方法
- `fetch()` 方法
- `fgetcsv()` 函数
- `fgets()` 函数
- 基于文件的身份验证
    - `fgetcsv()` 函数
    - `header()` 函数
    - 登录页面
    - 创建登出按钮
    - 限制访问
    - `$_FILES` 数组
    - 多文件上传
    - `name` 属性
    - `upload()` 方法
- 文件系统
    - `FilesystemIterator` 类
    - `new` 关键字
    - `RecursiveDirectoryIterator` 类
    - `setFlags()` 方法
    - `SplFileInfo` 方法
    - `RegexIterator` csv
    - 文件下拉菜单
    - PDF 和 Word 文档
    - `scandir()` 函数
- `FilesystemIterator` 类
    - `new` 关键字
    - `RecursiveDirectoryIterator` 类
    - `setFlags()` 方法
    - `SplFileInfo` 方法
- 文件上传类 (参见 类)
    - 配置设置
    - 目录上传
        - Mac OS X
        - Windows
    - `$_FILES` 数组 (另见 `$_FILES` 数组)
    - 图像元素
    - `isset()` 函数
    - `</form>` 标签
    - 远程服务器
    - `<form>` 标签
    - `move_uploaded_file()` 函数
    - `DOCTYPE` 声明
    - `</form>` 和 `</body>` 标签
    - `$max` 变量
    - `upload_test` 文件夹
    - 安全要点
- `flock()` 函数
- `fopen()` 函数
- 外键约束
    - 定义
    - 删除脚本
        - `CASCADE`
        - `errorCode()` 方法
        - `MySQLi`
        - `ON DELETE`
        - `PDO` 预处理语句
        - `SET NULL`
        - `InnoDB` 存储引擎
    - 设置 `CASCADE`
    - categories 表
    - 错误消息
    - `NO ACTION`
    - `关系视图`
    - `RESTRICT`
    - `SET NULL`
- `Format()` 方法
- `free_result()` 方法
- `fseek()` 函数
- `ftruncate()` 函数
- `fwrite()` 函数

#### G

- `getimagesize()` 函数

#### H

- `header()` 函数
- 超文本预处理器 (PHP)
    - 优势
    - Apache 网页服务器
    - 关联数组
    - `basename()` 函数
    - 条件语句
    - 配置设置
    - 自定义函数/类
    - 数据存储
    - `date()` 函数
    - `display_errors` 控制
    - `display_errors` 指令
    - 文档相对链接
    - 文档相对路径
    - 双扩展名
    - 双引号字符串
    - 创建下载链接
    - “编辑”和“删除”链接
    - 编辑 `php.ini`
    - 错误控制运算符
    - 错误消息
        - 复制并粘贴
        - 安全目的
    - 存在性变量
    - 特性
    - `file_exists()` 和 `is_readable()`
    - 文件扩展名
    - `footer.php` 和 `load index.php`
    - 函数/类定义
    - `get_filename.php`
    - `header()` 函数
    - `</head>` 标签
    - `if else` 语句
    - `include_once` 命令
    - `include_path` 命令
    - `include_path` 指令
    - 独立程序
    - 插入新记录
        - 使用 `MySQLi`
        - 使用 `PDO`
    - 安装
    - 本地测试环境
    - 位置
    - `MAMP` 安装
    - `menu.php` 和 `load index.php`
    - 多维数组
    - 嵌套函数
    - `ob_start()` 函数
    - 概述
    - `phpinfo()` 命令
    - `phpsols` 随机图片
    - `random_image.php`
    - `random_image.php` 和 `index.php`
    - 读写文件 (参见 读写文件)
    - 重定向错误页面
    - `require_once` 命令
    - 脚本编辑器特性
    - 安全性
    - 安全风险
    - 服务器端包含
    - `set_include_path()`
    - 站点根目录相对路径
    - `index.php` 的静态版本
    - `str_replace()` 函数
    - 超全局数组
    - 测试和配置 MAMP
        - Apache 和 MySQL 端口
        - 控制面板
    - 测试与安全
    - `ucfirst()` 函数
    - `ucwords()` 函数
    - 丑陋的间隙
    - 更新记录
        - 使用 `MySQLi`
        - 使用 `PDO`
    - `url_include` 指令
    - URLs
    - 虚拟主机
    - 网页优势
        - 维护
        - 图形化表示
    - 网页服务器的 `include_path`


## 网站支持

## 魔术引号

## 富文本格式 (RTF)

## 安全模式

### I

- `implode()` 函数

- `InnoDB` 存储引擎

    - `phpMyAdmin`

    - 远程服务器

    - `storage_engines.php`

    - 表选项

- 互联网信息服务 (IIS)

- 互联网服务提供商 (ISPs)

- `is_numeric()` 函数

- `isset()` 函数

### J, K

- JavaScript 对象表示法 (JSON)

### L

- `lastInsertId()` 方法

- 链接表

### M, N

- 邮件传输代理 (MTA)

- MIME 类型

- 多页表单

- 多数据库表

    - 交叉引用表

    - 条件语句

    - `行内` 注释

    - `<option>` 标签

    - `$selected_categories`

    - 参照完整性 (参见 参照完整性)

- 多表

    - 交叉引用表

    - 构建详情页

    - 文章和图片

    - `convertToParas()` 函数

    - 数据库连接文件

    - `DATE_FORMAT()` 函数

    - `<figure>`

    - `<h2>` 标签

    - `NULL`

    - 占位图片和文本

    - 只读连接

    - `SELECT` 查询

    - SQL 查询

        - 修改现有表

        - 添加额外列

        - 插入外键

        - 下拉菜单

        - `MySQLi`

        - `PDO`

    - 原则

    - 图片和博客

    - `INNER JOIN`

    - 智能链接创建

    - 多对多

    - 一对多

    - 一对一

    - 主键和外键

    - 主表/父表

    - 查找记录

    - 参照完整性

    - 从表/子表

    - `WHERE` 子句

- MySQL 优势

    - 备份数据

    - 二进制数据

    - 大小写敏感性

    - 数据和时间

    - 数据类型

    - 插入记录

    - 命名规则

    - 数字列类型

    - 预定义列表

    - 远程服务器设置

    - 创建表

    - 创建用户账户

- MySQL 改进版 (MySQLi) 扩展

    - 连接方式

    - 连接和数据库查询

    - `mysql_02.php`

    - 预处理语句

        - `bind_param()` 方法

        - `bind_result()` 方法

        - `close()` 方法

        - 条件语句

        - `DOCTYPE` 声明

        - `execute()` 方法

        - `fetch_assoc()` 方法

        - `fetch()` 方法

        - `prepare()` 方法

        - `stmt_init()` 方法

        - `store_result()` 方法

    - 远程服务器设置

    - 结果集

    - 可复用数据库连接器

        - `connection.php`

        - `connection_test.php`

        - `exit()` 函数

    - 故障排除

    - `SELECT` 查询

### O

- `ob_end_clean()` 函数

- `ob_end_flush()` 函数

- 面向对象编程 (OOP)

- 单向加密

    - 优点与缺点

    - 身份验证

        - `DOCTYPE` 声明

        - `MySQLi`

        - `PDO`

        - `$_SESSION`

        - 创建新表

        - `password_verify()` 函数的使用

    - 用户注册表单

        - `bindParam()` 方法

        - 条件语句

        - `DOCTYPE` 声明

        - `errorInfo()` 方法

        - `$errors` 数组

        - `MySQLi`

        - `PhpSolutions/Authenticate` 文件夹

        - `register_db.php`

        - `register_user_mysqli.php`

        - `rowCount()` 方法

    - P (此处原文如此，推测为过渡或章节标记)

### P

- 参数绑定

- 密码安全

    - `check()` 方法

    - `CheckPassword` 类

    - 条件语句

    - `DOCTYPE` 声明

    - 加密

    - `$errors` 数组

    - `filesize()` 函数

    - `fopen()` 模式

    - `getErrors()` 方法

    - `password_compat` 库

    - `password_hash()` 函数

    - `password_verify()` 函数

    - `preg_match()` 方法

    - `requireMixedCase()` 方法

    - `trim()` 方法

    - 用户注册系统

- `password_verify()` 函数

- Perl 兼容正则表达式 (PCRE)

- 相册

    - CSS 样式规则

    - 显示图片

    - `DOCTYPE` 声明

    - `getimagesize()` 函数

    - 使用 `MySQLi` 的表格单元格

    - 动态元素 (参见 动态元素)

    - `gallery_01.php` 编码

    - 数据库中存储图片

    - 创建缩略图

- PHP。参见 超文本预处理器 (PHP)

- PHP 数据对象 (PDO)

    - 修改列

    - 连接方式

    - 连接和数据库查询

    - `COUNT()` 函数

    - 预处理语句

        - 匿名占位符

        - `bindColumn()` 方法

        - `bindParam()`

        - `bindValue()`

        - `DOCTYPE` 声明

        - `execute()` 方法

        - `fetch()` 方法

        - 命名占位符

        - `prepare()` 方法

    - 远程服务器设置

    - 结果集

    - `rowCount()` 函数

    - `SELECT` 查询

- `phpinfo()` 函数

- PHP 脚本

    - 算术运算符

    - 数组定义

    - 关联数组

    - 黑盒

    - 布尔值

    - 复合算术赋值运算符

    - 复合连接运算符

    - 命令/语句

    - 注释变量

    - 比较运算符

    - 复数

    - 条件语句

    - 自定义函数

    - 数据类型

    - `DateTime` 类

    - `DateTimeZone` 类

    - `DateTimeZone` 对象

    - `doubleIt()` 函数

    - 双引号字符串

    - `echo` 快捷语法

    - 嵌入式语言

    - 空数组

    - 错误消息报告

    - 错误处理代码页面空白

    - 转义序列

    - 函数

    - 折中方案/可能情况

    - heredoc 语法

    - 隐式布尔值

    - 缩进代码

    - 索引数组

    - 检查数组

    - 逻辑运算符

    - 循环

    - 多维数组

    - 多行注释

    - nowdoc 语法

    - 面向对象编程

    - 传递参数

    - 传递引用参数

    - 传值

    - 函数

    - 属性和方法

    - 快速检查清单

    - 服务器端语言

    - 单行注释

    - 单引号字符串

    - 存储

    - 超全局数组

    - `switch` 语句

    - 临时变量

    - 三元运算符


- 印刷错误

- 变量和函数

    - 赋值运算符

    - 变量命名

- 通用循环

- 网站

- `while` 和 `do while`

- 处理和验证用户输入

    - 拦截邮件

    - 条件语句

    - `contact.php`

    - `foreach` 循环

    - `isSuspect()` 函数

    - 客户端验证

    - `mail()` 函数

    - 可复用脚本

        - 条件语句

        - `empty()` 函数

        - `$errors` 和 `$missing`

        - `foreach` 循环

        - `is_array()` 函数

        - `isset()` 函数

        - 标签

        - `$missing` 和 `$errors` PHP 条件语句

        - `processmail.php`

    - 自处理表单

    - 创建粘性表单字段

        - 评论文本区域

        - `echo` 命令

        - `htmlentities()` 函数

#### Q

- `query()` 方法

#### R

- `rand()` 函数

- `Random_image.php` 和 `index.php`

- `readfile()` 函数

- 读写文件

    - 追加模式

    - CSV 文件 (另见 逗号分隔值 (CSV) 文件)

    - `fclose()` 函数

    - `flock()` 函数

    - `fopen()` 函数

    - `fseek()` 函数

    - `ftruncate()` 函数

    - 函数内部指针

    - 模式

    - 覆盖现有文件

    - 文本文件

        - `array_slice()` 函数

        - `echo` 命令

        - `explode()` 函数

        - `file()` 函数

        - `file_get_contents()` 函数

        - 文件系统

        - `nl2br()` 函数

- 参照完整性

    - `blog_insert_mysqli.php`

    - 类型转换运算符

    - `$catError`

    - 条件语句

    - `$_FILES`

    - 外键

    - `getFilenames()` 方法

    - `getMessages()` 方法

    - `$_POST` 数组

    - `upload()` 方法

    - 分类和图片

        - `blog_insert_mysqli.php`

        - 复选框

        - 条件语句

        - `<form>` 标签

        - 多选 `<select>` 列表

        - `toggle_fields.js`

    - 交叉引用表

    - 文件上传

    - 外键约束 (参见 外键约束)

    - `InnoDB` 存储引擎

    - `INSERT` 查询

    - `PDO` 版本

    - 主键

- 远程文件访问

- RSS 订阅

    - `<script>` 标签/超链接

    - SimpleXML (参见 SimpleXML)

    - `strip_tags()` 函数

- 富文本格式 (RTF)

- `round()` 函数

- `rowCount()` 方法

- RSS 订阅

    - S

#### S

- `sayHi()` 函数

- `scandir()` 函数

- 安全套接层 (SSL) 连接

- 服务器端包含 (SSI)

- 会话

    - 创建

    - 销毁

    - 基于文件的身份验证

        - `fgetcsv()` 函数

        - `header()` 函数

        - 登录页面

        - 创建登出按钮

        - 限制访问

    - 已发送的报头

    - `ini_set()` 函数

    - 多页表单

    - 密码安全

        - `check()` 方法

        - `CheckPassword` 类

        - 条件语句

        - `DOCTYPE` 声明

        - 加密

        - `$errors` 数组

        - `filesize()` 函数

        - `fopen()` 模式

        - `getErrors()` 方法

        - `password_compat` 库

        - `password_hash()` 函数

        - `password_verify()` 函数

        - `preg_match()` 方法

        - `requireMixedCase()` 方法

        - `trim()` 方法

        - 用户注册系统

    - `PHPSESSID`

    - 重新生成会话 ID

    - 限制访问

        - `DOCTYPE` 声明

        - `ob_start()`

        - `session_destroy()`

        - `setcookie()`

    - 存储变量

    - `time()` 函数

- `setTimezone()` 方法

- `SHOWMAX`

- SimpleXML

    - `foreach` 循环

    - `<item>` 元素

    - RSS 新闻订阅

    - BBC 世界新闻

        - `DateTime` `format()` 方法

        - `foreach` 循环

        - `LimitIterator` 类

        - `LimitIterator` 构造函数

        - `newsfeed.css`

        - `setTimezone()` 方法

- SQL 命令

    - `DELETE`

    - `INSERT`

    - `SELECT`

    - `UPDATE`

- SQL 注入

    - 定义

    - 注入

    - 条件语句

    - `</form>` 标签

    - `MySQLi` 版本

    - `<option>` 标签

    - `PDO` 版本

    - `rowCount()` 方法

    - 预处理语句

    - `real_escape_string()`, MySQLi

    - 策略

- 标准 PHP 库 (SPL)。参见 `FilesystemIterator` 类

- `str_replace()` 函数

- `strtolower()` 函数

- 结构化查询语言 (SQL)

    - 不区分大小写

    - 数字

    - `SELECT` 关键字

    - `LIKE` 关键字

    - `ORDER BY` 子句

    - 选择特定列

    - `WHERE` 子句

    - 字符串

    - 空白字符

- `substr()` 函数

- `switch()` 函数

#### T

- 文本提取

    - `<br/>` 标签方法

    - 条件语句

    - `DATE_FORMAT()` 方法

    - `getFirst()` 函数

    - HTML 记录存储

    - `implode()` 函数

    - `LEFT()` 方法

    - 段落提取

    - PHP `substr()` 方法

    - `preg_split()` 函数

    - `<p>` 标签方法

    - `strrpos()` 和 `substr()` 方法

- 缩略图

    - `basename()` 函数

    - `<body>` 标签

    - `checkType()` 方法

    - 类 (另见 类)

    - `create()` 方法

    - `createThumbnail()` 方法

    - 目标文件夹

    - 尺寸计算

    - `imagecopyresampled()`

    - Mac OS X

    - MIME 类型

    - `$canProcess` 布尔值

    - `checkType()` 方法

- `DOCTYPE` 声明
- `getimagesize()`
- 图像选择
- `$maxSize` 属性
- `PhpSolutions` 文件夹
- `test()` 方法
- 服务器的能力
- 创建 setter 方法
- 使用 GD 函数
- 会话时间限制
- 双向加密
    - 优点与缺点
    - `AES_DECRYPT()` 函数
    - `AES_ENCRYPT()` 函数
    - 使用方式
    - 用户身份验证
        - 使用 `MySQLi`
        - 使用 `PDO`
    - `users_2way` 表
- 排版约定

#### U

- `ucfirst()` 函数
- `ucwords()` 函数
- `UNIQUE` 索引
- `UPDATE` 查询
- URL 编码

#### V, W, X, Y, Z

- 验证用户输入。参见 处理和验证用户输入