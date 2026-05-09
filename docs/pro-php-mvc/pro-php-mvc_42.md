# 第 15 章 ■ 搜索

```php
$version = array_shift($headers);

preg_match('#HTTP/(\d\.\d)\s(\d\d\d)\s(.*)#', $version, $matches);

$this->_headers["Http-Version"] = $matches[1];

$this->_headers["Status-Code"] = $matches[2];

$this->_headers["Status"] = $matches[2]." ".$matches[3];

foreach ($headers as $header)
{
    preg_match('#(.*?)\:\s(.*)#', $header, $matches);
    $this->_headers[$matches[1]] = $matches[2];
}
```

```php
function __toString()
{
    return $this-> getBody();
}
```

`Request\Response`类相对简单。它接受一个响应构造函数选项，该选项是 HTTP 请求的结果。它将响应字符串拆分为标头和正文，可通过 getter 方法获取。

清单 15-8 展示了这些类在创建第二个模板构造（partial 构造）时的用法。

***清单 15-8.*** `_partial()` 处理程序

```php
protected function _partial($tree, $content)
{
    $address = trim($tree["raw"], " /");
    if (StringMethods::indexOf($address, "http") != 0)
    {
        $host = RequestMethods::server("HTTP_HOST");
        $address = "http://{$host}/{$address}";
    }
    $request = new Request();
    $response = addslashes(trim($request-> get($address)));
    return "\$_text[] = \"{$response}\";";
}
```

`_partial()` 处理程序接受一个 URL（`$address`），该 URL 可以是绝对路径（例如，[`www.google.com`](http://www.google.com)）或相对于同一应用程序的路径（例如，[`localhost/messages/feed.html`](http://localhost/messages/feed.html)）。如果 URL 不以 `http` 开头，则将其视为相对 URL 并进行相应调整。我们向该 URL 发出 GET 请求，并将结果返回到模板的 `$_text` 数组中，该数组将渲染到最终的模板输出中。

接下来，我们需要添加清单 15-9 中所示的三个实用方法，这些方法将在剩余的模板构造中使用。

[www.it-ebooks.info](http://www.it-ebooks.info/)

