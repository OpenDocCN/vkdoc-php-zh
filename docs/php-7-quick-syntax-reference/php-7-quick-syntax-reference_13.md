# 14. 接口

接口规定了使用该接口的类必须实现的方法。它们使用 `interface` 关键字定义，后跟名称和代码块。其命名约定是以小写 `i` 开头，然后每个单词首字母大写。

```
interface iMyInterface {}
```

### 接口签名

接口的代码块可以包含实例方法的签名。这些方法不能有任何实现。相反，它们的方法体被分号替代。接口方法必须始终是公开的。

```
interface iMyInterface
{
public function myMethod();
}
```

此外，接口可以定义常量。这些接口常量的行为与类常量相同，只是它们不能被覆盖。

```
interface iMyInterface
{
const PI = 3.14;
}
```

接口不能继承自类，但它可以继承自另一个接口，这实际上是将多个接口合并为一个。

```
interface i1 {}
interface i2 extends i1 {}
```

### 接口示例

以下示例展示了一个名为 `iComparable` 的接口，它只有一个名为 `Compare` 的方法。请注意，此方法使用了类型提示来确保以正确的类型调用该方法。此功能将在后续章节中介绍。

```
interface iComparable
{
public function compare(iComparable $o);
}
```

`Circle` 类通过在类名后使用 `implements` 关键字并跟上接口名称来实现此接口。如果该类还有 `extends` 子句，则 `implements` 子句需要放在其后。请记住，尽管一个类只能继承自一个父类，但它可以通过指定逗号分隔的列表来实现任意数量的接口。

```
class Circle implements iComparable
{
public $r;
}
```

由于 `Circle` 实现了 `iComparable`，它必须定义 `compare()` 方法。对于此类，该方法返回圆半径之间的差值。实现的方法必须是公开的，并且与接口中定义的方法具有相同的签名。它也可以有更多参数，只要它们是可选的即可。

```
class Circle implements iComparable
{
public $r;
public function compare(iComparable $o)
{
return $this->r - $o->r;
}
}
```

## 接口的使用

接口允许类设计的多重继承，而不会出现功能多重继承所带来的复杂性。要求特定类设计的主要好处可以从 `iComparable` 接口看出，它定义了一个类可以共享的特定功能。它允许开发者在不了解类的实际类型的情况下使用接口成员。为了说明这一点，以下示例展示了一个简单的方法，它接受两个 `iComparable` 对象并返回较大的那个。

```
function largest(iComparable $a, iComparable $b)
{
return ($a->compare($b) > 0) ? $a : $b;
}
```

此方法适用于任何实现了 `iComparable` 接口的相同类型的两个对象。无论对象是什么类型，该方法都能工作，因为它只使用了通过该接口暴露的功能。

### 接口指南

接口为类提供了设计规范，而不包含任何实现。它是一个契约，实现它的类同意提供特定的功能。这有两个好处。首先，它提供了一种确保开发者实现特定方法的方式。其次，由于这些类保证拥有特定方法，因此即使在不知道类实际类型的情况下也能使用它们，这使得代码更加灵活。

