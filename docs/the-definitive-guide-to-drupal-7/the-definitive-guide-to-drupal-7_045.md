# 获取用户名

之前用到的 `dgd7glue_drupal_page_title()` 函数需要名副其实，即从 `drupal.org` 和 `groups.drupal.org` 的用户个人资料页面中获取用户名。

即便再疯狂的想法，在 Drupal 中也可能有人已经为你探过路了。就本例而言，这位先行者是 Kevin Hemenway，他更为人熟知的名字是 Morbus Iff。他开发的 Bot 模块为 #drupal 以及其他 IRC 频道中的 Druplicon 提供了支持（参见第 9 章）。该模块可以配置为，当给定 Drupal 站点的 URL（例如 [`http://example.com/node/523`](http://example.com/node/523)）时，神奇地获取该节点的标题。（对于 `drupal.org` 上的议题，它还能获取项目、状态等信息。但对你当前的用例来说，有趣的是它能在没有特殊集成的情况下，抓取任意站点的标题。）

知道了这一点，为何不执行 `drush dl bot` 呢？即便你并不打算使用它——只是看看它的代码（参见代码清单 33–13）。

**代码清单 33–13.** `bot_project.module` 节选

```
/**
 * 监听 URL 或数字 ID，并回复相关信息。
 *
 * @param $data
 *   IRC 库准备好的常规 $data 对象。
 * @param $from_query
 *   布尔值；表示是否是通过查询发起的请求。
 */
function bot_project_irc_msg_channel($data, $from_query = FALSE) {
// [与当前目的无关的代码未显示...]
      $result = drupal_http_request($url);
      if ($result->code != 200) { continue; }

      // 我们总是会显示标题，因此先抓取它以存入数据库。
      preg_match('/<title>(.*?) \|.*?<\/title>/', $result->data, $title_match);
      $title = $title_match[1] ? $title_match[1] : '<' . t('无法确定标题') . '>';
// ...
```

代码中充斥着模块作者 Morbus 的告诫，指出这并不是最佳做法——但它确实能用。你可以直接采用它，如代码清单 33–14 所示。

**代码清单 33–14.** 从 `drupal.org` 用户页面标题（或任意 `drupal.org` 页面标题）中抓取用户名的函数

```
/**
 * 获取 Drupal 站点的页面标题。
 *
 * dgd7glue_field_formatter_view() 中账号链接标题的回调函数。
 */
function dgd7glue_drupal_page_title($account_id, $href) {
  $result = drupal_http_request($url);
  // 如果无法获取标题，则使用 $account_id 作为标题。
  if ($result->code != 200) {
    return $account_id;
  }
  // 从页面的 HTML 源码中提取标题的第一部分。
  preg_match('/<title>(.*?) \|.*?<\/title>/', $result->data, $title_match);
  $title = $title_match[1] ? $title_match[1] : $account_id;
  return $title;
}
```

这段代码能工作，而且相当不错，它用名字替换了数字。不过，它仍然有一个问题：每次有人查看作者的资料页时，都会向 `drupal.org` 发起两次请求，每次都会从 `drupal.org` 或 `groups.drupal.org` 下载一整页完整的个人资料页面。无论如何，这个名字需要在本地缓存起来。

![images](img/square.jpg) **提示** 在实现缓存之前，请先确认你是否真的需要它。你可以通过调试器、查询日志记录器（如 Devel 模块提供的，`drupal.org/project/devel`），或临时包含一个 `watchdog()` 日志命令（`api.drupal.org/watchdog`）来测试需要缓存的代码是否被调用。详见附录 B，了解如何查找需要优化的性能问题。

## 使用 Drupal 的默认缓存表缓存简单数据

为了善待 `drupal.org`，更不用说提升你网站的性能了，不要在每次查询用户名时都抓取整个页面。

在格式化器层面做这项工作，意味着对于 Field API 的内置缓存来说已经为时已晚。定义一个新的 `cache_*` 表，或者向通用的 `cache` 表中添加一行或多行数据，都是可行的方案。要实现一些基本的缓存，你可以先查看 `cache` 表，然后倒推回去。在站点代码中搜索 `cache` 表中的键，很快就能发现 `cache_set()` 和 `cache_get()` 是 Drupal 用来存取缓存数据的函数。搜索 `cache_set` 或 `cache_get`（例如在 Drupal 代码根目录下执行 `grep -nHR 'cache_get' modules`）可以找到大量示例。

![images](img/square.jpg) **注意** Drupal 的缓存函数会自动为你处理静态缓存，这很不错。在本例中可能不需要，但静态缓存意味着当数据通过数据库查询从缓存中取出后，在同一个页面请求中不会重复执行这个查询。（详情参见 `cache_get()` 调用的 `api.drupal.org/_cache_get_object`。）

以 `locale.module` 中对语言元数据的缓存为例，你可以将缓存功能集成到获取页面标题的函数中。与缓存相关的补充内容以粗体显示在代码清单 33–15 中。

**代码清单 33–15.** 为从标准 Drupal 站点获取页面标题的函数添加缓存

```
/**
 * 获取 Drupal 站点的页面标题。
 *
 * dgd7glue_field_formatter_view() 中账号链接标题的回调函数。
 */
function dgd7glue_drupal_page_title($account_id, $href) {
  $url = $href . $account_id;
  if ($cache = cache_get('dgd7glue:' . $url, 'cache')) {
    $title = $cache->data;
  }
  else {
    $result = drupal_http_request($url);
    // 如果无法获取标题，则使用 $account_id 作为标题，但不缓存它。
    if ($result->code != 200) {
      return $account_id;
    }
    // 从页面的 HTML 源码中提取标题的第一部分。
    preg_match('/<title>(.*?) \|.*?<\/title>/', $result->data, $title_match);
    $title = $title_match[1] ? $title_match[1] : $account_id;
    cache_set('dgd7glue:' . $url, $title);
  }
  return $title;
}
```

现在，加载页面的请求只会发起一次，之后都会从缓存中读取，直到缓存被清除——在生产站点上，这可能是几周之后的事（开发环境除外）。

