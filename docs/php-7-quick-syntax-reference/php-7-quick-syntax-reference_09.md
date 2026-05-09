# 循环中的括号与替代语法

与条件语句类似，循环中的括号可以改写为使用冒号和`endwhile`、`endfor`或`endforeach`关键字的替代语法。

```
while ($i < 10): echo $i++; endwhile;
for ($i = 0; $i < 10; $i++): echo $i; endfor;
foreach ($a as $v): echo $v; endforeach;
```

这样做的主要好处是提高了可读性，尤其是在处理较长循环时。

## `break`

有两个特殊的关键字可以在循环内部使用——`break`和`continue`。`break`关键字结束循环结构的执行。

```
for (;;) { break; } // 结束 for 循环
```

它可以接受一个数字参数，指定要跳出多少层嵌套的循环结构。

```
$i = 0;
while ($i++ < 10)
{
    for (;;) { break 2; } // 结束 for 和 while 循环
}
```

## `continue`

`continue`关键字可以在任何循环语句中使用，用于跳过当前循环的剩余部分，并继续执行下一次迭代的开始处。

```
while ($i++ < 10) { continue; } // 开始下一次迭代
```

该关键字可以接受一个参数，指定要跳过多少层封闭循环的剩余部分。

```
$i = 0;
while ($i++ < 10)
{
    for (;;) { continue 2; } // 开始下一次 while 迭代
}
```

与许多其他语言不同，`continue`语句也适用于`switch`，在`switch`中它的行为与`break`相同。因此，要从`switch`内部跳过一次迭代，需要使用`continue 2`。

```
$i = 0;
while ($i++ < 10)
{
    switch ($i)
    {
        case 1: continue 2; // 开始下一次 while 迭代
    }
}
```

## `goto`

PHP 5.3 引入了第三个跳转语句`goto`，它可以跳转到指定的标签。标签是一个名称后跟一个冒号（`:`）。

```
goto myLabel; // 跳转到标签
myLabel:      // 标签声明
```

目标标签必须位于同一个脚本文件和作用域内。因此，`goto`不能用于跳入循环结构，只能跳出。

```
loop:
while (!$finished)
{
    // ...
    if ($try_again) goto loop; // 重新开始循环
}
```

通常，最好避免使用`goto`语句，因为它往往会使执行流程难以理解。

