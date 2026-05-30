# 第 10 章 ■ 模型

```php
$first = $query->first();

$class = get_class($this);

if ($first)

{

return new $class(

$query->first()

);

}

return null;

}

public static function all($where = array(), $fields = array("*"), 
$order = null, $direction = null, $limit = null, $page = null)

{

$model = new static();

return $model->_all($where, $fields, $order, $direction, $limit, $page);

}

protected function _all($where = array(), $fields = array("*"), 
$order = null, $direction = null, $limit = null, $page = null)

{

$query = $this

->connector

->query()

->from($this->table, $fields);

foreach ($where as $clause => $value)

{

$query->where($clause, $value);

}

if ($order != null)

{

$query->order($order, $direction);

}

if ($limit != null)

{

$query->limit($limit, $page);

}

$rows = array();

$class = get_class($this);

foreach ($query->all() as $row)

{

$rows[] = new $class(

$row

);

}

return $rows;

}

public static function count($where = array())
```

[www.it-ebooks.info](http://www.it-ebooks.info/)