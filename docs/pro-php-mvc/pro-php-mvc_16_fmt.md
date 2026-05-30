# 第 8 章 ■ 模板

```php
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

protected function _for($tree, $content)
{
    $object = $tree["arguments"]["object"];
    $element = $tree["arguments"]["element"];
    return $this->_loop(
        $tree,
        "for ({$element}_i = 0; {$element}_i < sizeof({$object}); {$element}_i++) {
            {$element} = {$object}[{$element}_i];
            {$content}
        }"
    );
}

protected function _if($tree, $content)
{
    $raw = $tree["raw"];
    return "if ({$raw}) {{$content}}";
}

protected function _elif($tree, $content)
{
    $raw = $tree["raw"];
    return "elseif ({$raw}) {{$content}}";
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)