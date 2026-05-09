# 排版后的内容

对象`stdClass`是一个内置的类定义，每当在没有任何类定义的情况下创建对象时都会使用它。这是因为 JSON 响应返回的是一个对象，而不是一个数组。

在 JSON 中，温度值没有单位。假设它始终以开尔文为单位返回温度，因此无需进行特殊检查。摄氏度和华氏度的温度值计算方式保持不变。

城市名称、风速等元素的构建方式也略有不同，但概念相似。处理响应的代码如下所示：

```php
<?php

// 15_4.php

$json = file_get_contents('http://api.openweathermap.org/data/2.5/weather?q=Los%20

Angeles,usxxx&appid=<API KEY>');

$weather = json_decode($json);

echo "City: {$weather->name}\n";

$k = $weather->main->temp;

$c = $k - 273.15;

$f = $c * 9 / 5 + 32;

printf("Temperature: %3.1f K / %3.1f C / %3.1f F\n", $k, $c, $f); echo "Wind: {$weather->wind->speed} mph, direction: {$weather->wind->deg}\n";
```

此脚本将生成类似于示例`14_4.php`中的输出。

```
City: Los Angeles
Temperature: 290.7 K / 17.6 C / 63.7 F
Wind: 1.31 mph, direction: 201.001
```

