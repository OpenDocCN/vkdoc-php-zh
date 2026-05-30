# 第 15 章 ■ 搜索

`request()`方法（以及`Request`类）使用`Curl`来发起 HTTP 请求。

该方法首先创建一个新的 curl 资源实例，然后为该实例设置一些参数。接着发起请求，如果请求成功，将以`Request\Response`类实例的形式返回。如果请求失败，则会抛出异常。

最后，销毁 curl 资源并返回响应。在深入了解`Request\Response`类之前，我们需要先看一下清单 15-6 中的那些设置方法。

## 清单 15-6. 请求设置方法

```php
protected function _setOption($key, $value)
{
    curl_setopt($this->_request, $key, $value);
    return $this;
}

protected function _normalize($key)
{
    return "CURLOPT_".str_replace("CURLOPT_", "", strtoupper($key));
}

protected function _setRequestMethod($method)
{
    switch (strtoupper($method))
    {
        case "HEAD":
            $this->_setOption(CURLOPT_NOBODY, true);
            break;
        case "GET":
            $this->_setOption(CURLOPT_HTTPGET, true);
            break;
        case "POST":
            $this->_setOption(CURLOPT_POST, true);
            break;
        default:
            $this->_setOption(CURLOPT_CUSTOMREQUEST, $method);
            break;
    }
    return $this;
}

protected function _setRequestOptions($url, $parameters)
{
    $this
        ->_setOption(CURLOPT_URL, $url)
        ->_setOption(CURLOPT_HEADER, true)
        ->_setOption(CURLOPT_RETURNTRANSFER, true)
        ->_setOption(CURLOPT_USERAGENT, $this->getAgent());

    if (!empty($parameters))
    {
        $this->_setOption(CURLOPT_POSTFIELDS, $parameters);
    }

    if ($this->getWillFollow())
    {
        $this->_setOption(CURLOPT_FOLLOWLOCATION, true);
    }

    if ($this->getReferer())
    {
        $this->_setOption(CURLOPT_REFERER, $this->getReferer());
    }

    foreach ($this->_options as $key => $value)
    {
        $this->_setOption(constant($this->_normalize($key)), $value);
    }

    return $this;
}

protected function _setRequestHeaders()
{
    $headers = array();
    foreach ($this->getHeaders() as $key => $value)
    {
        $headers[] = $key.': '.$value;
    }
    $this->_setOption(CURLOPT_HTTPHEADER, $headers);
    return $this;
}
```

`_setOption()`和`_normalize()`方法是便捷方法，用于在三个重要的`Request`设置方法中执行的任务。如果你熟悉 PHP 的`Curl`类，它们对你来说应该不陌生。

**注意** 你可以通过[`http://www.php.net/manual/en/book.curl.php`](http://www.php.net/manual/en/book.curl.php)了解更多关于 PHP `Curl`类的信息。

`_setRequestMethod()`方法设置与不同请求方法相关的`Curl`参数。某些请求方法需要设置额外的参数（如`GET`和`POST`），而其他请求方法则需要从响应中排除某些内容（如`HEAD`）。

`_setRequestOptions()`方法遍历所有需要设置的请求特定参数。这包括 URL、用户代理、请求是否应该跟随重定向等。它甚至会添加通过`setOptions()`设置方法（或构造选项）指定的任何选项。

最后，`_setRequestHeaders()`方法遍历通过`setHeaders()`设置方法（或构造选项）指定的标头，为请求添加任何自定义标头。

[www.it-ebooks.info](http://www.it-ebooks.info/)