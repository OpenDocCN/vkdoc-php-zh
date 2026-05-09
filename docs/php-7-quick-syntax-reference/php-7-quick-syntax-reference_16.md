# 特质

特质是一组可以插入到类中的方法。它于 PHP 5.4 引入，旨在实现更高的代码复用性，同时避免多重继承带来的复杂性。特质使用 `trait` 关键字定义，后跟名称和代码块。其命名规范与类相同，每个单词首字母大写。代码块只能包含静态方法和实例方法。

```
trait PrintFunctionality
{
public function myPrint() { echo 'Hello'; }
}
```

需要特质功能的类可以通过 `use` 关键字引入该特质，后跟特质名称。特质中的方法随后会像直接定义在该类中一样工作。

```
class MyClass
{
// 插入特质方法
use PrintFunctionality;
}
$o = new MyClass();
$o->myPrint(); // 输出 "Hello"
```

一个类可以使用多个特质，只需将它们放在逗号分隔的列表中。同样地，一个特质也可以由一个或多个其他特质组合而成。

## 继承与特质

特质方法会覆盖继承的方法。同样，类中定义的方法会覆盖由特质插入的方法。

```
class MyParent
{
public function myPrint() { echo 'Base'; }
}
class MyChild extends MyParent
{
// 覆盖继承的方法
use PrintFunctionality;
// 覆盖特质插入的方法
public function myPrint() { echo 'Child'; }
}
$o = new MyChild();
$o->myPrint(); // 输出 "Child"
```

## 特质使用指南

单继承有时会迫使开发者在代码复用和概念清晰的类层次结构之间做出选择。为了获得更高的代码复用性，方法可以移动到类层次结构的根部，但这会导致类开始包含它们不需要的方法，从而降低代码的可理解性和可维护性。另一方面，强制保持类层次结构的清晰性往往会导致代码重复，这可能引发不一致性。特质提供了一种避免单继承这一缺陷的方法，它能实现独立于类层次结构的代码复用。

