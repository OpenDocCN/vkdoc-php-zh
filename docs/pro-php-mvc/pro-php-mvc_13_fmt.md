# 第 8 章 ■ 模板

```php
foreach ($this->_map as $_delimiter => $_type)
{
    if (!$delimiter || StringMethods::indexOf($source, $type["opener"]) == -1)
    {
        $delimiter = $_delimiter;
        $type = $_type;
    }
    $indexOf = StringMethods::indexOf($source, $_type["opener"]);
    if ($indexOf > -1)
    {
        if (StringMethods::indexOf($source, $type["opener"]) > $indexOf)
        {
            $delimiter = $_delimiter;
            $type = $_type;
        }
    }
}
if ($type == null)
{
    return null;
}
return array(
    "type" => $type,
    "delimiter" => $delimiter
);
```

`_handler()`方法接收一个`$node`数组，并确定要执行的正确处理器方法。`handle()`方法使用`_handler()`方法获取正确的处理器方法并执行它，如果执行语句的处理器时出现问题，则抛出`Template\Exception\Implementation`异常。`match()`方法评估一个`$source`字符串，以确定它是否匹配某个标签或语句。

这个类可能看起来很奇怪，因为我们甚至不知道实现文件是什么样的。实际上，它的三个方法都直接处理仅在`Template\Implementation`子类中出现的内容。每种语言都有规则或语法。我们需要定义这些规则，以便我们的解析器能够知道如何处理需要解析的标记。我们在`Template\Implementation`子类中定义这些规则。清单 8-4 展示了默认模板解析器的语法映射。

## 清单 8-4. 默认模板实现的语法/语言映射

```php
namespace Framework\Template\Implementation
{
    use Framework\Template as Template;
    use Framework\StringMethods as StringMethods;

    class Standard extends Template\Implementation
    {
        protected $_map = array(
            "echo" => array(
                "opener" => "{echo",
                "closer" => "}",
                "handler" => "_echo"
            ),
            "script" => array(
                "opener" => "{script",
                "closer" => "}",
                "handler" => "_script"
            ),
            "statement" => array(
                "opener" => "{",
                "closer" => "}",
                "tags" => array(
                    "foreach" => array(
                        "isolated" => false,
                        "arguments" => "{element} in {object}",
                        "handler" => "_each"
                    ),
                    "for" => array(
                        "isolated" => false,
                        "arguments" => "{element} in {object}",
                        "handler" => "_for"
                    ),
                    "if" => array(
                        "isolated" => false,
                        "arguments" => null,
                        "handler" => "_if"
                    ),
                    "elseif" => array(
                        "isolated" => true,
                        "arguments" => null,
                        "handler" => "_elif"
                    ),
                    "else" => array(
                        "isolated" => true,
                        "arguments" => null,
                        "handler" => "_else"
                    ),
                    "macro" => array(
                        "isolated" => false,
                        "arguments" => "{name}({args})",
                        "handler" => "_macro"
                    ),
                    "literal" => array(
                        "isolated" => false,
                        "arguments" => null,
                        "handler" => "_literal"
                    )
                )
            )
        );
    }
}
```

清单 8-4 中的代码展示了所谓的语法/语言映射。它是我们模板方言的语言标签列表，以便解析器能够知道不同类型的模板标签是什么，以及如何解析它们。我们可以看到，默认模板方言只有三种类型的标签：`echo`、`script`和`statement`。

`echo`标签是解析器中最基本的标签；它们仅用于将变量数据添加到模板解析器的输出中。你可以将它们视为 PHP 的`echo`语句。

`script`标签用于执行任意的 PHP 脚本。虽然使用这些标签并不理想，但它们适用于需要内联执行 PHP 脚本的任何情况。你可以将它们视为开闭的`<?php ... ?>`标签。

`statement`标签用于我们想要模板支持的所有控制结构。它们包含多种子类型，例如`if`、`foreach`和`else`。它们还包含`macro`（或函数）和`literal`（类似 CDATA）语句子类型。

每个标签都有`opener`和`closer`字符串，告知解析器应如何处理每个片段的开始和结束。每个标签（和子类型）都有`handler`字符串，告知解析器在需要评估相应标签（或子类型）时应调用哪个内部函数。