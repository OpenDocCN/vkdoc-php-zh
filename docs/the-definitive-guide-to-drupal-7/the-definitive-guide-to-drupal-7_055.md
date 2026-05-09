# 构建可接受多个元素的表单

每个标签及其对应的替换标记集都需要在表单上拥有独立的位置，这意味着表单需要能够容纳数量可变的这类元素集。

## 懂得何时放手

最巧妙的方式是在需要时通过 AJAX 动态添加额外的表单元素集，它可以按需将 HTML 拉入页面。Drupal 的字段模块就是一个范例。

![images](img/square.jpg) **注意** 对于仅从章节标题无法判断的读者：本节不会进行任何开发工作。

遗憾的是，无限值字段使用的"添加另一项"链接是字段模块特有的。`modules/field/field.form.inc` 中 AJAX 回调 `field_add_more_js()` 及其相关功能的代码或许有参考价值，但 Drupal 7 的表单 API 没有提供自动化该过程的机制。

那么在构建模块的这个阶段该怎么办？简化处理。尽可能保持简单。现阶段不值得陷入复杂的用户界面增强中。（实际上，最好在首版中完全不做用户界面。这里之所以打破这一规则，是因为保存过滤器信息的常规方法不幸地不兼容 API；文本格式是作为一个整体保存的。）

![images](img/square.jpg) **注意** 从 Drupal 7 开始，每个过滤器实例都有自己的设置。也就是说，每种文本格式上的每个过滤器都是独立配置的：如果你在"过滤后的 HTML"文本格式上更改了图片缩放过滤器的设置，"完整 HTML"文本格式上的图片缩放过滤器设置不会发生变化。这极大地提升了灵活性，但同时也增加了保持共享过滤器设置一致的工作量。

## 制作始终接受两行额外内容的过滤器设置表单

设置表单将采用一种更简单的模式：首次呈现时及每次保存后，它始终提供至少两套空白的表单元素集。

设置回调函数返回的所有表单元素都会随过滤器对象一起保存，并可在 `$filter->settings` 中获取。过滤器对象（包括其设置数组）在所有过滤器回调（处理、准备、提示和设置自身）中均可用。你可以将任何所需的设置放入该数组的嵌套子数组中，例如将替换标记信息存入 `$filter->settings['rm']`。（作者曾考虑将每个标签及其替换标记对直接保存到设置数组中，但在生成表单时这种方法会导致混乱。）用于收集管理员替换标记信息的表单元素也应嵌套在 `'rm'` 数组中。

鉴于你需要为每个已保存的值提供一套表单元素，再额外增加两套空白元素，此时使用 `foreach` 循环是自然的做法。然而，无法向替换标记设置数组中添加两个空白标签数组，因为它们的键都是空字符串（`''`），会被合并为一个。为了避免在两个独立的循环中重复创建表单元素的代码，你可以将创建表单元素的逻辑抽离成独立函数，从而在需要时多次调用且无需重复代码。每套表单元素所需的三项内容包括：（结束）标签、替换其起始变体的起始标记，以及结束标记（参见清单 33–27）。

**清单 33–27.** 在可重复调用的函数中定义标签及替换标记的表单元素集

```
/**
 * 添加一组表单字段，用于新增标签和替换标记对。
 */
function _dgd7_tip_add_rm_formset(&$settings, $i, $tag = '', $replace =
  array('before' => '', 'after' => '')) {
  $settings['rm'][$i]['tag'] = array(
    '#type' => 'textfield',
    '#title' => t('标签'),
    '#maxlength' => 64,
    '#default_value' => $tag,
  );
  $settings['rm'][$i]['before'] = array(
    '#type' => 'textfield',
    '#title' => t('前置'),
    '#maxlength' => 1024,
    '#default_value' => $replace['before'],
  );
  $settings['rm'][$i]['after'] = array(
    '#type' => 'textfield',
    '#title' => t('后置'),
    '#maxlength' => 1024,
    '#default_value' => $replace['after'],
  );
}
```

这个函数做了几件有趣的事。主要是清晰地定义了三个类型为 `textfield` 的表单元素。它还接受一个迭代器（`$i`），从而可以按需多次将自身添加到 `$settings['rm']` 数组中，每次使用不同的整数键。`$settings` 数组通过引用传递（函数定义中 `&` 符号表明这一点），因此函数无需返回值；它直接修改了 `$settings` 变量。最后，函数为标签和替换标记接受了默认值，函数定义本身将这些值设为空，以便更容易添加空白表单字段。这正是 `_dgd7_tip_add_rm_formset()` 函数定义中 `$tag = '', $replace = array('before' => '', 'after' => '')` 部分的作用。

清单 33–28 展示了提供表单的函数：该表单包含若干行元素（用于编辑每个已保存的标签及其替换标记集）以及两行空白表单元素，供管理员添加更多的标签和替换标记集。

**清单 33–28.** 设置回调函数

```
/**
 * 标签过滤器的设置回调。
 */
function _dgd7_tip_settings($form, $form_state, $filter, $format, $defaults) {
  // 声明用于容纳设置表单元素的数组。
  $settings = array();

  // 获取默认设置。
  $filter->settings += $defaults;
  // "rm" 是替换标记的缩写。
  $rm = $filter->settings['rm'];

  $i = 0;
  foreach ($rm as $tag => $replace) {
    _dgd7_tip_add_rm_formset($settings, $i, $tag, $replace);
    // 将过滤器数量递增。
    $i++;
  }
  // 始终添加两套空白表单字段供填写。
  $total = $i + 2;
  for ($i; $i < $total; $i++) {
    _dgd7_tip_add_rm_formset($settings, $i);
  }
  return $settings;
}
```

`_dgd7_tip_add_rm_formset()` 函数在两个不同的循环中被调用。一个循环遍历所有现有或默认的标签及替换标记集（稍后会介绍默认设置的概念），另一个循环则在现有基础上额外添加两套空白字段集。`$i` 变量负责计数，确保每套字段拥有唯一的键。不过，这个整数键仅在通过表单收集数据时有效；在保存数据时，最好能将其去除。



### 使用验证函数在保存前处理数值

实际上，清单 33-28 中的代码并不完全有效：表单将使用其 `$i` 迭代整数值保存数据，这对于允许一次性保存多组数据是必要的，但在获取数据时，这个任意数值毫无意义。检索尝试假设 `$rm` 数组将标签作为键，而非数字。

以下两点让你能够在文本格式表单的过滤设置上下文中解决此问题（参见清单 33-29）：

*   通过设置 `#element_validate` 属性，可以为任何表单元素添加验证函数。
*   验证函数的功能不止于验证。它们可以使用 `form_set_value()` 更改将要保存的数据。

**清单 33-29.** 在设置回调函数中添加内容，为包含标签和替换标记集的表单元素设置验证函数

```
function _dgd7_tip_settings($form, $form_state, $filter, $format, $defaults) {
  // 声明将保存设置表单元素的数组。
  $settings = array();
  $settings['rm'] = array(
    '#element_validate' => array('dgd7_tip_rm_form_keys_validate'),
  );
  // [为节省空间，省略之前显示的其他代码...]
  return $settings;
}
```

经过大量试验，清单 33-30 中的测试数组结构似乎能以正确方式保存到设置中，并能在后续正常读取。

**清单 33-30.** 用于探索 Drupal 过滤 API 正确保存数据结构的试验函数

```
function dgd7_tip_rm_form_keys_validate($element, &$form_state) {
  $rm = array();
  $rm['{/testtag}'] = array(
      'before' => 'value for before markup',
      'after' => 'value for after markup',
  );
  form_set_value($element, $rm, $form_state);
}
```

试验过程是保存表单，并查看这些硬编码值是否按预期显示（标签为 `{/testtag}`，替换标记字段中的 `value for before markup` 和 `value for after markup`）。你绝不会用验证函数来硬编码值，但这提供了一种便捷的方式来测试用于保存数据的结构。显然，表单的其他所有部分都可以省去，因此我们也会这样做（参见清单 33-31）。

**清单 33-31.** 重组数据以标签为键保存、去除整数序列键的验证函数

```
/**
 * 在 filter_format_save() 运行前重新排列表单元素，使其以标签为键。
 */
function dgd7_tip_rm_form_keys_validate($element, &$form_state) {
  $rm = array();
  // 为每个元素创建以标签为键的版本。
  foreach ($element as $i => $value) {
    // 跳过非数值的表单元素（我们关心的元素都有数字键）。
    if (!is_numeric($i))  continue;
    $key = $value['tag']['#value'];
    // 不保存空键。
    if (!$key) continue;
    $rm[$key] = array(
      'before' => $value['before']['#value'],
      'after' => $value['after']['#value'],
    );
  }

  form_set_value($element, $rm, $form_state);
}
```

这段代码可能算不上优雅，但它提供了合理的数据存储，有助于完成即将进行的任务——一个与本模块直接用例更相关的任务，即在代码中提供默认值。不过，我们先来看一下验证的更常规用法。

### 验证过滤设置

为了使标签替换功能正常工作，模块需要确保提供的标签是包含 `/`（正斜杠）的结束标签。令人惊讶的是，文本过滤设置没有内置验证功能，这意味着没有现成的模型可供遵循。

你可以实现 `hook_form_alter()` 并添加一个全局表单验证函数，就像你完全自己定义整个表单时一样。更简单、更温和的方法是使用特定表单元素的 `#element_validate` 表单属性。

![images](img/square.jpg) **提示** 在 `api.drupal.org/forms_api_reference.html#element_validate` 阅读更多关于 `element_validate` 表单属性的信息。与往常一样，你也可以在核心代码中查找示例，例如在类 Unix 系统上使用命令行：`grep -nHR 'element_validate' modules/`

与大多数钩子和函数一样，用于验证元素的函数最重要的部分是它们的函数签名：`$element, &$form_state, $whole_form`。与号再次表示，尽管 `$form_state` 是一个数组，但它是通过引用传递给验证函数的，并且在函数内部所做的更改将应用于原始数据。

你需要验证待替换标签的具体信息是确认斜杠的存在。在网络上搜索“*php count number characters in a string*”（并点击一些结果）后，作者找到了 `php.net/substr_count`（参见清单 33-32）。（如果你认为你能猜到函数名，甚至接近正确名称，直接访问 `php.net/bestguess` 是查找函数的最快方法，因为它会自动提供一系列可能的匹配项。）

**清单 33-32.** 验证标签为可解析的结束标签

```
/**
 * 验证每个标签包含且仅包含一个斜杠。
 */
function dgd7_tip_rm_form_tag_validate($element, &$form_state, $whole_form) {
  if (strlen($element['#value']) && substr_count($element['#value'], '/') !== 1) {
    // 我们描述错误位置，因为提交后它很可能在一个不可见的垂直选项卡中。
    form_error($element, t('在“替换标记”过滤设置中，每个标签必须采用结束标签的形式，
且恰好包含一个斜杠 ("/")。起始标签是通过移除斜杠计算得出的。'));
  }
}
```

你的数据整理验证函数已经会丢弃标签文本字段中为空的替换标记表单数据，因此这个验证函数首先使用 `strlen($element['#value'])` 检查表单元素中是否有内容。如果没有，则不执行任何操作（不抛出错误）。`if` 语句的第二部分使用了 `substr_count()` 函数；如果斜杠数量不是恰好一个，则会抛出错误。



##### 在筛选器设置表单上添加说明

此模块应提供关于填写标签及前后标记字段的说明。通常的 Drupal 方式是在表单元素数组中使用`#description`，但这并不合适，因为您希望将所有字段作为一个整体来描述（而非逐一字段或字段集），并且希望说明位于其所描述的表单元素之前（而非像`#description`默认那样位于之后）。您只需要一些位于表单上方的文本。

网站已使用的图像调整筛选器模块（`drupal.org/project/image_resize_filter`）恰好具有这种分离的帮助文本。因此，可以很方便地借鉴其作者 Nathan Haug（quicksketch）的实现方式。查看`image_resize_filter.module`文件，您会发现他将帮助文本直接粘贴到了主题函数`theme_image_resize_filter_form()`中。

您可以基于这一思路进行改进，使其更优雅——仍然使用表单主题函数，但利用它来重新排布已在表单数组中正确定义的描述信息。第一步是实现`hook_theme()`以定义主题函数。第二步是向包含待描述表单元素的表单元素中添加两个属性：描述文本和使用主题函数的指令。第三步是定义该主题函数，使其在表单其余部分之前输出描述信息。清单 33–33 展示了该主题函数；请注意，大部分设置回调函数`_dgd7_tips_settings()`并未显示。

**清单 33–33. 实现主题函数，将描述信息置于表单元素顶部而非底部**

```
/**
 * 实现 hook_theme()。
 */
function dgd7_tip_theme() {
  return array(
    'dgd7_tip_settings' => array(
      'render element' => 'form',
    ),
  );
}

function _dgd7_tip_settings($form, $form_state, $filter, $format, $defaults) { // ...
  $settings['rm'] = array(
    '#description' => t('要设置标签和替换标记，请仅输入结束标签
  （例如 &lt;/tip&gt;）；起始标签将通过移除斜杠自动计算得出
  （此例中为 &lt;tip&gt;）。然后分别输入将替换起始和结束标签的前后标记。'),
    '#theme' => 'dgd7_tip_settings',
    '#element_validate' => array('dgd7_tip_rm_form_keys_validate'),
  );
// ...
}

/**
 * 主题回调函数，用于输出设置表单的描述信息。
 */
function theme_dgd7_tip_settings($vars) {
  $form = $vars['form'];
  return '<p>' . render($form['#description']) . '</p>'
         . drupal_render_children($form);
}
```

关于定义和使用主题函数的更多信息，请参见第 9 章。如第 14 章、第 15 章、附录 C 及本书其他部分所述，使用`render()`显示元素意味着该元素将不再被显示（除非仅渲染该元素或使用`show()`重新暴露它）。在显示表单其余部分时，需要使用`drupal_render_children()`而非`render()`，以避免无限循环。

#### 创建自己的钩子

接下来是有趣的部分。在开发 Drupal 时，您经常实现其他模块的钩子。创建自己的钩子算得上是难得的体验！这是您的模块回馈社区的机会，也是询问其他模块是否想加入“派对”的时刻。本次契机在于，当筛选器被新添加到某个格式时，需要添加标签和替换标记集。

创建钩子有点形而上学：如果一个钩子被定义了但无人实现，它是否存在？（如果这个问题让您夜不能寐，您之后可以自己实现这个钩子。）钩子的诞生，源于向其他代码提供响应调用的机会。最常用的调用方式，也是创建 Drupal 钩子的方法，是使用`module_invoke_all()`函数，如清单 33–34 所示。

**清单 33–34. 调用钩子，让其他模块有机会为筛选器提供默认设置**

```
/**
 * 实现 hook_filter_info()。
 */
function dgd7_tip_filter_info() {
  $filters['dgd7_tip'] = array(
    'title' => t('替换标记'),
    'description' => t('允许使用简单标记来表示需要包裹在自定义标记中的文本段落，
  例如用于强调提示、注释或其他特色插入内容。'),
    'process callback' => '_dgd7_tip_process',
    // 允许其他模块声明默认的标签和替换标记。
    'default settings' => array(
      'rm' => module_invoke_all('dgd7_tip_defaults'),
    ),
    'settings callback' => '_dgd7_tip_settings',
    'tips callback' => '_dgd7_tip_tips',
  );
  return $filters;
}
```

`module_invoke_all()`函数旨在从多个来源获取数据并将其整合。它使用 PHP 函数`array_merge_recursive()`来实现这一点，因此任何包含新键的内容都会添加到它返回的数组中，而任何包含相同键的内容则会覆盖之前已有的数据。对于替换标记，如果有两个模块实现了该钩子并为同一个短标签提供了标记，则最后被调用的模块会胜出。这是钩子工作机制的常见特性，您无需担心。

通常，在调用钩子时，即使想不出使用它的理由，也应传递一些上下文信息。在本例中，虽然没有任何有意义的上下文可以传递，但如果您维护的是公开模块，请留意问题队列：永远要假设别人会用您无法想象的方式，对您的 API 进行更古怪的操作。

请注意，此解决方案结合了代码提供的默认设置和管理员覆盖或新增的设置，但其灵活性不如 CTools 等真正的可导出配置。如果此模块获得了可观的使用量，实现这一功能将留给作者或您作为后续练习。

#### 过滤内容

您做这一切不就是为了一个目的吗？哦，是的！将用户输入的内容在显示时以不同格式呈现。要将标签转换为替换标记，您需要实现筛选器的处理回调函数，如清单 33–35 所示。

**清单 33–35. 替换标记文本筛选器的处理回调函数**

```
/**
 * 标签筛选器的处理回调函数。
 */
function _dgd7_tip_process($text, $filter) {
  if (!isset($filter->settings['rm']) || !is_array($filter->settings['rm'])) {
    return $text;
  }
  foreach ($filter->settings['rm'] as $ctag => $replace) {
    dgd7_tip_replace_tags($text, $ctag, $replace['before'], $replace['after']);
  }
  return $text;
}
```

此函数的第一部分检查是否存在任何需要应用的替换标记；如果没有，则提前退出，直接返回未修改的文本。（虽然理论上不应该存在未设置的设置，但稍微宽容一点也无妨。）

![images](img/square.jpg) **注意** 请记住，如果处理回调函数未返回任何值，文本不仅不会原样显示，还会完全消失。


好的，作为高级文档工程师和翻译员，我将严格按照您提供的注意事项和示例，将给定的英文文本翻译成中文。


