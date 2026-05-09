# 清单 9-12. `Database\Driver` 数据访问方法

```php
namespace Framework\Database
{

use Framework\Base as Base;
use Framework\ArrayMethods as ArrayMethods;
use Framework\Database\Exception as Exception;

class Query extends Base
{

public function first()
{

$limit = $this->_limit;
$offset = $this->_offset;

$this->limit(1);
$all = $this->all();
$first = ArrayMethods::first($all);

if ($limit)
{

$this->_limit = $limit;
}

if ($offset)
{

$this->_offset = $offset;
}

return $first;
}

public function count()
{

$limit = $this->limit;
$offset = $this->offset;
$fields = $this->fields;

$this->_fields = array($this->from => array("COUNT(1)" => "rows"));
$this->limit(1);

$row = $this->first();

$this->_fields = $fields;

if ($fields)
{

$this->_fields = $fields;
}

if ($limit)
{

$this->_limit = $limit;
}

if ($offset)
{

$this->_offset = $offset;
}

return $row["rows"];
}

}
}
```

