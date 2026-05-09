# 接下来，我们需要创建 `$_validators` 数组中定义的验证方法，如清单 16-3 所示。

## 第 16 章 ■ 设置

**清单 16-3.** 验证方法

```
protected function _validateRequired($value)
{
    return !empty($value);
}
 
protected function _validateAlpha($value)
{
    return StringMethods::match($value, "#^([a-zA-Z]+)$#");
}
 
protected function _validateNumeric($value)
{
    return StringMethods::match($value, "#^([0–9]+)$#");
}
 
protected function _validateAlphaNumeric($value)
{
    return StringMethods::match($value, "#^([a-zA-Z0-9]+)$#");
}
 
protected function _validateMax($value, $number)
{
    return strlen($value) < = (int) $number;
}
 
protected function _validateMin($value, $number)
{
    return strlen($value) > = (int) $number;
}
```

这些验证方法都相当简单，各自执行所需的验证，并在成功时返回 `true`，失败时返回 `false`。在后续章节添加自定义字段验证时，我们需要牢记这个重要行为。

`validate()` 方法的基本功能可以通过清单 16-4 所示的伪代码来理解。

**清单 16-4.** `validate()` 方法的工作方式

```
function validate
{
    设置错误标志为 false
    遍历每个列
    {
        如果列需要验证
        {
            遍历每个条件
            {
                执行条件验证方法
                如果验证失败
                {
                    生成错误消息并保存到数组
                    设置错误标志为 true
                }
            }
        }
    }
    返回错误标志
}
```

清单 16-5 中展示的实际 `validate()` 方法本质上相同，但包含更多细节。

**清单 16-5.** `validate()` 方法

```
public function validate()
{
    $this->_errors = array();
    $columns = $this->getColumns();
 
    foreach ($columns as $column)
    {
        if ($column["validate"])
        {
            $error = false;
            $pattern = "#[a-z] + \(([a-zA-Z0-9, ]+)\)#";
            $raw = $column["raw"];
            $name = $column["name"];
            $validators = $column["validate"];
            $label = $column["label"];
            $defined = $this->getValidators();
 
            foreach ($validators as $validator)
            {
                $function = $validator;
                $arguments = array(
                    $this->$raw
                );
 
                $match = StringMethods::match($validator, $pattern);
 
                if (count($match) > 0)
                {
                    $matches = StringMethods::split($match[0], ",\s*");
                    $arguments = array_merge($arguments, $matches);
                    $offset = StringMethods::indexOf($validator, "(");
                    $function = substr($validator, 0, $offset);
                }
 
                if (!isset($defined[$function]))
                {
                    throw new Exception\Validation("The {$function} validator is not defined");
                }
 
                $template = $defined[$function];
 
                if (!call_user_func_array(array($this, $template["handler"]), $arguments))
                {
                    $replacements = array_merge(array(
                        $label ? $label : $raw
                    ), $arguments);
                    $message = $template["message"];
 
                    foreach ($replacements as $i => $replacement)
                    {
                        $message = str_replace("{{$i}}", $replacement, $message);
                    }
 
                    if (!isset($this->_errors[$name]))
                    {
                        $this->_errors[$name] = array();
                    }
 
                    $this->_errors[$name][] = $message;
                    $error = true;
                }
            }
        }
    }
 
    return !$error;
}
```

`validate()` 方法首先获取一个列列表并进行迭代。对于每个列，我们判断是否需要进行验证。然后，我们将 `@validate` 元数据拆分为一个验证条件列表。如果某个条件带有参数（例如 `max(100)`），我们就提取这些参数。

接着，我们对列数据运行每个验证方法，并为那些验证失败的条件生成错误消息。最后返回一个 `true`/`false`，以指示整个验证是否通过。

这在我们的控制器中改变了一些事情。首先，我们不再需要在每个动作中创建单独的验证逻辑。验证逻辑在我们的模型中定义，并且在我们调用 `validate()` 方法时就会生效。

让我们看看如何修改 `Users` 控制器中的 `register()` 动作，如清单 16-6 所示。

**清单 16-6.** `register()` 动作

```
public function register()
{
    $view = $this->getActionView();
 
    if (RequestMethods::post("register"))
    {
        $user = new User(array(
```



`"first" = > RequestMethods::post("first"),`

[www.it-ebooks.info](http://www.it-ebooks.info/)

