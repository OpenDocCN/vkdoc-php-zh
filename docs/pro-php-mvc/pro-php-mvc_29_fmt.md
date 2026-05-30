# 第 10 章 ■ 模型

```php
public function getColumn($name)
{
    if (!empty($this->_columns[$name]))
    {
        return $this->_columns[$name];
    }
    return null;
}

public function getPrimaryColumn()
{
    if (!isset($this->_primary))
    {
        $primary;
        foreach ($this->columns as $column)
        {
            if ($column["primary"])
            {
                $primary = $column;
                break;
            }
        }
        $this->_primary = $primary;
    }
    return $this->_primary;
}

public static function first($where = array(), $fields = array("*"), 
$order = null, $direction = null)
{
    $model = new static();
    return $model->_first($where, $fields, $order, $direction);
}

protected function _first($where = array(), $fields = array("*"), 
$order = null, $direction = null)
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
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)