# 第 15 章 ■ 搜索

***清单 15-9.*** 扩展实现实用方法

```php
protected function _getKey($tree)
{
    if (empty($tree["arguments"]["key"]))
    {
        return null;
    }
    return trim($tree["arguments"]["key"]);
}

protected function _setValue($key, $value)
{
    if (!empty($key))
    {
        $default = $this-> getDefaultKey();
        $data = Registry::get($default, array());
        $data[$key] = $value;
        Registry::set($default, $data);
    }
}

protected function _getValue($key)
{
    $data = Registry::get($this->getDefaultKey());
    if (isset($data[$key]))
    {
        return $data[$key];
    }
    return "";
}
```

`_getKey()` 方法简单地从语言构造中提取提供的存储键。`_setValue()` 方法提供了可从任何地方访问的存储解决方案。过去，我们使用 `Registry` 类来存储类实例，但其美妙之处在于我们也可以存储普通对象和数组。此方法检索数据数组（如果不存在则默认为空数组），并设置提供的 `$key`/`$value`。然后将数据数组存回注册中心。最后，`_getValue()` 方法查询存储的数据数组，以获取与提供的 `$key` 匹配的值。

创建这些方法后，我们可以查看最终的模板构造，从清单 15-10 中所示的 set 构造开始。

***清单 15-10.*** `set()` 处理程序

```php
public function set($key, $value)
{
    if (StringMethods::indexOf($value, "\$_text") > -1)
    {
        $first = StringMethods::indexOf($value, "\"");
        $last = StringMethods::lastIndexOf($value, "\"");
        $value = stripslashes(substr($value, $first + 1, ($last - $first) - 1));
    }
    if (is_array($key))
    {
        $key = $this->_getKey($key);
    }
    $this->_setValue($key, $value);
}
```

`set()` 处理程序有两个有趣之处。首先，它是我们在任何模板实现中使用的第一个公有构造处理程序。这是因为我们希望共享存储不仅可以用于模板文件内部，还可以在应用程序的控制器和操作中使用。

`set()` 方法有趣的第二个原因是它将值字符串从类似于 `$_text = "foo";` 的形式修改为类似于 `foo` 的形式。这是因为 `Template` 类解析模板文件中文本节点的方式。如果你回顾一下 `Template->_script()` 方法，你会看到我们首先做的事情之一就是将纯文本节点转换为直接将其添加到 `$_text` 数组的代码。这是预期的行为，我们需要在这种边缘情况下还原。

我们通过删除第一个 `"` 字符之前的所有内容以及最后一个 `"` 字符之后的所有内容来实现这一点。这给了我们原始数据，即我们想要存储的内容。

接下来，我们处理 `prepend()` 和 `append()` 处理程序，如清单 15-11 所示。

***清单 15-11.*** `prepend()` 和 `append()` 处理程序

```php
public function append($key, $value)
{
    if (is_array($key))
    {
        $key = $this->_getKey($key);
    }
    $previous = $this->_getValue($key);
    $this->set($key, $previous.$value);
}

public function prepend($key, $value)
{
    if (is_array($key))
    {
        $key = $this->_getKey($key);
    }
    $previous = $this->_getValue($key);
    $this->set($key, $value.$previous);
}
```

与 `set()` 处理程序一样，这些处理程序方法也是公有的。它们也允许提供字符串或节点树作为 `$key`。如果提供了节点树，将使用 `_getKey()` 方法提取所需的存储键。

[www.it-ebooks.info](http://www.it-ebooks.info/)

