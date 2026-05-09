# 15. 抽象

抽象类提供了一个部分实现，其他类可以在此基础上构建。当一个类被声明为抽象时，意味着该类除包含正常的类成员外，还可能包含必须在子类中实现的不完整方法。

### 抽象方法

在抽象类中，任何方法都可以被声明为抽象。这些方法随后保持未实现状态，仅指定它们的签名，而它们的代码块被分号替代。

```
abstract class Shape
{
abstract public function myAbstract();
}
```

### 抽象示例

举例来说，以下类有两个属性和一个抽象方法。

```
abstract class Shape
{
private $x = 100, $y = 100;
abstract public function getArea();
}
```

如果一个类继承自此抽象类，则它必须重写该抽象方法。方法签名必须匹配，但访问级别可以放宽限制。

```
class Rectangle extends Shape
{
public function getArea()
{
return $this->x * $this->y;
}
}
```

不能实例化抽象类。它们仅作为其他类的父类，部分地规定了它们的实现。

```
$s = new Shape(); // 编译时错误
```

然而，抽象类可以继承自非抽象（具体）类。

```
class NonAbstract {}
abstract class MyAbstract extends NonAbstract {}
```

### 抽象类与接口

抽象类在许多方面与接口相似。它们都可以定义派生类必须实现的成员签名，并且两者都不能被实例化。主要区别在于：首先，抽象类可以包含非抽象成员，而接口不能。其次，一个类可以实现任意数量的接口，但只能继承自一个类（无论是否抽象）。

```
// 定义默认功能和定义
abstract class Shape
{
public $x = 100, $y = 100;
abstract public function getArea();
}
// 类是一个形状
class Rectangle extends Shape { /*...*/ }
// 定义特定功能
interface iComparable
{
function compare();
}
// 类可以进行比较
class MyClass implements iComparable { /*...*/ }
```

### 抽象指南

抽象类提供了一个部分实现的基类，它规定了子类必须如何行为。当子类共享一些相似性，但在其他实现上存在差异并且需要子类自行定义时，抽象类最为有用。与接口类似，抽象类是面向对象编程中非常有用的结构，有助于开发者遵循良好的编码标准。

