# 第 6 章

![图片](img/frontdot.jpg)

## 条件语句

条件语句用于根据不同的条件执行不同的代码块。

## If 语句

只有当括号内的条件计算结果为 `true` 时，`if` 语句才会执行。该条件可以包含任何比较运算符和逻辑运算符。在 PHP 中，条件不必是布尔表达式。

```php
if ($x == 1) {
  echo "x is 1";
}
```

为了测试其他条件，`if` 语句可以通过任意数量的 `elseif` 子句进行扩展。只有当之前的所有条件均为假时，才会测试每个额外的条件。

```php
elseif ($x == 2) {
  echo "x is 2";
}
```

`if` 语句最后可以有一个 `else` 子句，如果之前的所有条件均为假，则会执行该子句。

```php
else {
  echo "x is something else";
}
```

至于花括号，如果只需要有条件地执行单个语句，则可以省略它们。

```php
if ($x == 1)
  echo "x is 1";
elseif ($x == 2)
  echo "x is 2";
else
  echo "x is something else";
```

## Switch 语句

`switch` 语句检查一个整数、浮点数或字符串与一系列 `case` 标签之间的相等性。然后将执行流程传递给匹配的 case。该语句可以包含任意数量的 `case` 子句，并且可以以一个 `default` 标签结尾，用于处理所有其他情况。

```php
switch ($x)
{
  case 1: echo "x is 1"; break;
  case 2: echo "x is 2";  break;
  default: echo "x is something else";
}
```

请注意，每个 `case` 标签后的语句不需要用花括号括起来。相反，语句以 `break` 关键字结束，以便跳出 switch。如果没有 `break`，执行流程将会"穿透"到下一个 case。如果多个 case 需要以相同方式求值，这会很有用。

## 替代语法

PHP 为条件语句提供了替代语法。在这种语法中，`if` 语句的开始花括号被替换为冒号，结束花括号被移除，最后一个结束花括号被 `endif` 关键字替换。

```php
if ($x == 1):     echo "x is 1";
elseif ($x == 2): echo "x is 2";
else:             echo "x is something else";
endif;
```

类似地，`switch` 语句也有替代语法，它使用 `endswitch` 关键字来终止该语句。

```php
switch ($x):
  case 1:  echo "x is 1"; break;
  case 2:  echo "x is 2";  break;
  default: echo "x is something else";
endswitch;
```

对于较长的条件语句，替代语法通常更可取，因为这样可以更容易地看出这些语句在哪里结束。

## 混合模式

在代码块的中间可以切换回 HTML 模式。这提供了另一种编写向网页输出文本的条件语句的方式。

```php
<?php if ($x == 1) { ?>
  This will show if $x is 1.
<?php } else { ?>
  Otherwise this will show.
<?php } ?>
```

也可以使用替代语法来实现此目的，以使代码更清晰。

```php
<?php if ($x == 1): ?>
  This will show if $x is 1.
<?php else: ?>
  Otherwise this will show.
<?php endif; ?>
```

在输出 HTML 和文本时，特别是输出较大块时，通常更倾向于使用这种编码风格，因为它更容易区分 PHP 代码和网页上出现的 HTML 内容。

## 三元运算符

除了 `if` 和 `switch` 语句之外，还有三元运算符（`?:`）。此运算符可以替代单个 `if/else` 子句。该运算符接受三个表达式。如果第一个表达式计算结果为 `true`，则返回第二个表达式；如果为 `false`，则返回第三个表达式。

```php
// 三元运算符表达式
$y = ($x == 1) ? 1 : 2;
```

在 PHP 中，此运算符不仅可以作为表达式使用，还可以用作语句。

```php
// 三元运算符语句
($x == 1) ? $y = 1 : $y = 2;
```

编程术语*表达式*（*expression*）指的是可求值的代码，而*语句*（*statement*）则是以分号或右花括号结尾的代码段。