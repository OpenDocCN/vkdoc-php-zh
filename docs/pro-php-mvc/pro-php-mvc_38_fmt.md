# 此构件的目标

此构件的目标是获取一个子模板并将其放置在主模板中。子模板应与主模板同时处理，以便任何逻辑能够同步执行。这是模板之间相互通信以共享数据的一种方式。

我们首先使用此实现创建一个新的`Template`实例。这意味着子模板可以拥有它们自己的子模板，并且也能访问其他新的模板构件。

我们获取文件的内容（文件名由`include`标签提供）。然后，我们解析模板实例，并传入模板文件内容。关键在于为每个被包含的模板创建一个新方法，以便所有处理能同时进行。每个`Template`实例生成的函数都返回一个字符串，因此我们只需将此功能封装到一个新函数中并立即调用（将其返回值添加到`$_text`数组中）。这意味着当主模板被处理时，它将调用子模板的结果。这或许是我们正在添加的所有新模板构件中最棘手的一个。

### URL 请求

下一个构件需要一套新的类，这些类将允许我们向远程 URL 发起 HTTP 请求。你可以在代码清单 15-4 中看到这些`Request`类中的第一个。

**代码清单 15-4.** `Request`类

```
namespace Framework
{

use Framework\Base as Base;
use Framework\StringMethods as StringMethods;
use Framework\RequestMethods as RequestMethods;
use Framework\Request\Exception as Exception;

class Request extends Base
{

    protected $_request;

    /**
     * @readwrite
     */
    public $_willFollow = true;

    /**
     * @readwrite
     */
    protected $_headers = array();

    /**
     * @readwrite
     */
    protected $_options = array();

    /**
     * @readwrite
     */
    protected $_referer;

    /**
     * @readwrite
     */
    protected $_agent;

    protected function _getExceptionForImplementation($method)
    {
        return new Exception\Implementation("{$method} not implemented");
    }

    protected function _getExceptionForArgument()
    {
        return new Exception\Argument("Invalid argument");
    }

    public function __construct($options = array())
    {
        parent::__construct($options);
        $this->setAgent(RequestMethods::server("HTTP_USER_AGENT",
        "Curl/PHP ".PHP_VERSION));
    }

    public function delete($url, $parameters = array())
    {
        return $this->request("DELETE", $url, $parameters);
    }

    function get($url, $parameters = array())
    {
        if (!empty($parameters))
        {
            $url .= StringMethods::indexOf($url, "?") ? "&" : "?";
            $url .= is_string($parameters) ? $parameters :
            http_build_query($parameters, "", "&");
        }
        return $this->request("GET", $url);
    }

    function head($url, $parameters = array())
    {
        return $this->request("HEAD", $url, $parameters);
    }

    function post($url, $parameters = array())
    {
        return $this->request("POST", $url, $parameters);
    }

    function put($url, $parameters = array())
    {
        return $this->request("PUT", $url, $parameters);
    }

}
}
```

`Request`类拥有所有常规`Base`子类的特性，以及一些请求方法（`get`、`post`、`put`、`delete`和`head`）。这些方法代表了不同类型的请求方法，但最终它们都会调用同一个`request()`方法。其他需要注意的事项包括：构造函数设置了用户代理，并且`get`方法将传入的参数数组转换为有效的查询字符串。

`request()`方法，如代码清单 15-5 所示，负责执行（或委派）实际工作。

**代码清单 15-5.** `request()`方法

```
function request($method, $url, $parameters = array())
{
    $request = $this->_request = curl_init();

    if (is_array($parameters))
    {
        $parameters = http_build_query($parameters, "", "&");
    }

    $this
    ->_setRequestMethod($method)
    ->_setRequestOptions($url, $parameters)
    ->_setRequestHeaders();

    $response = curl_exec($request);

    if ($response)
    {
        $response = new Request\Response(array(
            "response" => $response
        ));
    }
    else
    {
        throw new Exception\Response(curl_errno($request).' - '.curl_error($request));
    }
}
```