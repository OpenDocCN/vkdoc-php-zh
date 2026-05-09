# PHP 日期和时间处理

虽然 PHP 的 `checkdate()` 函数对于验证日期很有用，但它要求提供所有三个日期组成部分（月、日和年）。但如果你只想验证一个日期组成部分，例如月份，该怎么办？或者，你可能希望在将时间值（时:分:秒）或其特定部分插入数据库之前，确保其是合法的。`Calendar` 包提供了几种方法来验证日期和时间，或其任意部分。本列表介绍了这些方法：

- `isValid()`: 执行所有其他时间和日期验证方法，验证日期和时间
- `isValidDay()`: 确保日期在 1 到 31 之间
- `isValidHour()`: 确保值在 0 到 23 之间
- `isValidMinute()`: 确保值在 0 到 59 之间
- `isValidMonth()`: 确保值在 1 到 12 之间
- `isValidSecond()`: 确保值在 0 到 59 之间
- `isValidYear()`: 确保值在 Unix 系统上为 1902 到 2037，在 Windows 系统上为 1970 到 2037

#### PHP 5.1

虽然本章前面讨论的内置日期函数非常有用，但那些希望操作和导航日期的用户被冷落了。例如，没有一个现成的函数可以用来确定星期几之后的日期、十一月之后是哪个月份，或者给定年份是否为闰年。虽然上一节介绍的 `Calendar` 包提供了这些功能，但如果能通过默认分发版提供这些增强功能就更好了。那些长期渴望此类功能的人有福了，因为 PECL3 的日期和时间扩展已自版本 5.1 起被纳入标准 PHP 分发版。由 Pierre-Alain Joye 编写的日期和时间库（以下简称 `Date`）必将使许多 PHP 程序员的工作变得轻松许多。在本节中，你将了解 `Date`，并通过几个示例看到其强大的功能。

> **注意** 本章是在 PHP 5.1 正式发布前几个月编写的，当时 `Date` 扩展还没有可用的文档。因此，请预先注意，当你阅读本节时，其中的任何信息可能确实不正确。本节也未提供所有可用功能的全面总结，因为在撰写本文时，有几个方法无法正常工作，因此决定最好从材料中省略它们。这便是为了保持技术前沿而必须承担的风险！

#### Date 基础

在本章前面，半开玩笑地提到提供 `date()` 示例只是为了演示，因为为了记住那些有点莫名其妙参数的作用，你仍然需要多年来查阅文档（或本书）。`Date` 消除了很多猜测工作，因为它是完全面向对象的，这意味着处理日期的过程相当自然，因为方法名称是不言自明的。

例如，要设置月份，可以调用 `setMonth()` 设值方法；要获取年份，可以调用 `getYear()` 获取方法；依此类推。本章的其余部分将专门介绍此类及其众多方法。

> **注意** 由于 `Date` 依赖于 5.0 版本中可用的面向对象特性，因此不能将 `Date` 与任何更早版本一起使用。如果你尚未升级到 5.1 版本（但正在使用 5.0.X 版本）并想使用 `Date`，请从 `http://pecl.php.net/package/date_time` 下载。

#### Date 构造函数

在你使用 `Date` 功能之前，需要通过其类构造函数实例化一个日期对象。本节将介绍此构造函数。

3. PECL 是 PHP 扩展社区库，包含用 C 语言编写的 PHP 扩展。



##### `date()`

`object date ([integer *day* [, integer *month* [, integer *year* [, integer *weekstart*]]]])`

`date()` 方法是类的构造函数。你可以在实例化时通过 `day`、`month` 和 `year` 参数设置日期，也可以在之后使用各种修改器（setters）进行设置，这些修改器将在下文中介绍。要创建一个空的日期对象，只需像这样调用 `date()`：`$date = new Date();`

要创建对象并将日期设置为 2005 年 4 月 29 日，请执行：

```
$date = new Date(29,4,2005);
```

你可以使用可选的 `weekstart` 参数来告知对象应将一周中的哪一天视为第一天。默认情况下，日期对象假定一周从星期一开始，也就是说星期一的偏移量为 1。

奇怪的是，并没有一种便捷的方法将日期对象设置为当前日期。要这样做，你需要使用 `date()` 函数：

```
$date = new Date(date("j"),date("n"),date("Y"));
```

#### 访问器和修改器

`Date` 提供了几个访问器（getters）和修改器（setters），它们对于操作和检索日期组件值非常有用。本节将介绍这些方法。

##### `setDMY()`

`boolean setDMY (integer *day*, integer *month*, integer *year*)`

`setDMY()` 方法用于设置日期对象的日、月、年，成功时返回 `TRUE`，否则返回 `FALSE`。让我们将日期设置为 2005 年 4 月 29 日：

```
$date = new Date();
$date->setDMY(29,4,2005);
$dcs = $date->getArray();
print_r($dcs);
```

这将返回以下内容：

```
Array (
    [day] => 29 [month] => 4 [year] => 2005
    [hour] => 0 [min] => 0 [sec] => 0
)
```

`getArray()` 方法可以方便地将所有三个日期组件存储到一个数组中。接下来介绍这个方法。

##### `getArray()`

`array getArray()`

`getArray()` 方法返回一个包含三个键的关联数组：`day`、`month` 和 `year`：

```
$date = new Date();
$date->setDMY(29,4,2005);
$dcs = $date->getArray();
echo "The month: ".$dcs['month']."<br />";
echo "The day: ".$dcs['day']."<br />";
echo "The year: ".$dcs['year']."<br />";
```

结果如下：

```
The month: 4
The day: 29
The year: 2005
```

##### `setDay()`

`boolean setDay (integer *day*)`

`setDay()` 方法将日期对象的日属性设置为 `day`，成功时返回 `TRUE`，否则返回 `FALSE`。以下示例将日期设置为 2006 年 4 月 29 日，然后将日更改为 15：

```
$date = new Date(29,4,2006);
$date->setDay(15);
// 日期现在设置为 2006 年 4 月 15 日
```

##### `getDay()`

`integer getDay()`

`getDay()` 方法从日期对象返回日属性。示例如下：

```
$date = new Date(29,4,2006);
echo $date->getDay();
```

返回以下内容：

```
29
```

##### `setJulian()`

儒略日由历史学家约瑟夫·斯卡利杰（1540–1609）创建，旨在转换他在研究历史文献时遇到的许多不同的历法系统。它基于一个 7,980 年的周期，因为这个数字是几个常见时间周期（即太阴周、太阳周以及一个罗马税收周期）的倍数，这些周期构成了这些系统的基础。儒略日用从某个特定日期起经过的天数来表示，第一个儒略周期始于公元前 4713 年 1 月 1 日正午（按儒略历）；因此，2006 年 4 月 29 日对应的儒略日为 2453851.5。

> **注意** 儒略日与我们今天使用的、由尤利乌斯·凯撒于公元前 46 年确立的 365 天儒略历系统无关。

##### `getJuliaan()`

`int getJuliaan()`

`getJuliaan()` 方法返回根据日期对象指定的日期计算出的儒略日。有趣的是，截至撰写本文时，儒略日（Julian）被拼写为 `Juliaan`。如果你使用此方法，请务必关注未来的版本，因为未来它很可能会更正为正确的拼写。

##### `setMonth()`

更多信息请访问 http://pecl.php.net。

[www.it-ebooks.info](http://www.it-ebooks.info/)



`boolean setMonth (integer *month*)`

`setMonth()`方法将日期对象的月份属性设置为`month`，成功时返回`TRUE`，否则返回`FALSE`。以下示例将日期设置为 2005 年 4 月 29 日，然后将月份改为七月：

```
$date = new Date(29,4,2005);
$date->setMonth(7);
// 月份现在设置为七月（7）
```

`getMonth()`

`integer getMonth()`

`getMonth()`方法从日期对象返回月份属性。示例如下：

```
$date = new Date(29,4,2005);
echo $date->getMonth();
```

这将返回：

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 12 章 ■ 日期与时间

**293**

`setYear()`

`boolean setYear (integer *year*)`

`setYear()`方法将日期对象的年份属性设置为`year`，成功时返回`TRUE`，否则返回`FALSE`。以下示例将日期设置为 2005 年 4 月 29 日，然后将年份改为 2006：

```
$date = new Date(29,4,2005);
$date->setYear(2006);
// 年份现在设置为 2006
```

`getYear()`

`integer getYear()`

`getYear()`方法从日期对象返回年份属性。示例如下：

```
$date = new Date(29,4,2005);
echo $date->getYear();
```

返回的结果如下：

**验证器**

`Date`提供了用于判断日期是否在闰年以及验证日期正确性的方法。本节将介绍这两个方法。

`isLeap()`

`boolean isLeap()`

`isLeap()`方法如果日期对象表示的年份是闰年则返回`TRUE`，否则返回`FALSE`。以下脚本使用`isLeap()`结合三元运算符告知用户给定年份是否为闰年：

```
$year = 2005;
$date = new Date(date("j"),date("n"),$year);
echo "$year 是". ($date->isLeap() == 1 ? "" : "不"). "闰年。";
```

这将产生如下输出：

```
2005 不是闰年。
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**294**

第 12 章 ■ 日期与时间

`isValid()`

`boolean isValid()`

`isValid()`方法如果日期对象表示的日期有效则返回`TRUE`，否则返回`FALSE`。由于此方法不能静态调用，并且无法通过任何修改器的构造函数设置无效日期，因此目前`isValid()`的存在理由尚不明确。

**操作方法**

当然，此类真正的适用性在于其日期操作能力。本节将介绍可轻松操作日期的函数。

`addDays()`

`boolean addDays (int *days*)`

`addDays()`方法将`days`天添加到日期对象，当新的天数超过当前月份的总天数时调整月份和年份，成功时返回`TRUE`，否则返回`FALSE`。例如，假设对象的日期设置为 2005 年 4 月 28 日，我们使用`addDays()`添加五天：

```
$date = new Date();
$date->setDMY(28,4,2005);
$date->addDays(5);
$dcs = $date->getArray();
print_r($dcs);
```

将返回：

```
Array (
    [day] => 3 [month] => 5 [year] => 2005
    [hour] => 0 [min] => 0 [sec] => 0
)
```

`subDays()`

`boolean subDays (int *days*)`

`subDays()`方法从日期对象减去`days`天，当天数超过日期对象的天数分量时调整月份和年份，成功时返回`TRUE`，否则返回`FALSE`。例如，假设对象的日期设置为 2006 年 4 月 28 日，我们使用`subDays()`减去 14 天：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->subDays(14);
$dcs = $date->getArray();
print_r($dcs);
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 12 章 ■ 日期与时间

**295**

这将返回：

```
Array (
    [day] => 14 [month] => 4 [year] => 2006
    [hour] => 0 [min] => 0 [sec] => 0
)
```

`addMonths()`

`boolean addMonths (int *months*)`

`addMonths()`方法将`months`个月添加到日期对象的月份属性，当新的月份值大于 12 时调整年份，成功时返回`TRUE`，否则返回`FALSE`。例如，假设对象的日期设置为 2006 年 4 月 28 日，我们使用`addMonths()`添加九个月：

```
$date = new Date();
$date->setDMY(28,4,2006);
```



`$date->addMonths(9);`

`$dcs = $date->getArray();`

`print_r($dcs);`

以下是输出结果：

```
Array (
[day] => 28 [month] => 1 [year] => 2007
[hour] => 0 [min] => 0 [sec] => 0
)
```

如果新月份的天数少于`day`属性中的天数，那么`day`将向下调整至新月份的最后一天。

##### `subMonths()`

`boolean subMonths (int *months*)`

`subMonths()`方法从日期对象的`month`属性中减去`$months`，如果新的月份值小于零，则相应调整`year`，成功返回`TRUE`，否则返回`FALSE`。例如，假设对象日期设置为 2006 年 4 月 28 日，我们使用`subMonths()`减去九个月：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->subMonths(9);
$dcs = $date->getArray();
print_r($dcs);
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**296**

**第 12 章 ■ 日期和时间**

返回结果如下：

```
Array (
[day] => 28 [month] => 7 [year] => 2005
[hour] => 0 [min] => 0 [sec] => 0
)
```

如果新月份的天数少于`day`属性中的天数，那么`day`将向下调整至新月份的最后一天。

##### `addWeeks()`

`boolean addWeeks (int *weeks*)`

`addWeeks()`方法向日期对象的日期添加`$weeks`，成功返回`TRUE`，否则返回`FALSE`。例如，假设对象日期设置为 2006 年 4 月 28 日，我们使用`addWeeks()`添加七周：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->addWeeks(7);
$dcs = $date->getArray();
print_r($dcs);
```

返回结果如下：

```
Array (
[day] => 16 [month] => 6 [year] => 2006
[hour] => 0 [min] => 0 [sec] => 0
)
```

##### `subWeeks()`

`boolean subWeeks (int *weeks*)`

`subWeeks()`方法从日期对象的日期中减去`$weeks`，成功返回`TRUE`，否则返回`FALSE`。例如，假设对象日期设置为 2006 年 4 月 28 日，我们使用`subWeeks()`减去七周：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->subWeeks(7);
$dcs = $date->getArray();
print_r($dcs);
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 12 章 ■ 日期和时间**

**297**

返回结果如下：

```
Array (
[day] => 10 [month] => 3 [year] => 2006
[hour] => 0 [min] => 0 [sec] => 0
)
```

##### `addYears()`

`boolean addYears (int *years*)`

`addYears()`方法向日期对象的`year`属性添加`$years`，成功返回`TRUE`，否则返回`FALSE`。例如，假设对象日期设置为 2006 年 4 月 28 日，我们使用`addYears()`添加四年：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->addYears(4);
$dcs = $date->getArray();
print_r($dcs);
```

返回结果如下：

```
Array (
[day] => 28 [month] => 4 [year] => 2010
[hour] => 0 [min] => 0 [sec] => 0
)
```

##### `subYears()`

`boolean subYears (int *years*)`

`subYears()`方法从日期对象的`year`属性中减去`$years`，成功返回`TRUE`，否则返回`FALSE`。例如，假设对象日期设置为 2006 年 4 月 28 日，我们使用`subYears()`减去两年：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->subYears(2);
$dcs = $date->getArray();
print_r($dcs);
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**298**

**第 12 章 ■ 日期和时间**

返回以下输出结果：

```
Array (
[day] => 28 [month] => 4 [year] => 2004
[hour] => 0 [min] => 0 [sec] => 0
)
```

##### `getWeekday()`

`integer getWeekday()`

`getWeekday()`方法返回日期对象指定日期的数值偏移量。示例如下：

```
$date = new Date();
$date->setDMY(30,4,2006);
echo $date->getWeekday();
```

返回结果如下，表示星期日，因为星期日的数值偏移量是 7：`7`

##### `setToWeekday()`

`boolean setToWeekday (int *weekday*, int *n* [, int *month* [, int *year*]])`

`setToWeekday()`方法将日期设置为某年某月的第 n 个星期*weekday*，成功返回`TRUE`，否则返回`FALSE`。如果未提供月份和年份，则使用当前月份和年份。在撰写本文时，此方法存在缺陷；很可能在本书出版时已被修复。

##### `getDayOfYear()`

`integer getDayOfYear()`

`getDayOfYear()`方法返回日期对象指定日期的数值偏移量。示例如下：

```
$date = new Date();
$date->setDMY(4,7,1776);
echo $date->getDayOfYear();
```

结果是：

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 12 章 ■ 日期和时间**

**299**

##### `getWeekOfYear()`

`integer getWeekOfYear()`

`getDayOfYear()`方法返回日期对象指定周数的数值偏移量：

```
$date = new Date();
$date->setDMY(4,7,1776);
echo $date->getWeekOfYear();
```

返回：

##### `getISOWeekOfYear()`

`integer getISOWeekOfYear()`

`getISOWeekOfYear()`方法根据 ISO 8601 规范返回日期对象所代表日期的周数。ISO 8601 规定，一年的第一周是包含第一个星期四的那一周。例如，2005 年的第一天是星期日，但 1 月 2 日至 8 日包含了第一个星期四；因此，1 月 1 日甚至不算在第一周内。你可能觉得这有点奇怪；然而，这个决定几乎是随意的，它只是标准化了确定一年中第一周的方法。让我们通过查询 1 月 4 日所在的周数来实际看看这个解释：

```
$date = new Date();
$date->setDMY(4,1,2005);
echo $date->getISOWeekOfYear();
```

返回结果如下：

那么，既然 1 月 1 日不符合第一周的条件，它落在哪一周呢？你可能会惊讶地发现，ISO 标准实际上认为它是 2004 年的第 53 周：

```
$date = new Date();
$date->setDMY(1,1,2005);
echo $date->getISOWeekOfYear();
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**300**

**第 12 章 ■ 日期和时间**

返回：

##### `setToLastMonthDay()`

`boolean setToLastMonthDay()`

`setToLastMonthDay()`方法将日期对象的`day`属性调整为`month`属性指定月份的最后一天，成功返回`TRUE`，否则返回`FALSE`。示例如下：

```
$date = new Date();
$date->setDMY(1,4,2006);
$date->setToLastMonthDay();
echo $date->getDay();
```

返回以下输出结果：

##### `setFirstDow()`

`boolean setFirstDow()`

`setFirstDow()`方法将日期对象的`day`属性设置为`weekstart`属性指定的一周中的第一天，成功返回`TRUE`，否则返回`FALSE`。默认情况下，`weekstart`设置为星期一。以下示例将日期设置为 2006 年 4 月 28 日（星期五），然后将日期移动到该周的第一天（星期一）：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->setFirstDow();
$dcs = $date->getArray();
print_r($dcs);
```

返回：

```
Array (
[day] => 24 [month] => 4 [year] => 2006
[hour] => 0 [min] => 0 [sec] => 0
)
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 12 章 ■ 日期和时间**

**301**

##### `setLastDow()`

`boolean setLastDow()`

`setLastDow()`方法将日期对象的`day`属性设置为一周中的最后一天，成功返回`TRUE`，否则返回`FALSE`。这一天取决于`weekstart`属性的值，默认设置为星期一。以下示例将日期设置为 2006 年 4 月 28 日（星期五），然后将日期移动到该周的最后一天（星期日）：

```
$date = new Date();
$date->setDMY(28,4,2006);
$date->setLastDow();
$dcs = $date->getArray();
print_r($dcs);
```

返回：

```
Array (
[day] => 30 [month] => 4 [year] => 2006
[hour] => 0 [min] => 0 [sec] => 0
)
```

**摘要**



本章涵盖了大量内容，首先概述了在日常 PHP 编程任务中几乎每天都会出现的几个日期和时间函数。接着，我们深入探讨了“日期功夫”这门古老技艺，学习了如何结合这些函数的功能来执行有用的时间处理任务。我们还介绍了实用的 `Calendar` PEAR 包，了解了如何创建基于网格的日历，以及验证和导航机制。最后，对于那些生活在新兴技术前沿的读者，我们还提供了对 PHP 5.1 新日期处理功能的介绍。

下一章将聚焦于很可能激发你对 PHP 进一步学习兴趣的主题：用户交互。我们将通过表单进行数据处理，演示基本功能和高级主题，例如如何使用多值表单组件和自动化表单生成。你还将学习如何通过创建面包屑导航和自定义 404 消息来方便用户导航。

[www.it-ebooks.info](http://www.it-ebooks.info/)

