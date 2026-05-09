# 用 CSS 优化一个笨拙的表单元素

*建议*内容类型关联了*图书元素*词汇表。这个词汇表有十几个术语，只能选择其中一个，默认以垂直列表形式显示为单选按钮。这大大增加了用户提交建议时的滚动量。

为了解决这个问题，（当然）有一个模块可以做到，而且它的名字非常霸气：多列复选框单选按钮（`drupal.org/project/multicolumncheckboxesradios`）。然而，在撰写本文时，该模块在 Drupal 7 上仍有缺陷。此外，*图书元素*词汇表的术语数量还没多到需要分列显示的程度；水平排列看起来和垂直排列一样好，而且你可以用 CSS 来实现这一点。

![images](img/square.jpg) **注意** Drupal 7 改进了添加到表单元素 `<div>` 周围的类。在 Drupal 6 中，它被硬编码为 `form-item`。现在，表单元素名称和表单元素类型会作为 `form-item-name` 和 `form-item-type` 添加。你可以通过 `api.drupal.org/theme_form_element` 了解 Drupal 是如何做到这一点的（也能看到如果要更改，可以覆盖的主题函数）。

遗憾的是，你无法轻易地向已定义或修改的表单中添加自定义类。不过，通过基于类型和名称的类，你通常可以按需使用 CSS 或 JavaScript 来定位表单元素。总的来说，你无需额外添加 `<div>` 或其他包裹元素，也无需覆盖 `theme_form_element()` 函数，就能设置表单样式；参见代码清单 33–16。

**代码清单 33–16.** 为主题添加 CSS，使图书元素词汇表的单选按钮水平排列

```
/* 让图书元素单选按钮水平排列。 */
.form-item-field-element-und {
  display: inline-block;
  padding-right: 7px;
}
```



### 内容类型的上下文“新增”链接

当用户查看建议列表或单个建议时，如果她有权创建新建议，也应该被邀请提交自己的建议。

Drupal 7 提供了类似的操作，主要出现在管理页面，例如内容列表（`admin/content`）上方的 **+ 添加内容** 链接。

![images](img/square.jpg) **注意** 操作链接是 Drupal 7 的一种新的交互模式（参见第 32 章）。它们是 Drupal 表达“如果你在内容概览页面，很可能想要添加新内容”的方式。虽然 Drupal 中的其他本地任务会将你带到设置页面或列表页面，但操作链接执行操作，通常是添加某些内容。它们不会像其他任务那样默认渲染为选项卡，而是通过页面模板中的 `$action_links` 变量渲染为帮助区域正下方的链接。

就我个人而言，这些操作链接（例如 `admin/structure/content-types` 顶部的 *+ 添加内容类型*）对我来说几乎是不可见的——无论是由于轻微的缩进，还是更可能因为我习惯了 Drupal 6（该区域只有帮助文本），这个 Drupal 7 的新约定对我来说还不自然。然而，采用标准（无论如何，对于原生 Drupal 7 用户来说这可能会很自然）的好处超越了个人喜好。以特定方式处理特定类型链接的基本概念是合理的。

![images](img/square.jpg) **注意** 接下来描述的第一种方法并非将现有页面添加为操作链接的最佳方式，因此如果你在寻找直接答案，可以直接跳到第二种解决方案。我在此展示这种方法，是因为它是一种可行的解决方案（也是将新页面添加为操作链接的正确方式），更重要的是，它展示了探索的过程。

#### 寻找并遵循模型

如第 18 章所述，查看 Drupal 核心示例总是好的，但贡献模块实现核心约定的方式也是一个很好的参考——尤其是当贡献模块是 Views 时。在“管理 ![images](img/U001.jpg) 结构 ![images](img/U001.jpg) Views”页面（`admin/structure/views`），现有视图列表上方，是带有文本 *+ 添加新视图* 的标志性加号。你可以在 Views 代码中搜索该文本，以了解它是如何出现在那里的；参见代码清单 33–17。

***代码清单 33–17.** 使用 Grep 在 Views 项目中搜索“Add new view”，并显示结果*

```
cd ~/workspace/dgd7
grep -nHR "Add new view" sites/all/modules/views/
sites/all/modules/views/views_ui.module:38:    'title' => 'Add new view',
```

代码清单 33–18 中的搜索输出告诉你，在 Views UI 模块的第 38 行，你搜索的文本（“Add new view”）存在。转到该处，你可以看到它位于 `hook_menu()` 的实现中；参见代码清单 33–18。

***代码清单 33–18.** 在 views_ui.module 中定义“添加新视图”链接的代码*

```
  $items['admin/structure/views/add'] = $base + array(
    'title' => 'Add new view',
    'page callback' => 'views_ui_add_page',
    'type' => MENU_LOCAL_ACTION,
  );
```

这很有启发性。菜单项定义表明它出现的页面路径是 `admin/structure/views`，正如你所知。其**标题**是你看到的文本“Add new view”。特殊部分似乎是菜单项**类型**为 `MENU_LOCAL_ACTION`。

快速搜索 Drupal 代码中的 `MENU_LOCAL_ACTION`，可以找到其他示例；参见代码清单 33–19。

***代码清单 33–19.** node.module 中定义的菜单项，给出 `admin/structure/types` 上的链接*

```
  $items['admin/structure/types/add'] = array(
    'title' => 'Add content type',
    'page callback' => 'drupal_get_form',
    'page arguments' => array('node_type_form'),
    'access arguments' => array('administer content types'),
    'type' => MENU_LOCAL_ACTION,
    'file' => 'content_types.inc',
  );
```

然而，这并非需要添加到建议列表顶部的链接。你想要作为操作链接包含的链接是 `node/add/suggestion`。在 `node.module` 中搜索 `node/add`（跳过定义用户可创建*所有*内容类型列表页面的菜单项）会带你找到一组在 `foreach` 循环中定义的菜单项——每个内容类型一个菜单项；参见代码清单 33–20。

***代码清单 33–20.** node.module 的 `hook_menu()` 实现中，为每种内容类型创建 `node/add/CONTENT_TYPE` 页面的代码*

```
  foreach (node_type_get_types() as $type) {
    $type_url_str = str_replace('_', '-', $type->type);
    $items['node/add/' . $type_url_str] = array(
      'title' => $type->name,
      'title callback' => 'check_plain',
      'page callback' => 'node_add',
      'page arguments' => array($type->type),
      'access callback' => 'node_access',
      'access arguments' => array('create', $type->type),
      'description' => $type->description,
      'file' => 'node.pages.inc',
);
```

有没有办法让这个链接显示在其位置之外？似乎没有简单的方法，但实际上，你可以创建另一个不同路径的菜单项，该菜单项调用与你想要的节点添加表单相同的页面回调和页面参数。（稍后你会看到，这种方法并非最佳选择。）在这种情况下，这指的是你视图定义的 `suggestions` 路径和建议内容类型。

需要注意的最重要的事情是页面回调，即 `node_add()` 函数，以及页面参数，它只有一个参数，即节点类型的机器名。



