# 第 12 章 ■ 日期与时间

- `year`：年份的四位数字表示，例如`2005`。

- `0`：自 Unix 纪元以来的秒数。虽然范围取决于系统，但在基于 Unix 的系统上，通常为`–2147483648`到`2147483647`，而在 Windows 上，范围为`0`到`2147483648`。

**注意**：Windows 操作系统不支持负时间戳值，因此在此系统上使用此函数可解析的最早日期是 1970 年 1 月 1 日午夜。

考虑时间戳`1114284300`（2005 年 4 月 23 日 15:25:00 EDT）。将其传递给`getdate()`并查看数组元素：

```
Array (
[seconds] => 0
[minutes] => 25
[hours] => 15
[mday] => 23
[wday] => 6
[mon] => 4
[year] => 2005
[yday] => 112
[weekday] => Saturday
[month] => April
[0] => 1114284300
)
```

##### `gettimeofday()`

`mixed gettimeofday ([bool $return_float])`

`gettimeofday()`函数返回一个关联数组，包含当前时间的各项元素。对于运行 PHP 5.1.0 及更新版本的用户，可选参数`$return_float`会使`gettimeofday()`返回一个浮点值形式的当前时间。总共返回四个元素，包括：

- `dsttime`：指示所使用的夏令时算法，该算法根据地理位置而变化。共有 11 个可能的值，包括`0`（未实行夏令时）、`1`（美国）、`2`（澳大利亚）、`3`（西欧）、`4`（中欧）、`5`（东欧）、`6`（加拿大）、`7`（英国和爱尔兰）、`8`（罗马尼亚）、`9`（土耳其）和`10`（澳大利亚 1986 年变体）。

- `minuteswest`：格林威治标准时间（GMT）以西的分钟数。

- `sec`：自 Unix 纪元以来的秒数。

- `usec`：当时间分数超过整秒值时的微秒数。

在 2005 年 4 月 23 日 16:24:55 EDT 从测试服务器执行`gettimeofday()`会产生以下输出：

```
Array (
[sec] => 1114287896
[usec] => 110683
[minuteswest] => 300
[dsttime] => 1
)
```

当然，可以将输出赋值给一个数组，然后根据需要引用每个元素：

```php
$time = gettimeofday();
$GMToffset = $time['minuteswest'] / 60;
echo "Server location is $GMToffset hours west of GMT.";
```

这将返回以下结果：

```
Server location is 5 hours west of GMT.
```

##### `mktime()`

`int mktime ([int $hour [, int $minute [, int $second [, int $month [, int $day [, int $year [, int $is_dst]]]]]]])`

`mktime()`函数用于生成一个时间戳（以秒为单位），表示 Unix 纪元与给定日期和时间之间的差。除了`$is_dst`可能不太明显，每个可选参数的目的都显而易见：如果夏令时生效，则设置为`1`；否则设置为`0`；如果不确定，则设置为`-1`（默认值）。默认值会提示 PHP 尝试确定夏令时是否生效。

例如，如果您想知道 2005 年 4 月 27 日晚上 8:50 的时间戳，只需填入适当的值：

```php
echo mktime(20,50,00,4,27,2005);
```

这将返回以下结果：

这对于计算两个时间点之间的差值特别有用。例如，从当前时间到 2006 年 4 月 15 日午夜（美国下一个主要纳税日）还有多少小时？

```php
$now = mktime();
$taxday = mktime(0,0,0,4,15,2006);
// 差值（秒）
$difference = $taxday - $now;
// 计算总小时数
$hours = round($difference / 60 / 60);
echo "Only $hours hours until tax day!";
```

这将返回以下结果：

```
Only 8451 hours until tax day!
```

##### `time()`

`int time()`

`time()`函数用于检索当前的 Unix 时间戳。以下示例于 2005 年 4 月 23 日 15:25:00 EDT 执行：

```php
echo time();
```

这将产生以下结果：

使用之前介绍的`date()`函数，可以将此时间戳转换回人类可读的日期：

```php
echo date("F d, Y h:i:s", 1114284300);
```

这将返回以下结果：

```
April 23, 2005 03:25:00
```

如果您想要将特定的日期/时间值转换为对应的时间戳，请参阅上一节关于`mktime()`的内容。

[www.it-ebooks.info](http://www.it-ebooks.info/)