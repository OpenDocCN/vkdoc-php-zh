# 第 8 章 ■ 模板

```php
protected function _else($tree, $content)
{
    return "else {{$content}}";
}

protected function _macro($tree, $content)
{
    $arguments = $tree["arguments"];
    $name = $arguments["name"];
    $args = $arguments["args"];
    return "function {$name}({$args}) {
        \$_text = array();
        {$content}
        return implode(\$_text);
    }";
}

protected function _literal($tree, $content)
{
    $source = addslashes($tree["source"]);
    return "\$_text[] = \"{$source}\";";
}

protected function _loop($tree, $inner)
{
    $number = $tree["number"];
    $object = $tree["arguments"]["object"];
    $children = $tree["parent"]["children"];
    if (!empty($children[$number + 1]["tag"]) && $children[$number + 1]["tag"] == "else")
    {
        return "if (is_array({$object}) && sizeof({$object}) > 0) {{$inner}}";
    }
    return $inner;
}
```

**代码清单 8-17.** 模板类

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
    use Framework\Template\Exception as Exception;
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

