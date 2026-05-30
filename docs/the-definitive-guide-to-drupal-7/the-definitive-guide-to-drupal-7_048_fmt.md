# 寻找能满足需求的模块

最好的模块是你无需亲自编写的模块。不妨先四处寻找一下，看有没有现成的模块能处理从其他标记语言生成 HTML 标签这类任务。通过在线搜索诸如 "drupal text format transform tags"、"drupal replace markup"、"drupal input filter tags"、"drupal 7 text filters exportable" 等关键词，并专门在 `drupal.org` 模块库中搜索类似关键词（去掉 "Drupal" 一词后），确实能找到一些已有的实现。

![images](img/square.jpg) **提示** 在 Drupal.org 搜索模块时，记得应用模块筛选器。对应的链接可能类似：`drupal.org/search/apachesolr_multisitesearch/replace%20tags?filters=ss_meta_type%3Amodule`

与 `DefinitiveDrupal.org` 的需求类似，Markdown Filter（`drupal.org/project/markdown`）和 Textile（`drupal.org/project/textile`）模块（从文本处理的角度看）以及 BBCode（`drupal.org/project/bbcode`）模块，都是针对已知的标记系统设计的，而非满足自定义需求。这些模块以及 Typogrify（`drupal.org/project/typogrify`）等其他模块，都可以作为创建过滤器的参考示例。

搜索结果中还出现了另一个模块：SimpleHTMLDOM（`drupal.org/project/simplehtmldom`），它是同名库（可从 `simplehtmldom.sourceforge.net` 获取）的一个封装。这个模块作为工具可能有用，但你的文本操作需求并没有那么复杂。

确实存在一个用于标签替换的 Drupal 6 版本模块，其中包含一些看起来能实现你所需前后标签替换的功能：Rep[lacement] tags（`drupal.org/project/reptag`）。然而，它并未使用过滤器及文本格式系统，而是依赖 NodeAPI。此外，该 Drupal 6 版本从未发布过稳定版。这应该能让你在决定复制（而非移植并扩展）一个模块时，心理负担小一些。

这些标记替换需求也完全有可能通过配置来实现，或者作为 Flexifilter 模块（`drupal.org/project/flexifilter`）的一个子模块来完成——该模块旨在让创建自定义过滤器变得更简单。在相同思路下的另一个项目 Custom filter（`drupal.org/project/customfilter`）存在时间更久，维护也更积极。不过截至撰写本文时，这两个模块都还没有 Drupal 7 分支，也未能提供足够的吸引力，让人愿意采用先移植再构建子模块的方式。

鉴于时间紧迫、目标明确且具备编程能力，此时自行编写模块是合理的。（这可能不是最明智或最佳的决定，但至少表面上来看并非糟糕透顶的主意。）