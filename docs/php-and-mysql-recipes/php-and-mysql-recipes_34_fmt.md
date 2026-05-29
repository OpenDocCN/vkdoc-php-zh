# 配方 15-4. JSON API

### 问题

当客户端从多个来源消费 API 时，每个来源通常对内容的格式以及在响应中包含哪些元素有自己的定义。这使得处理这些 API 变得更加困难。

### 解决方案

2015 年，一个名为 JSON API 的新标准达到了 1.0 版本。该标准定义了 API 请求和响应应如何格式化的所有头部信息和其他细节。该标准记录在`http://jsonapi.org`。其目标是改善缓存，从而减少与 API 交互所需的网络流量。更少的网络流量使得在相同硬件上为更多用户提供服务成为可能。

所有 API 的基本结构是一个 JSON 结构，包含以下三个顶级对象中的一个或多个：`data`（数据）、`errors`（错误）和`meta`（元数据）。此外，还允许包含`jsonapi`、`links`（链接）和`includes`（包含）作为顶级对象。

`data`对象应包含响应的主要数据。对于返回单个资源的 API，它可以是一个对象或`null`；对于返回资源列表的 API，它可以是一个对象数组或空数组。

一个用于获取网站访问者数量的简单 API 响应可能如下所示：

```json
{
  'data': {
    'type': 'statistics',
    'id': 'visitors',
    'attributes': {
      'value': 25
    }
  }
}
```

每个资源必须包含`type`和`id`，且它们必须是字符串。可以在顶级添加一个`link`对象来表示 API 的链接。

```json
{
  'data': {
    'type': 'statistics',
    'id': 'visitors',
    'attributes': {
      'value': 25
    }
  },
  'links': {
    'self': "http://localhost/statistics/visitors"
  }
}
```

`link`部分也可用于引用同一数据集中的其他数据。如果 API 返回一篇博客文章，并且你想要链接到同一系列中的下一篇、上一篇、第一篇和最后一篇文章，响应可能如下所示：

```json
{
  'data': {
    'type': 'articles',
    'id': '5',
    'attributes': {
      'title': 'PHP and MySQL Recipes'
    }
  },
  'links': {
    'self': "http://localhost/articles/5",
    'next': "http://localhost/articles/6",
    'prev': "http://localhost/articles/4",
    'first': "http://localhost/articles/1",
    'last': "http://localhost/articles/17"
  }
}
```

所有请求和响应必须使用`Content-Type: application/vnd.api+json`，该标准还描述了如何处理相关数据（如博客文章的评论等）。

### 工作原理

使用像 JSON API 这样的标准的好处之一是，它能够通过一组通用的工具进行处理。

```php
<?php

// 15_5.php

class JsonResource {
  function __construct($type, $id) {
    $this->type = $type;
    $this->id = $id;
    $this->addLink('self', "$type/$id");
  }

  function addAttribute($name, $value) {
    if (!$this->attributes) {
      $this->attributes = new stdClass();
    }
    $this->attributes->$name = $value;
  }
}
```