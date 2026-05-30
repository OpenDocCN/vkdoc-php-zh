# 排版后的文档

![images](img/square.jpg) **提示** 在第 18 章至第 20 章中创建的 X 射线模块（可从`drupal.org/project/xray`获取）会为你访问的每个页面提供页面回调和页面参数，并能直接给出`node/add/suggestion`的相关信息，而无需直接查看`hook_menu()`的实现。

现有的代码已经看得够多了。是时候动手写代码了！代码清单 33-21 展示了一个菜单项定义，它将`node/add`菜单项的定义与`MENU_LOCAL_ACTION`类型结合在了一起。

***代码清单 33-21.** 在自定义代码中定义本地操作菜单项*

```
/**
 * 实现 hook_menu()。
 */
function dgd7glue_menu() {
  $items = array();
  $items['suggestions/add'] = array(
    'title' => "添加建议",
    'page callback' => 'node_add',
    'page arguments' => array('suggestion'),
    'access callback' => 'node_access',
    'access arguments' => array('create', 'suggestion'),
    'file' => 'node.pages.inc',
    'file path' => drupal_get_path('module', 'node'),
    'type' => MENU_LOCAL_ACTION,
  );
  return $items;
}
```

![images](img/square.jpg) **注意** 菜单项定义的键名中*不要*使用下划线（例如 `'page arguments'`、`'page callback'` 和 `'access callback'` 等）。

清除缓存（并确保模块已启用），奇迹出现了！你的自定义操作链接已经就位。

**键盘与座椅之间的问题（PEBKAC）**

当笔者初次尝试时，操作链接并未显示。我清除了缓存，将其添加到`admin/`路径下（而非由 Views 创建的`suggestions`路径），当然又清除了一次缓存，但它依然没有出现。我查看了菜单表，发现记录确实存在。直到我决定直接修改正在参考的示例（即“添加内容类型”链接）时，才注意到自己遗漏了示例中最关键的部分：`'type' => MENU_LOCAL_ACTION`！

再说一遍，如果我能制作模块，你也能。

我还犯的另一个错误是试图在`'file'`中填写 node 模块的完整路径。这根本行不通，通过访问[`http://api.drupal.org/hook_menu`](http://api.drupal.org/hook_menu)，我的错误很快得到了纠正。所借用代码所在模块的路径必须单独在`'file path'`指令中列出。（node 模块的路径通过`drupal_get_path()`函数获取，笔者通过在`api.drupal.org`上搜索`'path'`找到了这个函数。）

有些事情就是很棘手，一个打字错误可能会花费大量时间来调试。通常，当事情变得困难时，这往往意味着 Drupal 可能提供了更好的方法。本例正是如此，请继续阅读。

## 发现并采用更好的方法

这种为操作链接定义新菜单项的方法是可行的，但有没有更好的方法呢？在新的菜单项中重复使用页面回调函数，意味着添加建议的页面现在在站点上存在*两个*路径：一个是预期的`node/add/suggestion`，另一个是新的`suggestions/add`。这会导致快捷方式模块混乱（允许同一页面被两次添加到快捷栏），也可能让网站使用者感到困惑，降低他们的舒适感和理解度。

在查看`node.module`对`hook_menu()`的实现时，你可能已经注意到`admin/content`顶部的*+ 添加内容*操作链接并不是作为`MENU_LOCAL_ACTION`定义的。在`node.module`的所有文件中搜索`"添加内容"`，只会找到`node/add`页面本身。它又是如何被添加到`admin/content`的呢？搜索`admin/content`几乎立刻就能找到答案：函数`node_menu_local_tasks_alter()`。请查阅`node.module`或访问`api.drupal.org/node_menu_local_tasks_alter`，因为你可以直接获取并修改代码，将其变成自己的，就像代码清单 33-22 中那样。

***代码清单 33-22.** 通过实现 hook_menu_local_tasks_alter() 添加操作链接*

```
/**
 * 实现 hook_menu_local_tasks_alter()。
 */
function dgd7glue_menu_local_tasks_alter(&$data, $router_item, $root_path) {
  // 在 'suggestions' 页面上添加指向 'node/add/suggestion' 的操作链接。
  if ($root_path == 'suggestions') {
    $item = menu_get_item('node/add/suggestion');
    if ($item['access']) {
      $data['actions']['output'][] = array(
        '#theme' => 'menu_local_action',
        '#link' => $item,
      );
    }
  }
}
```

太棒了！这用更优雅的方案替换了之前定义的菜单项（感谢 Drupal，“总有钩子适合它”）。现在你已经用正确的方式为现有页面添加了自定义操作链接。懂得方法后，一切就变得简单了。但如果要从笔者跌跌撞撞的经历中吸取什么教训的话，那就是：不知道却勇于尝试，最终会变成知道怎么做。

在 Drupal 中，由 Views 创建的、展示一种或两种内容类型的列表非常常见。在列表顶部添加用于创建相同内容的链接，能极大地提升用户体验。现在，你知道了如何用十几行代码实现这一点。

## 制作自定义文本过滤器

Drupal 的文本格式过滤器是一种相当简单且强大的方式，可以改变内容的显示方式。（Drupal 的旧版本称其为*输入过滤器*，这容易引起误解，因为 Drupal 始终尊重用户提交的数据，是在*输出*时过滤内容。）在本节中，你将再次看到制作一个模块是多么简单、不吓人且实用，即使对像笔者这样在本章中容易开错头的人来说也是如此。

*《Drupal 7 权威指南》* 以及其他 Apress 的书籍，通过将提示、注释和其他类型的评论内容单独隔开（放在两行之间并使用不同字体）来突出显示。

![images](img/square.jpg) **提示** 在任何项目开始时，你都不会确切知道该如何做，但知道它总能有办法完成，是着手解决并付诸实践的关键第一步。

对于`DefinitiveDrupal.org`网站，你可以使用 HTML 和 CSS 来实现类似的效果。使用`div`和`span`配合 CSS 进行样式调整的 HTML 可能如下所示：

```
<div class="featured-element tip"><span class="featured-element-type">![images](img/U002.jpg)
<span class="leading-square">提</span>示</span> 手动输入包含![images](img/U002.jpg)
 div、span、类或 ID 的 HTML 代码，强烈说明我们做错了。</div>
```

然而，你并不希望作者每次需要高亮提示时都手动输入 HTML 代码。除了繁琐之外，这还会增加因微小错误而导致显示不一致的风险。与其输入前面的 HTML，不如让作者使用一种伪标记，它能够被替换为前述的 HTML，例如：

```
[tip] 手动输入包含 div、span、类或 ID 的 HTML 代码，强烈说明我们做错了。[/tip]
```

这样出错的可能性小得多。你清楚自己要做的事，有了想用的简化标记，以及希望从中生成的 HTML。那么，该从哪里开始呢？