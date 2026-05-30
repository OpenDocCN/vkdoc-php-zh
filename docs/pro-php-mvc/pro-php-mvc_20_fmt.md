# 排版后内容

```
try
{
    $function = $this->_function;
    return $function($data);
}
catch (\Exception $e)
{
    throw new Exception\Parser($e);
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)