# 第 10 章 ■ 模型

```php
foreach ($properties as $property)

{

$propertyMeta = $inspector->getPropertyMeta($property);

if (!empty($propertyMeta["@column"]))

{

$name = preg_replace("#^_#", "", $property);

$primary = !empty($propertyMeta["@primary"]);

$type = $first($propertyMeta, "@type");

$length = $first($propertyMeta, "@length");

$index = !empty($propertyMeta["@index"]);

$readwrite = !empty($propertyMeta["@readwrite"]);

$read = !empty($propertyMeta["@read"]) || $readwrite;

$write = !empty($propertyMeta["@write"]) || $readwrite;

$validate = !empty($propertyMeta["@validate"]) 
? $propertyMeta["@validate"] : false;

$label = $first($propertyMeta, "@label");

if (!in_array($type, $types))

{

throw new Exception\Type("{$type} is not a valid type");

}

if ($primary)

{

$primaries++;

}

$columns[$name] = array(

"raw" => $property,

"name" => $name,

"primary" => $primary,

"type" => $type,

"length" => $length,

"index" => $index,

"read" => $read,

"write" => $write,

"validate" => $validate,

"label" => $label

);

}

}

if ($primaries !== 1)

{

throw new Exception\Primary("{$class} must have exactly one 
@primary column");

}

$this->_columns = $columns;

}

return $this->_columns;
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

