# 使用 `phpunit.xml`

创建测试套件的另一种方式是将它们添加到 `/phpunit.xml.dist` 文件中。当我们通过 `composer` 安装 `cakephp` 时，会自动创建该文件。默认情况下，PHPUnit 会在你运行它的目录中查找名为 `phpunit.xml` 或 `phpunit.xml.dist` 的文件，并使用其中的值来改变自身的行为。在 CakePHP 中，此文件位于我们应用的根目录中。

这个 XML 文件描述了测试的不同设置。其中有一个 `<testsuites>` 部分，你可以在其中定义新的测试套件。让我们将以下行添加为 `<testsuites>` 元素的新子元素。

```
2      src/Model/Table
3      src/Model/Table/CommentsTableTest.php
4      tests/TestCase/Controller/CommentsControllerTest.php
```

我们新的测试套件名为 `ExcitingFeature`，因为我们把那些与我们正在开发的新功能相关的测试都归组到了一起。

可以存在多个 `<directory>` 元素。PHPUnit 会递归地添加该目录中的所有测试。

可以存在多个 `<exclude>` 元素。这并不令人意外，但这些文件将被排除在外。

也可以存在多个 `<file>` 元素。这些文件将被添加到测试套件中。

## 总结

我将这简短的一章专门用于测试套件。你学会了如何使用 `TestSuite` 对象或 `phpunit.xml` 文件来对特定测试进行分组。

