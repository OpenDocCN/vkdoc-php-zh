# 在定义了模板解析器必须能够处理的标签和语句类型后，我们需要实现处理相关标签和语句的函数，如清单 8-5 所示。

***清单 8-5.*** 语句处理函数

```
namespace Framework\Template\Implementation
{
    use Framework\Template as Template;
    use Framework\StringMethods as StringMethods;
    
    class Standard extends Template\Implementation
    {
        protected function _echo($tree, $content)
        {
            $raw = $this->_script($tree, $content);
            return "\$_text[] = {$raw}";
        }
        
        protected function _script($tree, $content)
        {
            $raw = !empty($tree["raw"]) ? $tree["raw"] : "";
            return "{$raw};";
        }
        
        protected function _each($tree, $content)
        {
            $object = $tree["arguments"]["object"];
            $element = $tree["arguments"]["element"];
            return $this->_loop(
                $tree,
                "foreach ({$object} as {$element}_i => {$element}) {
                    {$content}
                }"
            );
        }
    }
}
```

`_echo()`方法将字符串`“{echo $hello}”`转换为`“$_text[] = $hello”`，这样它就已经针对最终求值函数进行了优化。类似地，`_script()`方法将字符串`“{:$foo + = 1}”`转换为`“$foo + = 1”`。

`_each()`和`_for()`方法也类似，但依赖于`_loop()`方法来增强其输出，只要这些语句后面跟着`{else}`标签，就会检查这些语句所用数组的内容。

`_each()`方法返回用于对数组执行`foreach`循环的代码，而`_for()`方法则生成用于对数组执行`for`循环的代码。

`_if()`、`_else()`和`_elif()`方法返回代码，用于在我们的模板中执行这些操作。显然，我们可以重写这些方法（以及`_each()`和`_for()`方法）以使用替代的结束语法（`endif`、`endforeach`、`endfor`等），但这完全取决于你。

`_macro()`方法基于`{macro...}...{/macro}`标签集的内容创建一个函数的字符串表示形式。通过使用`{macro}`标签，可以定义函数，然后在模板中使用这些函数。

最后，`_literal()`方法直接引用其内部的任何内容。模板解析器在找到`{/literal}`结束标签之前，会一直直接引用这些内容。现在我们已经有了模板方言实现类（参见清单 8-6），可以开始在我们的解析器中使用它了。但我们的解析器是什么样子的呢？

***清单 8-6.*** 模板类

```
namespace Framework
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
    use Framework\Template\Exception as Exception;
    
    class Template extends Base
    {
        /**
         * @readwrite
         */
        protected $_implementation;
    }
}
```



/**
 * @readwrite
 */
protected $_header = "if (is_array(\$_data) && sizeof(\$_data)) \n
extract(\$_data); \$_text = array();";

/**
 * @readwrite
 */
protected $_footer = "return implode(\$_text);";

/**
 * @read
 */
protected $_code;

/**
 * @read
 */
protected $_function;

public function _getExceptionForImplementation($method)
{
    return new Exception\Implementation("{$method} method not implemented");
}

我们的`Template`类以一些可适应的属性和对异常生成方法的重写开始。然后我们添加了两个实用方法来帮助我们分解模板块，将标签从文本中分离出来，如**清单 8-7**所示。

**清单 8-7.** 标签解析方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
    use Framework\Template\Exception as Exception;

    class Template extends Base
    {
        protected function _arguments($source, $expression)
        {
            $args = $this->_array($expression, array(
                $expression => array(
                    "opener" => "{",
                    "closer" => "}"
                )
            ));

            $tags = $args["tags"];
            $arguments = array();
            $sanitized = StringMethods::sanitize($expression, "()[],.<>*$@");
            foreach ($tags as $i => $tag)
            {
                $sanitized = str_replace($tag, "(.*)", $sanitized);
                $tags[$i] = str_replace(array("{", "}"), "", $tag);
            }

            if (preg_match("#{$sanitized}#", $source, $matches))
            {
                foreach ($tags as $i => $tag)
                {
                    $arguments[$tag] = $matches[$i + 1];
                }
            }
            return $arguments;
        }

        protected function _tag($source)
        {
            $tag = null;
            $arguments = array();
            $match = $this->_implementation->match($source);
            if ($match == null)
            {
                return false;
            }

            $delimiter = $match["delimiter"];
            $type = $match["type"];
            $start = strlen($type["opener"]);
            $end = strpos($source, $type["closer"]);
            $extract = substr($source, $start, $end - $start);

            if (isset($type["tags"]))
            {
                $tags = implode("|", array_keys($type["tags"]));
                $regex = "#^(/){0,1}({$tags})\s*(.*)$#";
                if (!preg_match($regex, $extract, $matches))
                {
                    return false;
                }
                $tag = $matches[2];
                $extract = $matches[3];
                $closer = !!$matches[1];
            }

            if ($tag && $closer)
            {
                return array(
                    "tag" => $tag,
                    "delimiter" => $delimiter,
                    "closer" => true,
                    "source" => false,
                    "arguments" => false,
                    "isolated" => $type["tags"][$tag]["isolated"]
                );
            }

            if (isset($type["arguments"]))
            {
                $arguments = $this->_arguments($extract, $type["arguments"]);
            }
            else if ($tag && isset($type["tags"][$tag]["arguments"]))
            {
                $arguments = $this->_arguments($extract, $type["tags"][$tag]["arguments"]);
            }

            return array(
                "tag" => $tag,
                "delimiter" => $delimiter,
                "closer" => false,
                "source" => $extract,
                "arguments" => $arguments,
                "isolated" => (!empty($type["tags"]) ? $type["tags"][$tag]["isolated"]: false)
            );
        }
    }
}
```

`_tag()`方法调用实现的`match()`方法。`match()`方法会告诉我们的解析器，我们正在解析的模板块是标签还是纯字符串。如果不匹配，`match()`方法将返回`false`。然后它会提取我们在实现中指定的开始和结束字符串之间的所有内容——例如对于`{if $check}`，`_match()`方法会提取标签（`if`）、标签的剩余部分（`$check`）以及它是否是结束标签。

最后，`_tag()`方法会生成一个`$node`数组，其中包含关于该标签的一些元数据。

`_arguments()`方法用于任何具有特定参数格式的语句（例如`for`，`foreach`或`macro`）。它以整齐的关联数组形式返回`{...}`字符之间的内容。现在我们可以从文本中识别标签了，我们需要一个方法来分解模板字符串并将每个片段传递给这些实用方法。请参见**清单 8-8**中的这个方法。

**清单 8-8.** 模板`_array()`方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
    use Framework\Template\Exception as Exception;
```



```php
class Template extends Base
{
    protected function _array($source)
    {
        $parts = array();
        $tags = array();
        $all = array();
        $type = null;
        $delimiter = null;
        while ($source)
        {
            $match = $this->_implementation->match($source);
            $type = $match["type"];
            $delimiter = $match["delimiter"];
            $opener = strpos($source, $type["opener"]);
            $closer = strpos($source, $type["closer"]) + strlen($type["closer"]);
            if ($opener !== false)
            {
                $parts[] = substr($source, 0, $opener);
                $tags[] = substr($source, $opener, $closer - $opener);
                $source = substr($source, $closer);
            }
            else
            {
                $parts[] = $source;
                $source = "";
            }
        }
        foreach ($parts as $i => $part)
        {
            $all[] = $part;
            if (isset($tags[$i]))
            {
                $all[] = $tags[$i];
            }
        }
        return array(
            "text" => ArrayMethods::clean($parts),
            "tags" => ArrayMethods::clean($tags),
            "all" => ArrayMethods::clean($all)
        );
    }
}
```

`_array()` 方法本质上是将模板字符串分解为标签、文本以及两者组合的数组。它通过实现中的 `match()` 和 `ArrayMethods::clean()` 方法来实现这一点。

有了这些数组，我们就可以开始构建标签和文本的层级结构。由于我们的模板语言支持条件语句，并允许我们嵌套它们，我们需要将模板视为树状结构，而不仅仅是字符串替换的手段。**清单 8-9** 展示了我们构建此标签和文本层级结构的方法。

**清单 8-9.** 模板的 `_tree()` 方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
    use Framework\Template\Exception as Exception;

    class Template extends Base
    {
        protected function _tree($array)
        {
            $root = array(
                "children" => array()
            );
            $current = & $root;
            foreach ($array as $i => $node)
            {
                $result = $this->_tag($node);
                if ($result)
                {
                    $tag = isset($result["tag"]) ? $result["tag"] : "";
                    $arguments = isset($result["arguments"]) ? $result["arguments"] : "";
                    if ($tag)
                    {
                        if (!$result["closer"])
                        {
                            $last = ArrayMethods::last($current["children"]);
                            if ($result["isolated"] && is_string($last))
                            {
                                array_pop($current["children"]);
                            }
                            $current["children"][] = array(
                                "index" => $i,
                                "parent" => &$current,
                                "children" => array(),
                                "raw" => $result["source"],
                                "tag" => $tag,
                                "arguments" => $arguments,
                                "delimiter" => $result["delimiter"],
                                "number" => sizeof($current["children"])
                            );
                            $current = & $current["children"][sizeof($current["children"]) - 1];
                        }
                        else if (isset($current["tag"]) && $result["tag"] == $current["tag"])
                        {
                            $start = $current["index"] + 1;
                            $length = $i - $start;
                            $current["source"] = implode(array_slice($array, $start, $length));
                            $current = & $current["parent"];
                        }
                    }
                    else
                    {
                        $current["children"][] = array(
                            "index" => $i,
                            "parent" => &$current,
                            "children" => array(),
                            "raw" => $result["source"],
                            "tag" => $tag,
                            "arguments" => $arguments,
                            "delimiter" => $result["delimiter"],
                            "number" => sizeof($current["children"])
                        );
                    }
                }
                else
                {
                    $current["children"][] = $node;
                }
            }
            return $root;
        }
    }
}
```

`_tree()` 方法循环遍历由 `_array()` 方法生成的模板片段数组，并将它们组织成层级结构。纯文本节点会原封不动地分配给树，而额外的元数据则会与标签一起生成并分配。需要注意的一点是，某些语句具有 `isolated` 属性。这指定在语句之前是否允许有文本。当循环遇到一个孤立的标签时，它会移除前面的段（只要它是纯文本段），从而确保生成的函数代码在语法上正确。我们最后需要进行的模板处理是将模板树转换为 PHP 函数。这将允许我们无需重新编译即可重用模板。请查看**清单 8-10** 了解我们如何实现这一点。

**清单 8-10.** `_script()` 方法

```php
namespace Framework
{
    use Framework\Base as Base;
```



