# 第 8 章 ■ 模板

```php
class Template extends Base
{
    /**
     * @readwrite
     */
    protected $_implementation;

    /**
     * @readwrite
     */
    protected $_header = "if (is_array(\$_data) && sizeof(\$_data)) extract(\$_data); \$_text = array();";

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
```

[www.it-ebooks.info](http://www.it-ebooks.info/)