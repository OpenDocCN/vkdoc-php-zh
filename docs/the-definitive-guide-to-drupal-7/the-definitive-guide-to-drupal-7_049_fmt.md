# 选择方法

决定编写模块当然只是第一步，如何编写同样至关重要。

在阅读完本书关于模块开发的部分，了解了钩子和节点相关知识后，你可能会想在节点保存时通过`hook_node_insert()`或`hook_node_update()`对其进行截获并修改。但你应该抵制这种诱惑。Drupal 的一个显著特性就是它不会直接修改内容本身。你保存前看到的，与再次编辑时看到的内容完全一致。这意味着你的数据永远不会被破坏。认识到这是个优点后，你可能会想通过`hook_node_view()`中的处理，用你酷炫的样式替换占位标记。但这意味着 Drupal 每次显示节点时都必须执行文本处理工作。在构建一种操作现有文本并添加新缓存层的机制之前，不妨退一步，看看节点系统之外的可能性。（或者，如果实在无解，可以像第 9 章中讨论的那样，在 IRC 上提问。）

![images](img/square.jpg) **注意** 要了解 Drupal 7 中所有与节点相关的钩子，请参阅`api.drupal.org/node.api.php`，或打开任意 Drupal 7 副本中的`modules/node/node.api.php`文件。

如何改变用户输入文本的显示方式，是 Drupal 中的一个常见问题。事实上，Drupal 核心早已解决了这个问题。自 Drupal 5 起，用于管理内容显示时修改的方法就存在于 Filter 模块中。

在 Drupal 7 中，Filter 模块在管理界面中的路径为：管理 ![images](img/U001.jpg) 配置 ![images](img/U001.jpg) 内容创作 ![images](img/U001.jpg) 文本格式（`admin/config/content/formats`）。从代码层面（位于`modules/filter`目录）查看这个核心模块，会发现十个文件（`filter.admin.inc`、`filter.css`、`filter.js`、`filter.test`、`filter.admin.js`、`filter.info`、`filter.module`、`filter.api.php`、`filter.install`和`filter.pages.inc`），这似乎有点令人望而生畏。我们来看看它，但最好能找到一个仅实现过滤器提供功能、而非整个文本格式系统的模块。