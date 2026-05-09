# 第 15 章 ■ 搜索

`Request\Response`类负责处理 HTTP 请求的结果，如清单 15-7 所示。

***清单 15-7.*** 请求/响应类

```
namespace Framework\Request
{
    use Framework\Base as Base;
    use Framework\Request\Exception as Exception;

    class Response extends Base
    {
        protected $_response;

        /**
         * @read
         */
        protected $_body = null;

        /**
         * @read
         */
        protected $_headers = array();

        protected function _getExceptionForImplementation($method)
        {
            return new Exception\Implementation("{$method} not implemented");
        }

        protected function _getExceptionForArgument()
        {
            return new Exception\Argument("Invalid argument");
        }

        function __construct($options = array())
        {
            if (!empty($options["response"]))
            {
```



```php
$response = $this->_response = $options["response"];

unset($options["response"]);

}

parent::__construct($options);

$pattern = '#HTTP/\d\.\d.*?$.*?\r\n\r\n#ims';

preg_match_all($pattern, $response, $matches);

$headers = array_pop($matches[0]);

$headers = explode("\r\n", str_replace("\r\n\r\n", "", $headers));

$this->_body = str_replace($headers, "", $response);
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

