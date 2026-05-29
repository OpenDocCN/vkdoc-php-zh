# 测试回调

你可能已经注意到有些函数是由`bake`自动生成的。这些是测试回调。CakePHP 的单元测试包含以下回调方法：

*   `setUp`在每个`test`方法之前调用，因此这是初始化通用对象的最佳位置。应始终调用`parent::setUp()`。

*   `tearDown`在每个`test`方法之后调用。不要忘记调用`parent::tearDown`。

`tearDownAfterClass`在测试用例中的测试方法启动后调用一次。此方法必须是`static`的。