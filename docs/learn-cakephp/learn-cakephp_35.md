# 12. 测试套件

![A337704_2_En_12_Figa_HTML.jpg](img/A337704_2_En_12_Figa_HTML.jpg)

这套衣服真的很适合我

在开发特定功能时，我们会运行某些测试。在部署之前，我们应该运行所有测试，以确保没有破坏任何东西。这就是测试套件发挥作用的地方。

## 使用 TestSuite

`TestSuite` 提供了一些方法，可以基于文件系统轻松创建测试套件。

要为所有模型表格测试创建一个测试套件，我们需要在 `/tests/TestCase/` 目录下创建一个名为 `AllModelTableTest.php` 的文件。

```
1  <?php
2  use Cake\TestSuite\TestSuite;
3
4  class AllModelTableTest extends TestSuite
5  {
6      public static function suite()
7      {
8          $suite = new TestSuite('所有模型表格测试');
9          $suite->addTestDirectory(TESTS . 'TestCase/Model/Table');
10         return $suite;
11     }
12 }
```

上述代码将 `/tests/TestCase/Model/Table` 文件夹中的所有测试用例组合在一起。

现在，你可以一次性运行所有模型表格测试。

```
$ cd ∼/public_html/cakeBlog
$ vendor/bin/phpunit tests/TestCase/AllModelTableTest.php
```

你可以用同样的方式为所有控制器等创建测试套件。

如果你只想添加几个文件，可以使用 `$suite->addTestFile($filename)`。

你可以通过 `$suite->addTestDirectoryRecursive(TESTS . 'TestCase');` 递归添加 `/tests/TestCase` 下的目录，这样可以一次性运行所有测试。别忘了，直接使用 `vendor/bin/phpunit` 也能达到同样的效果。



