# PHP 模板解析器实现

## 类的引入与基础结构

```php
use Framework\ArrayMethods as ArrayMethods;
use Framework\StringMethods as StringMethods;
use Framework\Template\Exception as Exception;

class Template extends Base
{
    protected function _script($tree)
    {
        $content = array();
        
        if (is_string($tree))
        {
            $tree = addslashes($tree);
            return "\$_text[] = \"{$tree}\";";
        }
        
        if (sizeof($tree["children"]) > 0)
        {
            foreach ($tree["children"] as $child)
            {
                $content[] = $this->_script($child);
            }
        }
        
        if (isset($tree["parent"]))
        {
            return $this->_implementation->handle($tree, implode($content));
        }
        
        return implode($content);
    }
}
```

## `_script()` 方法说明

最后一个受保护的解析器方法是 `_script()` 方法。它遍历（由 `_tree()` 方法生成的）层次结构，解析纯文本节点，并间接调用每个有效标签的处理程序。`_script()` 方法应生成语法正确的函数体。

## 解析器的完整性与简化需求

我们的解析器在技术上已经完备。如果这些受保护的方法是公开的，我们就可以将一系列语句串联起来实现多种用途：将模板转换为数组，将该数组转换为树，并利用脚本转换该树。不过，我们想要更简便的方法。需要牢记的是，我们的模板是预编译的，也就是说我们需要先将它们编译成函数，然后才能使用。

因此，我们可以通过提供两个公共方法来简化模板解析：一个用于将模板转换为函数，另一个用于根据输入数据数组来处理模板。让我们看看清单 8-11 中的这两个方法。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 主公共解析器方法

**清单 8-11.** 主公共解析器方法

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
    use Framework\Template\Exception as Exception;
    
    class Template extends Base
    {
        public function parse($template)
        {
            if (!is_a($this->_implementation, "Framework\Template\Implementation"))
            {
                throw new Exception\Implementation();
            }
            
            $array = $this->_array($template);
            $tree = $this->_tree($array["all"]);
            $this->_code = $this->header . $this->_script($tree) . $this->footer;
            $this->_function = create_function("\$_data", $this->code);
            
            return $this;
        }
        
        public function process($data = array())
        {
            if ($this->_function == null)
            {
                throw new Exception\Parser();
            }
            
            try
            {
                $function = $this->_function;
                return $function($data);
            }
            catch (\Exception $e)
            {
                throw new Exception\Parser($e);
            }
        }
    }
}
```

## `parse()` 与 `process()` 方法说明

除了为模板解析器分配模板实现之外，在评估和填充模板时，你只会使用 `parse()` 和 `process()` 这两个公共方法。`parse()` 方法利用我们迄今为止为模板解析器编写的所有代码来创建一个可用的函数。它将处理后的模板包裹在我们之前看到的 `$_header` 和 `$_footer` 变量中。然后，通过调用 PHP 的 `create_function` 方法对其求值，并将其赋值给受保护的 `$_function` 属性。

[www.it-ebooks.info](http://www.it-ebooks.info/)

`process()` 方法检查受保护的 `$_function` 属性是否存在，如果不存在则抛出 `Template\Exception\Parser` 异常。然后，它尝试使用传递给它的 `$data` 执行生成的函数。如果函数执行出错，则抛出另一个 `Template\Exception\Parser` 异常。

## 示例模板展示

让我们看看解析器在后台如何处理清单 8-12 中所示的模板。

**清单 8-12.** 示例模板

```php
{if $name && $address}
    name: {echo $name} <br />
    address: {echo $address} <br />
{/if}
{elseif $name}
    name: {echo $name} <br />
{/elseif}
{foreach $value in $stack}
    {foreach $item in $value}
        item ({echo $item_i}): {echo $item} <br />
    {/foreach}
{/foreach}
{foreach $value in $empty}
    <!--never printed-->
{/foreach}
{else}
    nothing in that stack! <br />
{/else}
{macro test($args)}
    this item's value is: {echo $args} <br />
{/macro}
{foreach $value in $stack["one"]}
    {echo test($value)}
{/foreach}
{if true}
    in first <br />
    {if true}
        in second <br />
        {if true}
            in third <br />
        {/if}
    {/if}
{/if}
```


```{literal}
{echo "hello world"}
{/literal}
```

模板解析仅当有数据应用时才有意义。在我们的示例中，将使用清单 8-13 给出的数据数组。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 8 章 ■ 模板

**清单 8-13.** 示例模板数据

```
array(
    "name" => "Chris",
    "address" => "Planet Earth!",
    "stack" => array(
        "one" => array(1, 2, 3),
        "two" => array(4, 5, 6)
    ),
    "empty" => array()
)
```

■ **注意** 模板解析器生成的函数代码十分杂乱。下面的代码看起来可能很奇怪，但这只是解析器执行字符串替换的结果。

如果查看示例模板，我们大概能想象出背后构建的层级结构。`_script()`方法编译生成的函数虽然有些杂乱，但仍然值得关注。请看清单 8-14 中生成的函数。

**清单 8-14.** 生成的函数

```
extract($_data); $_text = array();$_text[] = " ";if ($name && $address) {$_text[] = "
name: ";$_text[] = $name;$_text[] = " < br />
address: ";$_text[] = $address;$_text[] = " < br />
";}elseif ($name) {$_text[] = "
name: ";$_text[] = $name;$_text[] = " < br />
";}$_text[] = "
";foreach ($stack as $value_i => $value) {
$_text[] = "
";foreach ($value as $item_i => $item) {
$_text[] = "
item (";$_text[] = $item_i;$_text[] = "): ";$_text[] = $item;$_text[] = " < br />
";
}$_text[] = "
";
}$_text[] = "
";
// 完整列表已缩短
```

真是混乱！最主要的原因是解析器保留了原始示例模板中的空白字符（特别是换行符）。你看到的大部分杂乱内容都源自这些空白。处理后的模板（应用数据后的解析模板）则相当简洁，如清单 8-15 所示。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 8 章 ■ 模板

**清单 8-15.** 结果

```
name: Chris
address: Planet Earth!
item (0): 1
item (1): 2
item (2): 3
item (0): 4
item (1): 5
item (2): 6
nothing in that stack!
this item's value is: 1
this item's value is: 2
this item's value is: 3
in first
in second
in third
{="hello world"}
```

### 优势

你可能会问，用一种模板语言替换另一种有什么好处。毕竟 PHP 本身也算一种模板语言。我们甚至没有完全移除 PHP 的痕迹——`$`符号和严格的语法——那究竟获得了什么呢？

归根结底，我们本可以在一个简单的模板方言上花费更多时间。我们甚至可以让模板看起来和感觉上像普通的 HTML。而我们最终得到的是一个比普通 PHP 更易于理解和学习的东西。

你希望能够将代码交给普通的前端技术/设计师，让他们在没有丰富 PHP 经验的情况下，也能立即看懂并理解发生了什么。我们的模板方言/解析器有助于实现这一目标。

市面上已经有很多模板方言/解析器。我们可能会用到比这里构建的更复杂的方案，但了解创建一个模板解析器背后的原理是有益的，这样我们既能欣赏他人的工作成果，也能根据客户或项目经理的需求进行定制。

最后，我们将看一下完整的`Template\Implementation\Standard`和`Template`类，如清单 8-16 和 8-17 所示。

**清单 8-16.** `Template\Implementation\Standard`类

```
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
            [www.it-ebooks.info](http://www.it-ebooks.info/)
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
```

```php
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
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

