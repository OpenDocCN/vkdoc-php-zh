# 编写一个主要模块

作者：Benjamin Melançon

> *“你需要我，我也需要你 / 没有了彼此，人们什么都做不了。”*

——艾瑞莎·弗兰克林（《思考》）

“模块”的字典定义是“多个不同但又相互关联的部分之一，可用于构建更复杂的结构。”这恰好描述了如何用 Drupal 构建复杂的网站：一个模块接一个模块。（你以为 Drupal 中有哪个东西的命名是明智的吗？）

模块，就其定义而言，从来不会独立存在，而是与其他模块共同构成一个运行的整体。Drupal 的模块化特性使得分别编写和维护的项目能够扩展 Drupal 的功能。

![images](img/square.jpg) **注意** 模块从来不会独立存在这一事实，根据 Drupal 协会法律顾问的解释，具有法律含义。Drupal 是根据 GNU 通用公共许可证（GPL）授权的自由软件。因为 Drupal 模块在定义上属于衍生作品，我们制作和分发的任何模块都必须允许他人查看、复制和修改（即修改他们自己的副本！）。为自己或单个客户构建的模块不被视为已分发，但值得注意的是，这种共享与协作的精神——以及其带来的各种实际益处——是有法律框架支持的。

在本章使用的术语中，基础模块是指扩展 Drupal 的模块，而主要模块则是指本身旨在被扩展的模块。主要模块是模块化的（准备被修改或扩展），是 Drupal 模块化特性的一种映射。按照这个定义，LoginToboggan (`drupal.org/project/logintoboggan`) 是一个基础模块，因为它改变了 Drupal 的工作方式；而 Advanced help (`drupal.org/project/advanced_help`) 是一个主要模块，因为它为其他模块提供了接入点。对于后一类模块，更常见的术语是 API 模块，但在本章中，这个术语特指纯 API 模块。

一个主要模块拥有供其他模块使用的 API。回想一下，API 代表应用程序编程接口。它是代码之间进行沟通的方式（当它们遵循规范并通过官方渠道时）。钩子的调用和实现（例如第 19 章中描述的那些），加上我们模块可用的实用函数，共同构成了 Drupal 的 API。

我们将在本章开始构建的模块将允许其他模块扩展它，就像它扩展 Drupal 7 一样。该模块名为 Form messages，其目标是实现表单元素的即时内联验证。（最初命名为 AJAX form messages，但在开发过程中发现仅使用 AJAX 调用并非最佳方案，因此重新命名。）它将修改表单，使其能够显示由 API 和配套用户界面提供的内联通知和错误信息。其他模块将能够使用 Form messages API 来提供错误消息和内联验证例程。因此，这个模块将同时拥有输入和输出的 API：利用 Drupal 的 API 在 Drupal 基础上构建（与每个模块一样），并定义自己的 API 以便其他模块在此基础上构建。我们不会在本章中完成这个模块，甚至不会接近完成（尽管相关工作和文档将在 `dgd7.org/strategy` 上继续）。相反，我们将为一个主要模块奠定基础。

本章有两个主要目标：

*   介绍一种模块开发策略（以及大量技巧），帮助你为构建任何提供 API 的模块做好准备。
*   涵盖 Drupal 7 开发中核心的一些概念：实体和字段。

### 如何不构建模块

这一节并非一个最坏情况下的模块制作灾难故事（尽管负面例子非常有趣且富有启发性——关于一个跨越四个 Drupal 版本却依然适用的经典讽刺文章，请参阅 Nick Lewis 的《通往 Drupal 地狱之路》，`drupal.org/node/77487`）。不，本节的前提比那更简单：最好的模块是你无需构建的那个。

当你着手构建一个模块时，你需要尽职尽责地搜索是否有现成的模块能满足你的需求。你需要确保自己没有重复他人的工作。如果在搜索了 `drupal.org` 和万维网之后，你还没有找到能实现你目标的现有模块，那么请访问 `groups.drupal.org`（Drupalistas 常简称为 g.d.o）并加入“贡献模块创意”小组，`groups.drupal.org/contributed-module-ideas`。在这里你可以发布讨论（地址为 `groups.drupal.org/node/add/story?gids[]=5445`）——顺便提一句，你可以从这个 URL 看出，g.d.o 上讨论的机器名称是 story，而贡献模块创意小组的节点 ID 是 5445。如果你没有设置通知，请记得经常回来看你的帖子！也可以去 IRC 上询问，包括在 `#drupal-contribute` 频道，了解你所针对的问题领域存在哪些模块。

![images](img/square.jpg) **提示** 如果你发现类似的项目已经开发完成或正在进行中，请尽可能与其他开发者合作。模块重复不仅浪费你作为开发者的时间，还会让用户和贡献者更难选择使用什么或把精力投入何处。更多信息请参见 `drupal.org/node/23789`。

针对 Form Messages 模块，我搜索了 *Drupal AJAX form messages*、*drupal AJAX form validation* 和 *drupal inline form validation* 以及其他变体，都没有找到类似的模块或项目。我在贡献模块创意小组中发布了该模块的目标，并交叉发布到了表单 API 和可用性小组（参见 `groups.drupal.org/node/113564` 上的帖子）。

最后一次尝试是在 `#drupal-contribute` 中提问：“在 Drupal 中，对于填写表单时提供即时内联反馈方面，已经完成了哪些工作（如果有的话）？（理想情况包括节点表单上的字段组合。）”你当然可以提出比这更好的问题，但请记住一条经验法则：任何可以用“是”或“否”来回答的问题都不是你能提出的最佳问题。（更多 IRC 技巧和礼仪，请参见第 9 章）。在 IRC 上提问就像在黑暗中开枪——即使有人知道答案，他们也必须在线、看到问题，并且处于足够清醒的精神状态来构思回答。对于一个足够重大到可以成为一个主要模块，却还没有人着手研究的话题，这种提问更像是大海捞针。但这并不意味着它不值得去做。

仅仅是提问这个行为——即使没有人加入你一起集思广益——也能帮助你更好地理解自己试图做什么。你甚至可能开始回答自己的问题。在提问的过程中我想到，Drupal 核心中有一个地方可以在你填写表单时提供即时反馈：密码强度指示器。有一个核心模型可以遵循，这是非常好的事情。

之后，在构建模块的过程中，请向那些表示过兴趣的人（或任何可能感兴趣的人）报告你的进展。你的项目页面和问题队列是公开陈述计划、跟踪实现目标功能的进度以及与他人协作的好地方。关于后一点，不要指望在你有用的东西很久以前就有人参与。关于如何在 `drupal.org` 上设置沙盒项目，请参见第 37 章。



### 了解 Drupal 提供的工具

在任何项目中，充分掌握你所能使用的工具，都会让你拥有更好的开端（并最终取得更好的成果）。第 21 章介绍了 Drupal 中一些常用的 API，实际上还有更多。

> *“如果你试图用‘虫眼视角’（与鸟瞰视角相对）来摸索 Drupal，可能会浪费掉一整个漫长周末。”*  
> ——杰夫·伊顿

第 34 章将提供这种“虫眼视角”的一部分。你可以通过设置调试器，观察 Drupal 在执行不同操作时如何一步步遍历其代码，来继续深入探索。

了解 Drupal 提供的能力，有助于判断你是否正试图让 Drupal 做它并不擅长的事情。这是任何项目的第一步：评估潜在工具。就像使用 Drupal 做事有多种方法一样，不使用 Drupal 也有很多途径。当然，如果涉及内容、用户、权限、在网站上展示内容等诸多方面，你可能确实需要 Drupal。

Drupal 的核心本质是一个 API 集合。Drupal 架构师杰夫·伊顿强调的并非 Drupal 提供钩子的模块，而是首先关注那些驻留在 `includes` 目录中的所有实用功能。Drupal 的 `include` 文件定义了用于处理以下事务的函数：

- 菜单（路由）
- 数据库抽象层
- 会话管理
- 缓存
- 文件存储与流包装器
- 本地化与语言
- 主题（渲染）
- 表单与表单处理
- 日期处理
- 图片处理
- 分页与表格排序
- 批处理
- 令牌
- 电子邮件
- 实体
- 模块系统
- XMLRPC
- AJAX
- Unicode 及其他通用工具
- 更新
- ……等等

没有人能自动记住所有这些辅助库（顺便说一句，`includes/graph.inc` 提供了一个函数，用于对有向无环图进行深度优先搜索），更不用说全部掌握它们的用法了。但它们是我们的基础工具箱，在我们开始启用或下载模块之前就始终存在。当然，Drupal 的核心模块提供了额外的关键功能，例如用户处理和输入过滤。贡献模块可以提供自己的实用函数和 API，有些模块甚至*仅仅*是为其他模块提供工具。`CTools` 和 `VotingAPI` 就是纯粹的 API 模块。

### 你的模块应该提供 API 吗？

通常来说，你制作的任何模块都应该为那些可能想要与你的模块协作或在其基础上构建的模块提供一个 API。同时，一个设计良好的 API 通常很难做到；随着其他模块尝试使用它，往往需要对其进行修订或扩展。

如果你的模块很简单，并且已充分使用了 Drupal 的 API，那么它就不需要创建自己的 API。然而，如果你的模块为设置创建了独立的用户界面，那么它至少应该始终为这些设置提供一个配套的 API。

#### 保持 API 与 UI 分离

模块中通过提交表单（用户界面）完成的任何操作，也应该能够通过一行代码实现。事实上，处理表单的提交函数应始终使用 API 函数来保存更改。

如果在提交函数中直接进行数据库查询，那你就做错了。在表单提交函数中，你绝不应看到 `db_insert()`、`db_update()`、`db_delete()`、任何 `db_` 开头的函数，或者 `drupal_write_record()`。这意味着，保存、更改或删除表单所影响的信息只能通过用户界面完成——或者通过重新创建这些函数、伪造表单提交来实现。这会使其他模块更难、更丑陋地与你的模块所提供的服务和信息进行交互。（在表单验证中修改数据库则更糟糕。）相反，表单提交函数应将表单的数据传递给一个 API 函数，该函数与表单提交流程清晰解耦。（遗憾的是，Drupal 核心在这方面尚未形成一致的模型。）

保持 API 与 UI 分离的终极体现，是将用户界面放在项目中一个独立的模块中，该模块可以关闭。Drupal 核心的字段模块由字段 UI 模块作为补充，后者可以禁用。视图模块同样也有一个可以禁用的视图 UI 模块。

#### 使用 API 隐藏复杂性

其他模块基于你的模块构建的好处在于，该模块的作者无需考虑你的模块所做的所有事情。对于那些其全部意义就在于提供通用功能的模块（即 API 模块）来说，这一点尤为突出。杰夫·伊顿在 DrupalCon DC 大会上讲述了 `VotingAPI` 模块（`drupal.org/project/votingapi`）的故事。当时他正在开发自己早期的较大型网站之一，需要投票功能。他找到了一些很酷的投票模块，但它们之间无法协同工作。“于是，我选了那个拥有最漂亮 Flash 组件的模块，然后毫不客气地掏空了它的内核，”他说。

`VotingAPI` 就此诞生，它包含两个用于获取和设置投票的函数。在询问其他维护投票模块的人“能否毫无理由地让我的模块成为你的依赖项？”之后，有一个人接受了他的提议。他立刻不得不增加了三个函数，以满足该模块更精细的需求。

“我们第一次总是会搞错，”伊顿说道。第二次也是，第三次也是。经验法则是，API 需要至少经过三个实现案例（即它们被用于完成特定任务的场景）的测试，你才能期望它们具有通用性。

当你的 API 能够处理三种不同的用例时，它很可能已经隐藏了大量的复杂性。

![images](img/square.jpg) **提示** 雇佣型模块开发——受雇开发模块——可以非常有趣，尤其是与好的客户合作时，因为需求已经为你明确和规划好了。即使需求再好，一个模块（如同一个网站）也会在构建和根据实际需求测试的过程中不断演变。当你试图开发一个对许多人都有用的模块时，请将客户的规范视为*众多*用例之一——并且不要指望你的所有开发时间都能得到报酬。制作一个通用的模块总是比一次性开发更费力，但从长远来看，你、你的客户以及社区都能从中受益。

### 让你的模块更模块化

为其核心功能提供 API 是让其他模块可扩展你的模块的最重要方式，但并非唯一方式。

如同任何大型模块一样，让“表单消息”模块模块化有很多种方法。几乎所有方法的共同思路，就是将功能分离成封装好的片段，使你的模块在内部与其自身的 API 协同工作。当你代码的某些部分通过通道与代码的其他部分通信时，这些相同的通道也同样适用于其他模块中的代码。



#### 释放模块中钩子的力量

正如你的模块会使用 Drupal 提供的钩子来改变其行为（例如使用`hook_menu()`在某个 URL 显示页面或执行操作），也很可能使用其他贡献模块提供的钩子（例如使用`hook_views_default()`提供默认视图），你的模块也可以提供钩子供其他模块实现。

表单消息（Form Messages）的主要 API 将是为模块提供定义消息的能力。这可能表现为定义机制，让模块能够以`hook_default_views()`允许模块提供默认视图的相同方式，来提供默认消息。表单消息的这个基础钩子将在本章的在线配套资源`dgd7.org/strategy`中介绍。需要注意，在定义钩子时，你应该在随模块一同提供的`modulename.api.php`文件中记录每个钩子的示例实现。

除了定义完整的钩子，还有更轻量级的方式让其他模块接入你的模块。每当你的模块收集数据数组时，都可以给其他模块操作这些数据的机会。最简单的方式就是使用令人愉悦的`drupal_alter()`函数。通常，模块会先定义自己的钩子让其他模块提供信息，然后使用`drupal_alter()`创建一个修改钩子，在所有数据收集完成后允许其他模块进行修改。

![images](img/square.jpg) **注意** 在现有模块中添加一行`drupal_alter()`语句的补丁，通常是让你在其他人的模块中实现所需功能的最简单方式。添加一个满足他人需求且得到模块维护者认可的重大功能可能遥不可及，但说服维护者提交一个让你能基于自己模块功能进行构建的改动，则应该更容易实现。

当然，我们也有极具说服力的、彻底自私的理由来开放模块，允许大众参与干预。精心放置的钩子调用能让其他人替我们完成艰苦的工作。我们的模块越是以这种方式构建，就越能像 Jeff Eaton 那样（停顿一下，扶正他想象中的领带）说：“解决那个问题不是我的事；我只维护 API。”

在构建纯 API 模块时，专注于维护 API 并使其能够应对众多使用场景——但不在你的模块中解决它们——这一点尤为重要。Jeff Eaton 在结束他的 API 模块演讲时敦促我们：“保持专注——把一件事真正做好。Drupal 正朝着协同工作的方向发展；关键在于让它们协同得非常好。”这就是著名的 Unix 哲学。Unix 管道发明者、Unix 传统奠基人之一 Doug McIlroy 这样总结：“编写只做一件事且做到极致的程序。编写能协同工作的程序。”

![images](img/square.jpg) **提示** 在项目初期，尽早开始与你所在领域正在开展工作的其他人交流。

#### 渐进增强：在启用时利用其他模块

如果你的模块目的是扩展另一个模块，或需要调用其函数，你就必须将其列为依赖项。但大多数情况下，你的模块可以被其他模块增强，或者你的模块可以通过提供更多选择或功能来增强其他模块。例如，如果你的模块以任何方式存储数据，你可能希望将这些数据暴露给 Views 模块（通常通过使用 Drupal 的 Field API 即可免费获得此功能）。

尽可能让这些模块依赖成为可选。你不希望强迫用户除了安装你的模块外还要安装半打模块。相反，检查你要使用的特定函数是否存在，如果不存在则优雅降级。在此语境下，优雅降级意味着你的模块会失去一些额外功能，但不会崩溃，并且能继续提供基础功能。你的目标应当是把模块构建成具有条件性增强，而非依赖关系。

许多优秀的模块都设计为能够利用其他模块提供的功能。这最好被理解为渐进增强而非优雅降级，因为缺失的功能没有变通方案，只是在启用支持模块时简单地增加功能。

例如，Drupal 核心的菜单模块与区块模块集成，但并不要求依赖它。在`hook_help()`的实现中（该函数将当前页面的路径作为参数来执行相应操作），包含以下代码：

```
  if ($path == 'admin/structure/menu' && module_exists('block')) {
    return '<p>' . t('每个菜单都有一个对应的区块，可在<a href="@blocks">区块管理页面</a>上进行管理。', array('@blocks' => url('admin/structure/block'))) . '</p>';
  }
```

关键部分是`module_exists('block')`。只要区块模块启用，菜单模块就会将其每个菜单作为区块提供。只要帮助模块启用，你就能在菜单管理页面顶部看到这一提示。这两个利用其他模块的例子都是渐进增强的体现。

![images](img/square.jpg) **注意** 为什么有人不启用帮助模块？为什么有人不启用区块模块？模块作者无权评判站点构建者的选择。就区块模块而言，事实上它还没有（完全）达到某些 Drupal 站点所要求的强大和灵活程度。有些开发者转而使用面板模块（其自身包含调用区块的代码），其他人则使用上下文模块。面板模块和上下文模块都体现了 Drupal 的模块化特性——允许替换核心功能——并提醒我们，若非绝对必要，永远不要让我们的模块依赖另一个模块。

使用 Drupal 的钩子系统是一个模块对另一个模块正在执行的操作做出反应的最简单方式。

你可以提供默认视图并将数据暴露给 Views 模块。你可以为 PathAuto 和动态文本区域提供令牌，也可以提供 Drush 命令。你可以通过实现`hook_help()`（另一个由模块提供、但任何模块都不必实现的钩子）来提供基本的 Drupal 核心帮助。你还可以通过调用高级帮助模块的钩子来提供高级帮助文档页面；这些页面仅当高级帮助模块启用时才会被使用。



### 测试环境入门

启动一个主要模块时，为其搭建一个全新的测试环境是必要的。获取一份 Drupal 副本，以你正在开发的项目命名安装目录，并通过命令行进行安装。更多关于 Drush 的内容（正是它让这一切变得简单），请参阅第 2 章和第 26 章。以下是使用 `drush site-install`（`si`）命令创建测试站点的命令行步骤，并以我们正在开发的模块命名该站点。

![images](img/square.jpg) **注意** 关于命名——这是开始编写模块时首先要做的事情之一。模块名称，就目录和代码而言，必须仅包含小写字母、数字和下划线，并且必须以字母开头。建议名称具有描述性而非简短，如果是名词，则使用单数形式（以便与类和数据库表的推荐命名实践兼容）。我将模块命名为 `formmsgs`，同时违反了这两条建议，实属惭愧。不过，我并不为在模块名称中避免使用下划线而道歉；这是基于我对命名空间冲突的担忧（将模块命名为 `form_messages` 并实现 `hook_help()` 意味着，如果存在一个名为 `form` 的模块实现了潜在的 `hook_messages_help()`，则会因函数名重复而导致致命错误）。

```
cd ~/workspace
cd formmsgs
drush dl drupal --drupal-project-rename=formmsgs
drush si --db-url=mysql://root:rootpass@localhost/formmsgs
```

默认情况下，`drush site-install` 将使用标准安装配置文件，并赋予超级用户（用户 ID 1）用户名 `admin` 和密码 `admin`。

![images](img/square.jpg) **提示** 你可以进一步自动化这些步骤。关于自动化创建测试站点及其 Drupal 安装的示例，请参见 `dgd7.org/sh`。

接下来，为新模块创建一个目录；对于测试站点来说，`sites/default` 目录下即可。（如果 `sites/default` 中尚不存在 `modules` 目录，则 `-p` 标志会告诉 `mkdir` 先创建它，再创建 `formmsgs` 目录。）同时，立即为模块启动一个 Git 仓库，如下所示：

```
mkdir -p sites/default/modules/formmsgs
cd sites/default/modules/formmsgs
git init
```

在添加和编辑文件时，通过使用 `git add .`（用于添加新文件和已修改文件，句点表示添加所有可用的内容）和 `git commit` 命令，让 Git 记录你的所有更改。

#### 借用一些代码来起步

我通过从 Unique fields 模块借用代码来开始我的 `.info` 文件，因为我预计会与该模块紧密合作（或者至少从中学习很多），所以我可以同时进行学习和借鉴。

```
drush dl unique_field
cp ../../../all/modules/unique_field/unique_field.info formmsgs.info
gvim formmsgs.info
```

修改其中的内容。Info 文件很简单，不会有太多重复内容。但从模板开始是很好的做法，如果这个模板恰好也是你感兴趣的模块，那就更棒了。复制整个 `.module` 文件不是一个好主意，但查看示例甚至复制其中的部分内容则可以很好地工作。

```
name = AJAX form messages
description = "[formmsgs] Provides immediate, in-form validation."
core = 7.x
```

![images](img/square.jpg) **注意** 在描述中包含模块机器名是一个惯例（即使只有我偏爱这样做），这有助于用户在模块管理页面上搜索到该模块。

#### 在 Drupal.org 的沙盒中分享你的代码

我目前只有一个 `.info` 文件，但养成尽早且频繁地提交和分享的习惯是非常好的（参见清单 24–1）。分享项目的命令在第 38 章中讨论，并在你通过 `drupal.org/node/add/project-project` 创建项目后提供给你。

***清单 24–1.** 开始在你的 Drupal.org 上分享代码*

```
git add .
git commit -m "Initial commit for AJAX form messages module; the .info file."
git remote add origin mlncn@git.drupal.org:sandbox/mlncn/910490.git
git push origin master
git checkout -b 7.x-1.x
git push origin 7.x-1.x
```

清单 24–1 中针对你项目定制的步骤，可以在你项目的 Git 说明选项卡中找到。对于 Form Messages 模块，这是 `drupal.org/node/910490/git-instructions`，你的完整项目或沙盒项目的说明可以在相同的路径下找到（将 `910490` 替换为你项目的节点 ID）。

![images](img/square.jpg) **提示** 幸运的是，分享是双向的。关于代码的最佳实践，请参考核心模块以及被广泛开发和使用的模块，例如 Views、Token、Administration Menu、Date、Webform、Devel、Voting API 等（包括本书中涉及的其他模块，如第 25 章中的 Commerce 和第 31 章中的 Apache Solr）。然而要知道，随着你学到更多知识，你可能需要走自己的路并进行重大的修改。关于这一点，也请前往 `dgd7.org/strategy` 订阅 Form Messages 模块代码和方法的更新。

### 规划你的方法

我一直说我会提供一个 API，但这意味着什么？这意味着更多——在某些情况下，它意味着不同的事情——而不仅仅是简单地在另一个模块中拥有用户界面。将某物视为 API 的一种方式是：你的模块什么也不做。它仅仅是存在。Form Messages 的 API 将完全基于其他模块告诉它要做的事情，来处理与表单的交互。

这意味着我必须将显示消息所需的内容提炼为其基本组成部分。你不能假设你的模块有任何方法可以自行知道何时或以何种方式应该做某事。

那么，以下是 Form Messages 模块必须向与之配合的模块提出的一些问题。为你自己的模块提出这些问题，将告诉你 API 模块必须存储和处理哪些数据，以及它如何与这些其他模块交互——简而言之，它需要什么样的 API。

-  表单上的什么元素会触发消息评估？
-  什么决定消息是否会被显示，以及它需要哪些信息？
-  消息说了什么？
-  触发器在表单提交时（正常验证）运行，还是仅用于内联 AJAX 验证？

概述你的应用程序必须知道什么，对于定义它需要哪些 API 函数（以及它必须存储的数据，在“定义你的数据模型”部分中讨论）至关重要。然而，回答这类问题也有助于你弄清楚应用程序必须做什么，并且在规划 API 的每个细节之前，先让某些东西工作起来绝对是值得的。

#### 概述一个 API

AJAX Form Messages 的核心想要表达的是：“我在一个表单上。当每个元素发生事情时，你想让我做什么？”

API 的核心必须是一个钩子或其他方式，供模块（包括通过 UI 模块的用户）注册关于消息的所有信息。（出于性能考虑，这可能不应在加载表单时进行。）我将深入探讨必须注册哪些信息的细节，并扩展“定义你的数据模型”部分中提出的问题。

一旦处理好了存储和获取这些信息，并且创建了模块修改这些信息的方式，基本 API 就完成了。甚至另一个模块也可以访问这些信息，并取代 AJAX Form Messages 与表单交互的实现。

API 的其他方面可能包括一种让模块影响输出默认值（例如消息放置的位置）的方式，但大多数其他潜在的 API 功能将是关于可以定义的消息信息以及使这些信息可修改的辅助功能。



### 深入实践

如果你觉得自己的问题开始进入理论范畴，那么就该动手编写能产生实际结果的代码了——*即使你明知未来会为此引入 API*。过早抽象出 API 可能适得其反。不过，正如前文所述，*首先*提出 API 需要回答的那类问题，确实能帮助你立刻写出更好的代码，即便你在第一版迭代中跳过了某些 API 的实现。

以 AJAX 表单消息为例，你需要将所有信息集中存储在一个中心位置，而不是每次显示表单时都去轮询所有模块。但处理这个问题会妨碍你理解触发器与消息本身的工作机制。而通过聚焦 API 所揭示的问题与答案，已经为你提供了大量可供尝试的思路与方向。

例如，触发错误或其他消息的必须是一个类似于验证函数的功能。理想情况下，你可以复用验证函数来判断是否应显示错误消息。然而，验证函数通常接收整个表单，而我这里讨论的是 AJAX 请求——它可能在同一个表单上多次触发，且必须保证速度。效率至关重要。那么，需要向评估函数发送哪些信息呢？

最简便的方式当然是传递整个表单，但或许可以让它既能接收一个表单（可以只包含你关心的部分，但依然能被任何常规且不过度严格的验证函数处理），也能接收单个具体元素。经过研究，日期验证函数 `date_validate()`（参见 `api.drupal.org/date_validate`）显然期望接收一个表单元素，而非整个表单（尽管其参数名为“`$form`”）。

![images](img/square.jpg) **注意** 通过选择使用 FormAPI，我们已经确保自己不会仅局限于节点（内容）表单——至少在最基础的 API 层面不会如此。

### 研究表单元素

AJAX 表单消息将在用户输入时监听表单元素，然后向这些表单元素发回消息。它将提供一个结构，用于存储一个函数，该函数用于检查用户输入内容并决定返回何种消息。这意味着它需要一种可靠的方法来在表单内部及跨表单时识别这些元素——因此你必须仔细研究表单的结构以及构成表单的元素。这种方法已在第 19 章中使用过，当时通过实现表单钩子来查看网站表单的结构。

```
/**
 * 实现 hook_form_alter()。
 */
function formmsgs_form_alter(&$form, &$form_state, $form_id) {
  debug($form, $form_id, TRUE);
}
```

将表单 ID 作为 `debug()` 的第二个参数，是一个巧妙的小技巧，可以精确显示你正在处理的是哪个表单。第三个参数 `TRUE` 是一个值得养成的良好习惯，这样 `debug()` 会使用一种不那么美观但更健壮（更不容易崩溃）的函数来打印输出。

![images](img/square.jpg) **陷阱** 如果你的 `debug()` 语句什么也没打印，请检查管理界面中的日志和错误设置：管理 ![Image](img/U001.jpg) 配置 ![Image](img/U001.jpg) 开发 ![Image](img/U001.jpg) 日志和错误（`admin/config/development/logging`）。要通过添加 `$conf['error_level'] = 2;` 到本地的 `settings.php` 文件来始终显示此类消息。通过在 `settings.php` 中添加四行代码来显示所有错误，具体方法如本书前文及 `dgd7.org/err` 所述。

**代码清单 24–2.** 从 `node/add/article` 页面打印表单变量的示例

```
  'title' =>
  array (
    '#type' => 'textfield',
    '#title' => '标题',
    '#required' => true,
    '#default_value' => NULL,
    '#maxlength' => 255,
    '#weight' => -5,
  ),
```

所有表单元素均由一个数组定义。代码清单 24–2 中的标题元素是 `$form` 数组的直接子元素，但它也可能嵌套在一个组中。其名称 'title' 是所示数组的键。它的位置（嵌套层级）和名称（'title'）共同构成了该表单上的唯一标识，但在另一个表单上可能被用于完全不同的元素。因此，你用来定位表单元素的工具并不精确。

表单元素的位置有些混乱。（甚至连“表单元素位置”这个说法都很别扭。）你必须决定：要么尝试找到即使已被移动到字段集分组内的表单元素，要么要求实现表单消息钩子的代码精确指出表单元素在表单数组中的位置。更便捷地找到无论被移动到何处的表单元素的方法是通过 `form_set_error()`（`api.drupal.org/form_set_error`），但它比 `form_error()`（`api.drupal.org/form_error`）更慢。至少为了简化起步，你将要求提供完整的位置信息。

不过你有点操之过急了。在弄清楚如何存储表单元素标识之前，你应该先确保能用一个测试用的表单元素来完成你需要做的事情。

![images](img/square.jpg) **注意** 研究用户编辑页面中密码强度检查所使用的 JavaScript，你会发现它根本没有使用 AJAX，而是直接在 JavaScript 中完成整个密码强度检查。这暗示了你的 API 可能具备另一种灵活性：你可以不亲自处理 JavaScript，而是允许模块定义自定义的 JavaScript。因此，它们可能根本不提供一个通过 AJAX 调用的函数。（我是通过实现 `hook_form_alter()` 查看表单结构，发现 `'pass'` 表单元素被定义为 `#type 'password_confirm'`，然后搜索 `'password_confirm'` 找到用户密码 JavaScript 的。`password_confirm` 表单类型由 `includes/form.inc` 中的函数 `form_process_password_confirm()` 展开，该函数会分别向第一个和第二个密码字段添加类 `'password-field'` 和 `'password-confirm'`。搜索这两个类中的任意一个，我便找到了 JavaScript 文件 `modules/user/user.js`。或许有更直接的查找方法，但永远不要畏惧探索！）



### 概念验证

现在来证明你可以创建一个表单元素，当用户在元素中输入内容时，根据 Drupal 中的函数显示一条消息。但让我们稍稍提高一点难度；来尝试运行一个验证函数。

Drupal 如何将验证函数分配给特定表单元素的方式，在 `api.drupal.org/api/drupal/developer--topics--forms_api_reference.html/7#element_validate`（这个超长的网址已链接自 `dgd7.org/strategy`）上的表单 API 文档中有所描述。这会是一个完美的锚点，用于通过通用的 `hook_form_alter()` 实现为元素添加 AJAX 回调。

在 Drupal 核心模块中搜索 `'element_validate'`（例如在 Drupal 根目录下的命令行中运行 `grep -nHR 'element_validate' modules/`）会找到一个值得尝试的示例。在 `modules/image/image.admin.inc` 文件的第 77 行，`element_validate` 属性被设置为使用 `image_style_name_validate()` 函数。

访问用于添加图片样式的管理页面（`admin/config/media/image-styles/add`），并在你的 `hook_form_alter()` 实现中加入 `debug($form, $form_id);`，你可以看到这个元素（即这个内联验证候选对象）的结构。

```
  'name' =>
  array (
    '#type' => 'textfield',
    '#size' => '64',
    '#title' => '样式名称',
    '#default_value' => '',
    '#description' => '名称用于生成图片的 URL。请仅使用小写
字母数字字符、下划线 (_) 和短横线 (-)。',
    '#element_validate' =>
    array (
      0 => 'image_style_name_validate',
    ),
    '#required' => true,
  ),
```

编写一个名为 AJAX 表单消息的模块时，离不开大量的异步 JavaScript，但本章仅涵盖基础知识。关于 JavaScript 的更多内容，请参见关于 jQuery 的 第 17 章，以及本章的在线后续内容（`dgd7.org/strategy`），对我而言这个后续内容的学习曲线肯定会非常陡峭。

![images](img/square.jpg) **提示** 如果你的模块编写了一组可以被视为一个软件包的 JavaScript 和/或 CSS 文件，请通过实现 `hook_library()` 将其作为库提供给其他模块，并通过使用 `#attached['library']` 或 `drupal_add_library()` 将其包含到你自己的页面中。更多信息请参见 `api.drupal.org/hook_library` 和 `drupal.org/node/756722`。

直接借鉴开发者示例项目（`drupal.org/project/examples`）中优秀的 AJAX 模块，你可以使用 `hook_form_alter()` 为 `'name'` 表单元素添加一个 AJAX 回调。Drupal 表单 API 文档中关于 AJAX 的示例似乎都没有将表单文本字段中的文本输入作为触发事件，但文档确实链接到了 `api.jquery.com/category/events`，那里有多个选项，其中包括 `keyup()`。

```
/**
 * 实现 hook_form_alter()。
 */
function formmsgs_form_alter(&$form, &$form_state, $form_id) {
  if ($form_id == 'image_style_add_form') {
    $form['name']['#ajax'] = array(
      'callback' => 'formmsgs_image_style_name',
      'event' => 'keyup',
      'wrapper' => 'formmsgs-image-style-name',
    );
    $form['name']['#suffix'] = '<div id="formmsgs-image-style-name">默认消息。</div>';
  }
}
```

请记住，这完全是概念验证，并且硬编码到了特定的表单和元素，与我们计划构建的 API 相去甚远。它允许你测试你的 AJAX 回调是否确实被调用，而无需担心失败可能出在其他地方。

```
/**
 * 测试回调函数。
 */
function formmsgs_image_style_name() {
  return '瞬间变化。';
}
```

成功了！当你在名称表单字段中开始输入任何内容时，表单下方的“默认消息。”文本会变为“瞬间变化。”。

![images](img/square.jpg) **注意** 当返回一个直接的字符串并使用默认的 'replace' 方法时，与 'wrapper' 指令匹配的整个占位符元素都会被替换。这包括，例如，你匹配的那个包含 ID 的 div。这意味着，除非你的返回文本再次包含该包装元素和 ID，否则 AJAX 将无法再次找到它并替换或更改任何内容。接下来使用的 AJAX 命令参考自示例项目（`drupal.org/project/examples`），并在 `api.drupal.org/api/group/ajax_commands/7` 中有文档说明，它提供了更强大的功能。

你可以独立地检查可以处理哪些信息。下面是同一个回调函数，但这次包含了传递给它的两个变量：表单数组和表单状态数组：

```
function formmsgs_image_style_name($form, $form_state) {
  die(var_export($form,TRUE));
}
```

通过这种方式，你可以看到通过 AJAX 传递给回调函数的变量。完整输出请参见 `dgd7.org/273`；其中最重要的是我们拥有了 name 元素的当前值——也就是你刚刚输入的内容。

```
function formmsgs_image_style_name($form, $form_state) {
  image_style_name_validate($form['name']);
  $message = form_get_error($form['name']);
  if (!$message) {
    $message = "默认消息。";
  }
  $commands = array();
  $commands[] = ajax_command_html('#formmsgs-image-style-name', $message);
  return array('#type' => 'ajax', '#commands' => $commands);
}
```

这将文本“默认消息”替换为“样式名称中请仅使用小写字母数字字符、下划线 (_) 和短横线 (-)”，如图 24-1 所示。

![images](img/2401.jpg)

***图 24-1**. 通过 AJAX 在表单上立即显示图片样式名称的验证错误消息。*

这看起来令人印象深刻且相当简洁——运行验证函数，获取错误，并使用 AJAX `html` 命令返回它——但实际上它几乎无法正常工作。不过，作为使用普通验证函数的概念验证内联验证函数，它已经完成了任务。每次都会正确返回“请仅使用小写字母数字…”消息，或者什么都不返回。不幸的是，当用户正在输入时，它会导致文本区域失去焦点。此外，它传递了整个 `form` 和 `form_state` 变量，而没有将其限制在需要的部分，这对于每次按键验证来说开销太大了。实际上，很明显，像这样的允许字符的内联验证（与防止重名不同，后者需要检查数据库）应该仅使用 jQuery 来完成，而无需对 Drupal 进行 AJAX 调用。

请记住，这只是一个概念验证。除了在输入时因为延迟而丢失的字符、不必要的进度提示器以及源自 Drupal 便捷的 `#ajax` 属性的其他问题（你可以使用 `#path` 属性代替 `#callback` 属性来获得更多控制权）之外，你还对 Drupal 现有的验证函数和 `form_get_error()` 做了可能无法长期稳定工作的不恰当处理。但概念已经被证实了！你可以在解决性能和实现细节的同时，继续建立一个 API。



### 定义你的数据模型

为了让表单消息实现内联验证，它需要获取应显示的消息内容、触发显示的条件以及更多相关信息。这是 API 的一部分，应被视为如此，但你也同时将其视为你想要存储的数据。常规的表单验证本身并不需要数据模型，但如果你打算允许人们通过用户界面来定义消息和模块，那么你就需要一个两者交汇的数据模型。

正是在这里，你需要深入探讨 AJAX 表单消息需要确切了解哪些信息，并全面细致地思考其需求。每条消息可应用于多个表单——例如，具有相同字段的节点表单，或者搜索块表单与主搜索表单。目前看来，让每个表单消息从一个单一表单元素的主视角出发是合理的，即使有多个字段与验证相关。

像这样逐一攻克问题后，你会得出一个初步的待存储数据列表。每条消息和评估规则的组合需要提供以下内容：

*   一个或多个表单 ID，或一个用于匹配表单 ID 的模式。
*   可选的进一步条件逻辑，用于决定是否应用评估和消息。（这通常是必需的，因为仅凭表单 ID 不够；例如，所有内容类型都使用相同的节点表单。）
*   触发器/接收器表单元素。这是用户输入或点击以触发消息的位置，同时也是接收消息——即消息应显示的位置。也许触发器/接收器也可以是多重值的，例如两个电话号码字段应该接收相同的验证和消息。
*   可选的相关表单元素。这允许内联验证考虑到其他表单字段中输入的值，例如要求在一个词汇表中的选择与另一个词汇表中的选择形成唯一组合。
*   一个要执行的评估函数。它至少接收一个值：触发器/接收器中的值，以及一个包含其他表单元素值的数组（若指定了其他元素）。第三个参数也是可选的，即一个上下文数组，例如用于存储用户 ID。如果提供了上下文，它将是一个回调函数数组，用于创建一个带键值的数组。
*   当评估函数返回一个命中结果（任何严格不等于`false`的值，`=== FALSE`）时显示的一条静态消息。
*   一个消息回调函数——可选，用于替代上述字段中设置的消息，以便你可以随意设置消息内容，但 AJAX 表单消息 API 无需关心。此回调函数将接收评估函数的结果以及评估函数被提供的任何数据。
*   这是一个允许表单提交的警告，还是一个将被添加到表单验证例程中的错误？（或者是一个已应用于表单验证例程的错误，这远比依赖一个名为“表单消息”的模块来为你做表单验证要好得多。）
*   一个可选的在默认状态下显示的消息。
*   一个可选的为成功选择或输入提供的消息。

其余的事情将由表单消息模块处理。嗯，一旦它被编码完成。

消息将始终在触发错误的表单元素处显示。除非触控板输入方式允许某些不合常规的三指操作，否则任何时候只有一个表单元素真正处于活动状态：即正在输入或选择值的那一个。可以将这视为简化代码的一种合理化解释，但用户界面应该只在你输入文本或进行选择的地方显示变化，而不是在表单的其他位置。

#### 如何存储数据以及如何在用户界面中编辑它

> *“理论上，理论和实践是统一的。实际上，它们并非如此。”*
>
> ——尤吉·贝拉

数据的存储和在用户界面中的编辑应该是两个独立的问题。它们确实应该如此。定义数据模型。定义其存储方式。在没有用户界面的情况下使其正常工作。然后在此基础上构建用户界面。

但是，复用 Drupal 提供的工具这一实际问题会改变方法。将复杂的配置对象（比如上一节中概述的）视为带有字段的实体是有意义的。这在 Drupal 中存在争议。我个人非常喜欢最终拥有一个能够追踪谁在何时做了什么的管理界面——借助表单消息实体的修订功能，这很容易实现。

当你想要一个用户界面能够完美地与代码中捕获的相同信息协同工作时，你就需要配置是可导出的，这意味着如果使用实体，就需要以 CTools 风格使实体可导出。一方面，这听起来庞大而令人畏惧。另一方面，这正是 Drupal 发展的自然趋势，可能已经有其他疯狂的人在致力于做同样的事情。虽然不容易找到，但不知疲倦的 Wolfgang Ziegler (fago) 已经记录了他的 Entity API 模块，用于创建可导出的实体，其目的正是为了存储配置。这份文档位于 `drupal.org/node/1021526`。(不幸的是，那种导出实体的方法与 CTools 方法是分离的，两者尚未发展出集成的方法。)

再次强调，使用实体存储配置是有争议的。接下来的章节也作为对实体的介绍，无论你将它们用于什么目的！请注意，我这里所说的导出并非指导出实体类型定义（这些本来就在代码中），而是指导出实体字段中存储的内容。这就是为什么这种方法存在争议；许多人认为字段中的数据永远是数据，而非配置。在创建可导出实体之前，有没有办法使用基于代码的表单消息定义？也许有，但这似乎需要在 Drupal 本身能提供存储机制的情况下，自己编码一个存储机制。

理想情况下，它应该是一个 JSON 导出，因为 JSON 允许安全地复制粘贴以进行导入。允许人们粘贴 PHP 代码是无法保证安全的，并且 CTools 计划切换到 JSON。根据对使用 Entity API 的 Profile2 和 Message 模块的实践，它们确实会导出为 JSON。

同样理想的是，如果永远不需要用户界面，该导出应仅存在于代码中。在这一点上，不幸的是，Entity API 总是自动导入到数据库中，而不是在运行时从代码中读取。但总体权衡之下，使用 Entity API 模块的可导出实体相当符合你的偏好。

此外，你还能使用`entityFieldQuery()`，这是所有 Drupal 7 先行开发者都在热议的功能。显示实体应使用`EntityFieldQuery` (`api.drupal.org/EntityFieldQuery`) 或 Views——避免创建你自己的查询和显示系统。请参阅 `dgd7.org/entities` 查看示例。

![images](img/square.jpg) **提示** 让你的模块可导出。与良好 API 密切相关的是让模块的配置**可导出**。目前，在 Drupal 中辅助配置导出主要有两种方法：对于实体，使用 Entity API；对于任何可以放入数据库表中的内容，使用 CTools。可导出性的首要规则是不依赖数字键，这两种解决方案都解决了这个问题。CTools 更常见且测试更充分，其用法在 Views 和 Panels 等模块中通过示例有详尽的文档说明。

### 提供一种新的实体类型

尽管可导出实体并非得到所有 Drupal 开发者的普遍认可，但我们还是继续创建这样一个实体，因为它似乎非常适合我们的用例。

虽然实体对 Drupal 7 来说是新的，但创建它们并非未知领域。整个节点系统现在都基于实体，评论、术语、词汇表、文件和用户也是如此。贡献模块可以并且确实定义了自己的实体类型，你的模块也可以。



#### 何时创建实体类型

创建实体类型最普遍的原因是为了拥有自己的字段化实体。在创建新实体之前，应始终尝试使用字段扩展现有实体。但是，不要将节点用于非内容（non-content）的任何事情——这通常是创建自己实体的最佳时机。Commerce 模块（参见第 25 章）将实体用于产品等，因为产品具有与内容不同的需求。就产品而言，它们尤其需要许多子类型，而将这些子类型用作内容类型最多只能算是一种滥用。

#### 如何创建实体类型

实体类型通过实现 `hook_entity_info()` 来声明。请参阅（像往常一样）优秀的 Examples 项目（位于 `drupal.org/project/examples`）、`api.drupal.org/hook_entity_info`；并查阅在线教程和示例的最新列表，请访问 `dgd7.org/entities`。

`AJAX Form Messages` 选择使用实体来存储配置（我是否提过这有争议？），部分原因是 Entity API（`drupal.org/project/entity`）提供的导出能力，这是一个贡献模块，它为核心实体增加了许多功能。这意味着你的实体定义将与仅基于核心的实现略有不同，尤其是在你利用 Entity API 提供的能力时。你可以按照贡献的 Entity API 的在线文档来创建可导出的实体，地址为 `drupal.org/node/1021526`。

使用 Entity API 的第一步（如同依赖任何模块一样）是将其声明为依赖项——很容易忘记后面再做这件事，而当你的用户在尝试启用你的模块时，该模块破坏了他们的网站，他们可不会感谢你。

`name = AJAX 表单消息 API`
`description = "[formmsgs] 提供表单内即时验证要求的通知。"`
`package = AJAX 表单消息`
`core = 7.x`
`dependencies[] = entity`

在这个 `.info` 文件中，还有一些其他事项需要注意。因为主模块旨在成为一个 API 模块，并附带一个 UI 和几个可选的支持模块（可能还有更多单独贡献的模块），所以你需要在名称中包含 API，并为其指定一个包指令，这将使所有属于 "AJAX Form Messages" 包（package）的模块在模块管理页面（`admin/modules`）上被分组在一起。

在定义实体时，下一步可以在你的 `.install` 文件中进行。每个实体都需要一个数据库表来保存其基本信息。这包括一个整数序列 ID 列（或者字段，如列表 24-3 的模式定义中所称），这是任何实体所必需的；以及机器可读的名称列，这对于实体可导出至关重要。

**列表 24–3**。在表单消息模块的 `formmsgs.install` 文件中实现 `hook_schema()`

```
<?php

/**
 * @file
 * AJAX 表单消息的数据库模式、安装和卸载函数。
 *
 * 此处定义了实体基础表。
 */

/**
 * 实现 hook_schema()。
 */
function formmsgs_schema() {
  $schema = array();
  $schema['formmsgs'] = array(
    'description' => '存储所有 formmsgs 实体的信息。',
    'fields' => array(
      'fmid' => array(
        'type' => 'serial',
        'not null' => TRUE,
        'description' => '主键：唯一的表单消息 ID。',
      ),
      'name' => array(
        'description' => '表单消息的机器可读名称。',
        'type' => 'varchar',
        'length' => 32,
        'not null' => TRUE,
      ),
      'label' => array(
        'description' => '此表单消息的人类可读名称。',
        'type' => 'varchar',
        'length' => 128,
        'not null' => TRUE,
        'default' => '',
      ),
      'status' => array(
        'description' => '指示表单消息是否激活的布尔值。',
        'type' => 'int',
        'size' => 'tiny',
        'not null' => TRUE,
        'default' => 1,
      ),
    ) + entity_exportable_schema_fields(),
    'primary key' => array('fmid'),
    'unique keys' => array(
      'name' => array('name'),
    ),
  );
  return $schema;
}
```

一个值得注意的小技巧是 `+ entity_exportable_schema_fields()`，它使用了 Entity API 模块提供的一个便捷函数，为实体的表添加了额外的几列。这些列（或字段）用于导出状态和提供模块的名称，Entity API 的导出功能需要它们，但这样可以省去你自己定义它们的麻烦。



```markdown
创建新型实体的下一步是在你的模块中实现 `hook_entity_info()`。这一步相当公式化。关键要素是确定在 `hook_schema()` 实现中定义的基础表（我们刚才在 `.install` 文件中完成了这一工作）。对于任何内容类实体，一个很好的参考模型是 `modules/node/node.module` 中的 `node_entity_info()`，因此可以从该函数以及 `drupal.org/node/878804` 的 Entity API 文档中借鉴一些内容。Entity API 特有的部分包括控制器类（将在后续讨论），以及将 `'exportable'` 属性设置为 `TRUE` 的能力。参见清单 24-4。

**清单 24-4**。在 `formmsgs.module` 中定义新的表单消息实体

```
<?php

/**
 * @file
 * 提供表单内的即时验证需求。
 */

/**
 * 实现 hook_entity_info()。
 */
function formmsgs_entity_info() {
  $return = array(
    'formmsgs' => array(
      'label' => t('表单消息'),
      'controller class' => 'EntityAPIController',
      'entity class' => 'Formmsgs',
      'base table' => 'formmsgs',
      'fieldable' => TRUE,
      'exportable' => TRUE,
      'entity keys' => array(
        'id' => 'fmid',
        'name' => 'name',
        'label' => 'label',
      ),
      'access callback' => 'formmsgs_entity_access',
      'module' => 'formmsgs',
      'admin ui' => array(
        'path' => 'admin/structure/formmsgs',
        'file' => 'formmsgs.admin.inc',
      ),
      'bundle keys' => array(
        'bundle' => 'name',
      ),
      'bundles' => array(
        'formmsgs' => array(
          'label' => t('消息'),
        ),
      ),
      'view modes' => array(
        'full' => array(
          'label' => t('在表单上'),
          'custom settings' => FALSE,
        ),
      ),
    ),
  );
  return $return;
}
```

`'controller class'` 为 `EntityAPIController`，它能让你的实体利用 Entity API 的功能。定义 `'entity class'` 是该过程所必需的。在上述代码中，实体类是 `Formmsgs`。这个类需要被定义，我们将在下一节进行。

由于你希望使用 Field API 来收集和存储表单消息的数据，因此将 `'fieldable'` 设置为 `TRUE`。这是核心实体的功能，与来自 Entity API 的 `'exportable'` 属性不同。

`'base table'` 必须是 `hook_schema()` 中定义的表名。实体类型属性 `'label'` 仅为 `'表单消息'`（即如何称呼此类型实体），而在 `'entity keys'` 属性中，`'label'` 指的是你的基础表中存储各个表单消息标签的列（也称为 label）。ID（`fmid`）和名称（`name`）列同样在 `'entity keys'` 属性中进行匹配。

实体键 `'name'` 是 Entity API 提供的另一特性；它允许表单消息通过机器名称进行导出。在跨部署或跨不同站点时，使用顺序数字 ID 进行导入和导出效果不佳。

所有与包相关的属性都可以省略，因为 `formmsgs` 实体只有一个包，在没有特别指定的情况下，Drupal 会自动以实体自身名称命名该包。我们有机会定义单个包及其标签，且与这里的所有代码一样，遵循核心、贡献模块和 `drupal.org` 文档中的示例。本章中的链接（以及更多内容）也可以在 `dgd7.org/entities` 找到。

![images](img/square.jpg) **注意** 在创建不借助 Entity API 的实体类型时，你可能需要定义自己的控制器类，例如 `FormmsgsController`，并扩展 Drupal 提供的类（如 `DrupalDefaultEntityController`）。在该类中可以包含创建方法以及对继承方法的扩展。不常用的类（例如创建实体时使用的类）应存放在外部文件中，并通过 `.info` 文件中的 `files[]` 指令引用。这有助于提升未使用操作码缓存的站点性能。在此示例中，Entity API 模块替我们处理了这些。Drupal 只会在需要用到 `EntityAPIController` 类时加载其 `includes/entity.controller.inc` 文件。`EntityAPIController` 扩展了 `DrupalDefaultEntityController` 并做了大量工作，但如果你需要更多功能，可以创建自己的类来扩展 `EntityAPIController`。

将 `'entity class'` 属性设置为 `Formmsgs` 意味着你必须定义这个类。如果该类不止这几行代码，将其放入 `formmsgs.info` 中通过 `files[]` 指令引用的 `include` 文件中会更合理。

```
/**
 * 用于表单消息实体的类。
 */
class Formmsgs extends Entity {

  public $label;
  public $status;

  public function __construct($values = array()) {
    parent:__construct($values, 'formmsgs');
  }
}
```

这是对 Entity API 的 `Entity` 类的轻量级扩展；主要作用是调用 `Entity` 类的构造函数。它还声明了 `label` 和 `status` 变量，使它们在创建表单消息时即使为空变量也能无错误地使用。

### 定义实体访问回调函数

Entity API 需要一个访问回调函数。在前面的 `hook_entity_info()` 实现的 `'access callback'` 属性中，你将该函数命名为 `formmsgs_entity_access()`。在列出回调函数名称时，省略括号 `()`。这个访问回调函数改编自 `entity/modules/callbacks.inc` 中的 `entity_metadata_comment_access()`。

```
/**
 * Entity API 提供的 formmsgs 管理区的访问回调。
 *
 * @TODO 为 Entity API 打补丁，使其接受 hook_menu 风格的 'access arguments'，
 * 以便在简单的 user_access() 情况下无需此函数。
 */
function formmsgs_entity_access($op, $entity = NULL, $account = NULL) {
  return user_access('administer formmsgs');
}
```

Entity API 要求提供访问回调，以换取它提供的实体管理和编辑用户界面。正如 `@TODO` 注释所述，当你只想基于简单的用户权限来确定访问时，似乎应该能直接提供权限字符串作为参数，而无需创建自己的访问回调来包装 `user_access()`。无论如何，为了让访问回调生效并拥有特定于实体的权限，你需要为新实体定义一个或两个权限。
```


### 定义权限

第 21 章建议，若非必要，请避免定义新权限。但如果你的模块实现了新的功能，且站点管理员应能根据用户角色允许或拒绝该功能时，你就需要定义新权限。代码清单 24–5 展示了权限的定义过程。

**代码清单 24–5.** 系统模块中 `hook_permission()` 实现的片段

```
/**
 * 实现 hook_permission()。
 */
function system_permission() {
  return array(
    'administer modules' => array(
      'title' => t('管理模块'),
    ),
    'administer site configuration' => array(
      'title' => t('管理站点配置'),
      'restrict access' => TRUE,
    ),
// ...
  );
}
```

`'restrict access' => TRUE` 指令会指示 Drupal 在权限管理页面的权限名称下方（如果存在描述，则在描述之后）打印一条提示：*警告：仅授予可信角色；此权限具有安全影响。* 它没有其他用途；这只是为模块开发者提供的一种便利，用于以一致的方式提醒管理员不要随意分配某些权限。

如果你想就授予某个权限向管理员发出更精确的警告，可以直接在描述字段中添加消息。核心的筛选器模块对已过滤格式和完整 HTML 格式（以及任何未配置为回退格式的格式）就是这样做的。筛选器模块还会为每种文本格式动态生成一个权限——这很酷，但描述中的自定义警告才是当前主题，此处以粗体强调：

```
  // 为每种文本格式生成权限。警告管理员：其中任何一种都可能有潜在危险。
  foreach (filter_formats() as $format) {
    $permission = filter_permission_name($format);
    if (!empty($permission)) {
      // 仅当查看此页面的用户有权访问文本格式配置页面时，才链接到该页面。
      $format_name_replacement = user_access('administer filters') ? l($format->name,
'admin/config/content/formats/' . $format->format) : drupal_placeholder($format->name);
      $perms[$permission] = array(
        'title' => t("使用 !text_format 文本格式", array('!text_format' =>
$format_name_replacement,)),
        'description' => drupal_placeholder(t('警告：此权限可能具有安全影响，具体取决于文本格式的配置方式。')),
      );
    }
  }
  return $perms;
```

![images](img/square.jpg) **注意：** 筛选器模块的 `hook_permission()` 实现中还有一点酷到值得一提：每种文本格式的标题会链接到其配置页面，但前提是*管理用户*有权配置文本格式。它通过 `user_access()` 函数进行权限检查，从而增强了其权限定义的可用性！

表单消息模块的 `hook_permissions()` 实现则远没有那么令人兴奋。它确实有一个绕过权限，模仿了唯一字段模块。

```
/**
 * 实现 hook_permission()。
 */
function formmsgs_permission() {
  return array(
    'administer formmsgs' => array(
      'title' => t('管理 AJAX 表单消息'),
      'description' => t('允许管理员配置错误和警告消息。'),
    ),
    'bypass formmsgs' => array(
      'title' => t('绕过表单消息错误'),
      'description' => t('允许用户忽略通过表单消息设置的错误。'),
    ),
  );
}
```

### 为实体提供管理界面

Entity API 在提供基于其管理实体的管理用户界面方面投入了大量工作，但一些常规设置仍需你自行完成。之前在代码清单 24–4 中，通过 `hook_entity_info()` 定义表单消息实体时，你使用以下代码行定义了管理用户界面路径和一个单独的管理文件：

```
       'admin ui' => array(
'path' => 'admin/structure/formmsgs',
'file' => 'formmsgs.admin.inc',
),
```

你需要通过在 `formmsgs.admin.inc` 文件中实现几个函数来兑现这一承诺（见代码清单 24–6）。当函数的名称以实体名后跟 `'_form'` 的形式命名时，Entity API 会自动接管主要的管理表单。（如果你的模块同时定义了一个与模块同名的、基于 Entity API 的增强实体，以及一个旧式的、模块拥有的节点类型，此回调将与节点表单回调冲突——但这种情况不太可能发生。）

**代码清单 24–6.** 在 `formmsgs.admin.inc` 中定义的表单消息实体管理界面

```
<?php
/**
 * @file
 * 仅在管理页面上需要的表单和函数。
 */

/**
 * 生成表单消息实体的添加/编辑表单。
 *
 * 此表单会被 Entity API 模块提供的管理界面自动接管。
 */
function formmsgs_form($form, &$form_state, $formmsg, $op = 'edit') {

  if ($op == 'clone') {
    $formmsg->label .= ' (克隆)';
    $formmsg->name .= '_clone';
  }

  $form['label'] = array(
    '#title' => t('标签'),
    '#type' => 'textfield',
    '#default_value' => $formmsg->label,
  );
  // 机器可读的表单消息名称。
  $form['name'] = array(
    '#type' => 'machine_name',
    '#default_value' => isset($formmsg->name) ? $formmsg->name : '',
    '#disabled' => ($op === 'edit') ? TRUE : FALSE,
    '#machine_name' => array(
      'exists' => 'formmsgs_load_by_name',
      'source' => array('label'),
    ),
    '#description' => t('此表单消息的唯一机器可读名称。只能包含小写字母、数字和下划线。'),
  );
  $form['status'] = array(
    '#type' => 'checkbox',
    '#title' => t('启用'),
    '#default_value' => $formmsg->status,
  );

  field_attach_form('formmsgs', $formmsg, $form, $form_state);

  $form['actions'] = array('#type' => 'actions');
  $form['actions']['submit'] = array(
    '#type' => 'submit',
    '#value' => t('保存表单消息'),
    '#weight' => 50,
  );
  return $form;
}

/**
 * 表单 API 提交回调，用于 formmsgs 实体添加/编辑表单。
 */
function formmsgs_form_submit(&$form, &$form_state) {
  $formmsg = entity_ui_form_submit_build_entity($form, $form_state);
  // 保存并返回。
  $formmsg->save();
  $form_state['redirect'] = 'admin/structure/formmsgs';
}
```

在 `formmsgs_form()` 中有一行非常关键的代码你尚未使用：`field_attach_form()` 函数。它将允许在为表单消息实体定义的字段中，随着标签和机器名称一起填写字段值。你将在下一节中通过编程方式定义字段。

下一个函数 `formmsgs_form_submit()` 是一个简单的表单提交函数实现。借助 Entity API 的辅助函数，你只需在表单消息对象上调用 `->save()` 方法即可。

然而，即使有了 Entity API 的帮助，仅凭这个表单，你还无法在管理界面中列出和编辑表单消息。你需要先定义一些 Entity API 依赖的加载函数。代码清单 24–7 中的代码应放入 `formmsgs.module` 文件中，因为它具有更通用的用途，但其直接需求是支持管理操作。



这些加载函数的模式基于`Profile2`模块（同样由实体 API 的创建者 fago 开发）。其中比较特殊的是`formmsgs_load_by_name()`，它模仿了`profile2_get_types()`，满足了实体 API 管理界面的一项特殊需求。

另外两个函数直接类比于`node_load()`和`node_load_multiple()`。你会看到，与节点加载函数类似，`formmsgs_load()`通过调用`formmsgs_load_multiple()`来工作。这遵循了加菲尔德定律：*一个是一种特殊情况。*关于 Larry Garfield（crell）目前提出的八条 API 设计箴言，请参阅我对他的演讲笔记（`data.agaric.com/aphorisms-api-design`）或基于这些笔记的 DrupalCon 芝加哥录音（或者可以在未来的 DrupalCon 或 Drupal 训练营上听他重新讨论这个主题）。

**Listing 24–7. 实体 API 管理界面正常工作所需的实体加载函数（定义在`formmsgs.module`中）**

```
/**
 * Fetches an array of all form messages, keyed by the formmsg machine name.
 *
 * Also used to check if machine name is used for an existing form message.
 *
 * @param $name
 *   If set, the form message with the given name is returned.
 * @return $formmsgs
 *   An array of form messages or, if $name is set, a single one.
 */
function formmsgs_load_by_name($name = NULL) {
  $formmsgs = entity_load('formmsgs', isset($name) ? array($name) : FALSE);
  return isset($name) ? reset($formmsgs) : $formmsgs;
}

/**
 * Fetch a form message object.
 *
 * @param $fmid
 *   Integer specifying the form message id.
 * @param $reset
 *   A boolean indicating that the internal cache should be reset.
 * @return
 *   A fully-loaded $formmsg object or FALSE if it cannot be loaded.
 *
 * @see formmsgs_load_multiple()
 */
function formmsgs_load($fmid, $reset = FALSE) {
  $formmsg = formmsgs_load_multiple(array($fmid), array(), $reset);
  return reset($formmsg);
}

/**
 * Load multiple profiles based on certain conditions.
 *
 * @param $fmids
 *   An array of form message IDs.
 * @param $conditions
 *   An array of conditions to match against the {formmsgs} table.
 * @param $reset
 *   A boolean indicating that the internal cache should be reset.
 * @return
 *   An array of form message objects, indexed by fmid.
 *
 * @see entity_load()
 * @see formmsgs_load()
 */
function formmsgs_load_multiple($fmids = array(), $conditions = array(), $reset = FALSE) {
  return entity_load('formmsgs', $fmids, $conditions, $reset);
}
```

所有这些加载函数最终都依赖于 Drupal 核心的`entity_load()`函数（参见`api.drupal.org/entity_load`），`EntityAPIController`为其提供了自己的实现。

现在你可以创建、列表、编辑和删除 Form message 实体，但每个实体只包含一个标签、一个机器名和一个状态。要获得你最初选择实体路线时所寻求的全部能力和灵活性，你的模块需要定义 Drupal 7 字段。

### 以编程方式创建和附加字段

字段可以通过用户界面创建并附加到实体分组（例如内容类型）。这确实是字段的一个关键原因和目的，但它们也可以在代码中定义。在罕见的情况下，使用字段来存储配置信息（这是一个有争议的用例，应该再次指出），`AJAX Form Messages`甚至不希望在用户界面中配置这些字段。为了向 Form messages 的用户提供与之前脑力激荡出的数据模型相匹配的所有字段，你绝对需要在代码中创建并附加这些字段。

#### 寻找模型

核心中有几个地方以编程方式将字段附加到内容类型，这与将字段附加到你自己的实体和分组是类似的。Node 模块有一个封装了向内容类型添加`body`字段的函数`node_add_body_field()`，你可以在`modules/node/node.module`或`api.drupal.org/node_add_body_field`中看到它。Standard 安装配置文件还附加了一个 Taxonomy 字段，可以在`profiles/standard/standard.profile`中大约第 283 行看到。

对于消息，你需要一个文本字段。要精确查看有哪些文本字段可供使用，请直接查看 Drupal 核心的 Text 模块，它是 Field 模块的子模块，位于`modules/field/modules/text.module`。

```
function text_field_info() {
  return array(
    'text' => array(
      'label' => t('Text'),
      'description' => t('This field stores varchar text in the database.'),
      'settings' => array('max_length' => 255),
      'instance_settings' => array('text_processing' => 0),
      'default_widget' => 'text_textfield',
      'default_formatter' => 'text_default',
    ),
    'text_long' => array(
      'label' => t('Long text'),
      'description' => t('This field stores long text in the database.'),
      'instance_settings' => array('text_processing' => 0),
      'default_widget' => 'text_textarea',
      'default_formatter' => 'text_default',
    ),
// ...
  );
}
```

`text`字段的最大长度（`max_length`）可能是 255 个字符，因此选择`text_long`格式对你有益。然而，这 255 个字符只是一个设置；在创建字段时，它可以被设置为不同的、更大的值。（最长安全值约为 50,000 字节，参见`drupal.org/node/1052248`）。

综合起来，要添加一个字段，你回到你的`.install`文件。这是一个两步过程：定义（创建字段）和附加到实体（创建字段实例），这两个步骤可以在`hook_install()`的实现中一起完成。

```
/**
 * Implements hook_install().
 */
function formmsgs_install() {
  // 定义字段。
  $field = array(
    'field_name' => 'field_message',
    'type' => 'text_long',
    'entity_types' => array('formmsgs'),
    'translatable' => TRUE,
  );
  $field = field_create_field($field);

  // 附加字段。
  $instance = array(
    'field_name' => 'field_message',
    'entity_type' => 'formmsgs',
    'label' => t('Message'),
    'bundle' => 'formmsgs',
    'description' => t('错误时显示的消息。'),
    'widget' => array(
      'type' => 'text_textarea',
      'weight' => -5,
    ),
  );
  field_create_instance($instance);
}
```

定义字段的过程很简单：创建一个包含字段信息的数组，然后对该数组调用`field_create_field()`。附加字段的过程类似，它不需要传入你创建的字段；而是使用与你刚刚创建的字段相同的字段名称，并同时提供你正在附加的实体类型名称。然后，它可以选择性地接受实例级别的设置。真正的技巧在于翻阅核心和贡献模块的`.install`文件，以寻找不同字段的示例。如有必要，你也可以定义自己的字段类型。为`AJAX Form Messages`创建自定义字段类型将在`dgd7.org/strategy`中进行记录。

![images](img/square.jpg)**注意** 请记住，一旦你发布了模块的测试版本（此时用户期望能够安全升级），除了在`hook_install()`中，你还需要在`hook_update_N()`的实现中定义和附加任何新字段。



### 明确“完成”的定义

> “在水面上行走和根据规范开发软件，如果两者都冻结了，那都很容易。”
> ——爱德华·V·贝拉德

当然，关于定义“完成”的这一节本应放在开头。（我没有把第 10 章关于项目规划与管理的内容写进来是有原因的。）但别让这吓跑你，这里有一些好建议：首先要明确定义目标。

几乎所有项目几乎都可以无限期地延展；你投入的时间越多，想到的酷炫功能就越多。尽早定义能够交付的最低标准有助于保持专注。用你自己的问题队列来提交功能请求，但不要让它们妨碍你完成目标。先交付以满足第一个用例，并尝试安排时间在之后重新审视它。

![images](img/square.jpg) **提示** 你不应该在哪些方面偷工减料？就是保留日后以不同方式处理问题的灵活性。每当关于最佳实现方式的决定最好留到以后再做时，你就要封装功能，定义代码各部分之间的边界和接口。这也意味着要遵守你自己的 API，而不是为你的代码需求设置特例；你的模块不应享有高于其他模块的特权。编码时要假设你以后无法编辑自己的代码。如果你需要更多灵活性，也要让其他模块能够使用这种灵活性。

当然，在开源项目中，任何人都可以认为项目还不够“完成”，并亲自投入工作。Drupal 社区的一大优势在于，新加入项目的人频繁地做出重大贡献，包括接管现有模块的维护工作。

对于这个模块而言，作为一项案例研究，“完成”被定义为*本书需要出版*。请关注 Form message 模块在 `dgd7.org/strategy` 和 `drupal.org/project/formmsgs` 上的持续冒险。

## 第六部分

![images](img/squ.jpg)

## 高级站点构建主题

**第 25 章** 涵盖了构建在线商店的内容，并带你了解从零开始重构 Ubercart（Drupal 6 排名第一的电商套件）为 Drupal 7 的 Drupal Commerce 的决策过程。本章对任何构建电商网站的人都有价值，同时也会邀请你加入 Commerce 开发者社区。

**第 26 章** 提供了对高级 Drush 用法的深入见解，这将像你初次使用 Drupal 命令行工具 Drush 时一样，彻底改变你的站点开发体验。本章包含让你开始编写自己的 Drush 脚本和命令的示例。

**第 27 章** 讲解了使用 Drupal 可插拔的缓存和存储机制来扩展到数百万站点用户（不仅仅是访客，这相对容易扩展，而是与站点进行大量交互的用户）的概念和实践。另请参阅附录 B。

**第 28 章** 给出了将语义网的力量引入 Drupal，反之亦然——让你的网站数据与计算机能理解的精确含义相关联，并连接到互联网上其他数据——的理论和实践。（SEO 提示：计算机包括搜索引擎。）

**第 29 章** 带领你了解 Drupal 的路由系统，为模块开发者和站点构建者提供关键背景知识。

**第 30 章** 带你参观在页面请求期间 Drupal 内部发生的情况，这是对第 29 章的完美补充，也是真正理解 Drupal 的好方法。

**第 31 章** 解释了如何使用和扩展 Solr 模块以获得更强大的搜索能力。在后一种能力中，它提供了一个将 Drupal 与 Web 服务集成并利用面向对象代码的示例。

**第 32 章** 深入探讨了 Drupal 7 中的用户体验改进、其背后的决策，以及如何在你的开发中使用新的最佳实践和一致的界面设计决策。

**第 33 章** 处理高级配置和大量粘合代码——实际上，是不惜一切代价——来完成在第 1 章和第 8 章中构建的 DefinitiveDrupal.org 网站。

**第 34 章** 介绍了一些流行的 Drupal 发行版——为特定目的打包的 Drupal 和模块集合，它们正以前所未有的方式传播 Drupal——并向你展示如何使用 Drupal 的安装配置文件功能制作你自己的发行版。

## 第 25 章

![images](img/squ.jpg)

## Drupal Commerce

作者：Ryan Szrama

由于为 Drupal 7 开发的 Drupal Commerce，Drupal 的电子商务功能比以往任何时候都更强大。Drupal Commerce 项目由一套核心的 Commerce 模块和一个利用了许多新的 Drupal 7 特性和 API 改进的实现策略组成。本章从 Drupal Commerce 的广泛概述开始，突出其关键特性，然后深入审视核心系统、它们的实现方式，以及应如何将它们结合使用。本章还包含为想要在自己的站点上实施 Drupal Commerce 的站点构建者和开发者提供的智慧箴言。本章以对项目开发历史、设计理念以及关键 Drupal 7 特性利用的讨论结束。

### Drupal Commerce 概述

从很多方面来看，Drupal 都是电子商务网站的理想平台。其核心模块和系统定义了用于将贡献模块深度集成到站点行为中以及与外部 Web 服务通信的 API。它包含了大量用于内容管理和社区构建的特性，使你能够围绕你的产品和服务建立一个社区，或者通过客户现有的社交网络关系来推广你的品牌。

从 Drupal 4.5 时代起，所有主要的电子商务模块都建立在 Drupal 的基础功能集之上，将其作为电子商务的基础，而不是与外部应用程序集成，Drupal Commerce 也不例外。随着 Drupal 的成熟，这个核心功能和主要贡献系统（如 Views）的基础也在不断发展，为构建在其上的模块提供了更大的灵活性和能力。因此，Drupal Commerce 是从头开始构建的，采用了围绕 Drupal 7 最新特性和 Drupal 贡献模块（如 Rules 和 Views）最大发展的全新架构。

最终的结果是一个可以从头开始构建以满足你业务需求的电商解决方案，无论业务是大是小。Drupal Commerce 站点受益于 Drupal 的安全性、扩展能力，以及其广泛选择的贡献模块的互操作性。凭借内置在核心本身的内容管理和社交商务工具，Drupal 是当今在线企业的一个非常强大的平台，无需与外部电商应用程序集成。



### 主要特性

核心功能集的范围有意进行了限制，因为 Drupal Commerce 的目标是为网站构建者和开发者提供电子商务的构建模块，以便他们能够创建定制化的电子商务解决方案。尽管如此，Commerce 模块仍然包含了电子商务应用所期望的基本功能集。这些功能主要通过某些核心模块对中的用户界面（UI）模块（如 Product 和 Product UI）以及专注于客户体验的纯 UI 模块（如 Cart 和 Checkout）来实现。

核心功能的基本总结包括：

*   支持任意数量可配置产品图片和数据字段的商品。
*   动态产品定价，允许基于 UI 的折扣和含税价格显示。
*   基于 Drupal 7 字段系统的灵活产品展示，强制将产品定义与展示点分离。
*   “智能”添加到购物车表单，根据其所代表的产品数量和类型以不同方式显示。
*   购物车系统，包含购物车区块和购物车更新表单，购物车作为特殊订单对象实现。
*   订单由订单项、客户档案引用和其他元数据组成。
*   各种类型的订单项，用于描述订单上的项目，如商品、税费、运费等。
*   各种用户可配置类型的客户档案，允许收集完成或履行订单所需的数据。
*   灵活的结账表单生成器，支持拖放式用户界面，可实现单页和多页结账。
*   支付系统，将现场和场外支付解决方案集成到正常结账工作流程中，并允许管理员进行跟踪和手动录入。
*   完整的订单、客户档案和支付交易日志记录。
*   支持多币种和多语言商店。

### 深入探索 Drupal Commerce

您可以在本章后续部分以及项目主页 `drupalcommerce.org` 上了解更多关于 Drupal Commerce 的起源、理念和核心 Drupal 创新。首先深入探究 Commerce 模块本身，将为您提供更高级主题的适当背景。因此，本节将介绍如何下载和安装 Commerce 模块，并通过一个简单商店配置的演练，详细检查构成项目核心系统的各种实体和字段。

该项目的源代码托管在两个位置：一个是在 GitHub 上的开发 Git 仓库，它会镜像到 `drupal.org` 上的一个仓库。要快速入门，您可以直接从项目页面 `drupal.org/project/commerce` 下载最新版本，并将其解压到您网站的模块目录。如果您计划向项目贡献代码，或者想基于最新代码进行开发，您可以克隆 Git 仓库，并按照 *Code Workflow* 手册（位于 `drupalcommerce.org/development/workflow`）中的说明，从最活跃的开发仓库中拉取代码。

> **提示** Drupal Commerce 项目负责人 Ryan Szrama 维护着最活跃的开发仓库，提交到主项目仓库的大部分代码都源于此。要找到他及其他开发者的活跃仓库，您可以查阅位于 `drupalcommerce.org/development/workflow/repositories` 的开发者文档。

在启用 Commerce 模块之前，您还需要下载并将以下依赖项的最新 Drupal 7 版本解压到您网站的模块目录中：

*   Address Field (`drupal.org/project/addressfield`)
*   Chaos tools suite (`drupal.org/project/ctools`)
*   Entity API (`drupal.org/project/entity`)
*   Rules (`drupal.org/project/rules`)
*   Views (`drupal.org/project/views`)

有了这些模块后，您就可以开始启用用于构建商店的模块了。如果您是从 Drupal 7 的标准安装开始，那么您应该已经启用了 Commerce 模块所依赖的可选核心 Drupal 模块：Contextual links 和 Field UI。如果这些模块未启用，您应该在启用从已下载项目中导入的以下依赖模块时启用它们：

*   Address Field
*   Entity CRUD API
*   Entity Tokens
*   Rules
*   Rules UI
*   Views
*   Views UI
*   Chaos tools suite 中列为 Views 模块依赖项的模块

> **提示** 虽然不是依赖项，但强烈建议使用 Administration Menu 模块，以便结合核心的 Overlay 模块在 Commerce UI 中导航。您可以从其项目页面 `drupal.org/project/admin_menu` 下载最新的 Drupal 7 版本。

现在，您已准备好启用 Commerce 模块。虽然模块安装页面上的 Commerce 字段集是按字母顺序列出模块的，但此处是按依赖关系顺序列出的：

*   *Commerce/Commerce UI* 定义了 Commerce 模块通用的特性和 API 函数，例如货币处理和 Field API 辅助函数。
*   *Price* 定义了一个具有多个显示格式化器的动态价格字段。
*   *Product/Product UI* 定义了产品实体以及用于创建和管理产品类型及产品的用户界面。
*   *Physical Product* 定义了用于销售实体产品的字段。
*   *Line Item/Line Item UI* 定义了订单项实体、用于模块定义订单项类型的 API，以及用于将订单项添加到订单的订单项引用字段。
*   *Product Reference* 定义了用于在其他实体上展示产品的产品引用字段，以及一个产品订单项类型。
*   *Product Pricing/Product Pricing UI* 实现了基于 Rules 的产品销售价格计算，以实现动态产品定价。
*   *Tax/Tax UI* 通过 API 和用户界面定义了税费类型，用于定义税率和管理含税价格显示。
*   *Customer/Customer UI* 定义了客户档案实体、用于创建和管理客户档案类型及档案的用户界面，以及用于将客户信息添加到订单的客户档案引用字段。
*   *Order/Order UI* 定义了订单实体以及用于创建和管理默认订单类型及订单的用户界面。
*   *Payment/Payment UI* 定义了支付交易实体以及用于通过结账表单和管理表单接受和管理支付的用户界面。
*   *Checkout* 定义了一个灵活的结账表单，带有支持单页和多页结账的拖放式结账表单生成器。
*   *Cart* 定义了特殊的购物车订单状态和用户界面组件，如购物车区块、更新表单以及结账集成。

安装后，其中一些模块将自动创建字段以配置某些实体供使用。例如，当启用 Product Reference 模块时，Line Item 模块会基于 Product Reference 对 `hook_commerce_line_item_info()` 的实现，定义一个带有默认价格字段的新订单项类型。本节的剩余部分将检查每个模块定义的系统和特性，并重点介绍需要进一步配置的方面。这些配置任务包括：

*   启用支持的货币并设置默认商店货币。
*   创建产品类型并添加产品。
*   创建产品展示节点类型。
*   启用支付方式。
*   自定义结账表单。
*   审查默认 Rules。

> **提示** 由于运行 Drupal Commerce 涉及大量模块，您可以使用 Commerce Kickstart 安装配置文件，该配置会在正常的 Drupal 安装过程中自动安装必要的模块。要查找和下载安装配置文件以及寻找创建自己配置文件的资源，请参阅 `drupalcommerce.org/development/installation-profiles` 上的文档。



#### 商务

`Commerce`模块定义了一系列 API 函数，供其他模块用于简化视图和表单 API 的常用功能。虽然随着 Drupal Commerce 的成熟，这个函数库和通用商店设置库肯定会不断增长，但目前其主要负责的功能是货币定义和格式化。`Commerce`模块根据 ISO 4217 标准定义了每一种可能的货币，同时允许通过`hook_commerce_currency_info_alter()`修改货币名称和格式数据。

`Commerce UI`模块定义了一个名为`商店`的顶级管理菜单项，所有其他 UI 模块都将其菜单项放置在该菜单项下，并在`商店`下放置一个`配置`项来容纳商务模块的设置表单。它还在`配置`菜单中添加了一个`货币设置`项，如图 25-1 所示。在向网站添加产品之前，您应该为商店选择默认货币，并启用您打算在产品定价中使用的任何其他货币。由商务模块添加到产品、订单项等实体捆绑包中的默认价格字段，在货币选择组件上会使用默认的商店货币，因为这些“锁定”字段的默认值无法通过字段 UI 进行调整。

![图片](img/2501.jpg)

**图 25-1.** 货币设置表单允许您指定默认的商店货币并启用任何其他货币。

#### 价格

`Price`模块主要定义了可以附加到实体上的价格字段，从而允许输入特定货币的价格。该价格字段为每个价格值存储一个金额和一个 ISO 4217 货币代码，并附带两种显示格式化器，可将价格值显示为原始数字或针对指定货币格式化的数字。可以通过图 25-2 所示的两种默认组件之一来输入价格：一个`价格文本字段`组件，允许为特定货币输入价格；以及一个`带货币的价格`组件，允许您在数据输入时从已启用的货币列表中选择一种货币。

![图片](img/2502.jpg)

**图 25-2.** `Price`模块定义了这两种用于输入价格值的标准组件。

虽然`Price`模块本身不会创建任何价格字段，但它确实提供了一个名为`commerce_price_create_instance()`的 API 函数，Drupal Commerce 中的其他模块使用该函数将其必需的、锁定的价格字段添加到它们的实体捆绑包中。`Product`模块使用此函数为产品类型添加默认价格字段，以确保您创建的每个产品都有一个价格值，如下所示：

```
/**
  * 确保基础价格字段存在于产品类型捆绑包中。
  */
function commerce_product_configure_product_type($type) {
  commerce_price_create_instance('purchase_price', 'commerce_product', $type, t('价格'));
}
```

如果对`commerce_price_create_instance()`的多次调用使用了相同的第一个参数（代表字段名称），则每个实例将使用相同的字段。对于产品而言，这允许视图和其他函数假设它们可以在所有产品类型的相同字段中找到价格数据。如果没有这种能力，几乎不可能构建可靠的产品目录和多用途的“加入购物车”表单。

此外，使用此函数创建的价格字段是“锁定”的，意味着这些字段的实例无法通过字段 UI 删除或修改。因此，它们在数据输入时默认使用`带货币的价格`组件，因为它使用默认的商店货币，并根据可用货币数量更改其表单元素。虽然该组件也能兼容那些在输入价格后又被禁用的货币，但您应将规划并在输入产品数据之前启用网站将使用的货币视为最佳实践。

#### 动态定价

价格字段有能力使其他模块在显示时动态更改价格的金额和货币。为各种折扣更改价格，以及为含税定价和多货币支持等功能调整价格显示，是许多电子商务网站的主要考量。`Price`模块通过`Product Pricing`模块与`规则`的集成交互来满足这些需求，允许您通过`规则 UI`配置价格调整。

仅根据显示点或某组折扣参数来更改价格本身并非难事，但由于使用`规则`进行动态定价需要 Drupal 加载并执行代码，因此这些数据无法直接在数据库中用于排序或过滤查询结果。换句话说，如果一个视图按价格从低到高对产品进行排序，那么当生成页面时，如果最贵的产品被打折成为列表中最便宜的，排序就会出错。为了避免这个问题，`Price`模块能够预计算并缓存源自独立`规则`的价格，这些规则使用一组一致的参数来产生可预测且可复现的价格调整。

#### 产品

在完成初始商店和货币配置后，下一步将是实施您的网站产品策略。当您开始与产品系统交互时，应该清楚了解您将销售哪些类型的产品以及您的客户将如何为它们付款。`Product`模块利用 Drupal 7 的实体系统定义了一个新的、可字段化的产品实体，该实体可以拥有任意数量的捆绑包，这些捆绑包被称为产品类型。该实体还定义了多种显示模式，允许您控制每个产品类型上的字段在不同位置的显示方式。

任何模块都可以使用`hook_entity_info()`以相同的方式定义新实体。当您为 Drupal 7 贡献模块或编写自己的模块来扩展 Drupal Commerce 时，如果您的代码依赖于一个具有可通过用户界面配置的捆绑包的可显示数据对象，则应考虑使用实体系统。请参阅本章稍后的开发讨论部分，了解讨论实体类型定义的代码示例。



#### 创建商品类型

除了在代码中定义商品类型外，您还可以使用商品 UI 模块提供的默认界面来添加和配置商品类型。该界面位于“商店”菜单下的“商品”菜单项中。商品主页面是一个视图，以表格形式列出站点上的所有商品，并带有一个“商品类型”选项卡，该选项卡列出所有当前可用的商品类型，并附带一些用于管理这些类型及其字段的管理链接（参见图 25-3）。

![images](img/2503.jpg)

**图 25-3。** “商品类型”选项卡允许您在站点上添加和配置商品类型。

虽然商品 UI 模块在安装时会创建一个基础商品类型，但大多数站点都需要定义额外的商品类型，或至少自定义这个基础商品类型。您应该为您销售的、共享一组共同属性或特征的每组商品添加一个独立的商品类型，例如衬衫的尺码或书籍的封面类型。这些属性在各类商品类型上以字段的形式表示，并通过商品类型的“管理字段”链接进行添加。正如后面“购物车”部分所讨论的，“添加到购物车”表单会调整其显示方式，以便根据表单上每个商品所包含的、具有明确选项集的任何必需的单一值字段，来进行商品选择。

请按照以下说明构建一个可能在服装店中使用的 T 恤商品类型：

1.  点击上面显示的“添加商品类型”链接，然后输入 *t-shirt* 作为商品类型名称。请注意，系统会自动为该商品类型创建一个有效的“机器名称”，该名称将在整个代码中用于指代此类型。
2.  使用“保存并添加字段”按钮提交表单，以创建新的商品类型并重定向到其“管理字段”选项卡。
3.  将字段表格中的“添加新字段”行拖拽到“标题”行和“价格”行之间的位置。在新字段的“标签”文本框中输入 *Size*，在其“名称”文本框中输入 *size*。选择“列表（文本）”作为要存储的数据类型，将“小组件”保留为“选择列表”，然后点击“保存”按钮以创建新字段并重定向到其设置表单。
4.  将字段设置的“允许值”保留为空，因为这些允许值适用于您的“尺码”字段的任何实例，如果您之后决定需要更多选项，将无法更新。使用“保存字段设置”按钮提交表单。
5.  您现在看到的表单包含特定于 T 恤商品类型上“尺码”字段的设置以及其他通用字段设置。在“T 恤设置”字段集中，选中“标签”文本框下方的“必填”复选框，以要求每个 T 恤商品都必须有一个尺码值。在“尺码字段设置”字段集中，确保“值的数量”设置为 1，并在“允许值”文本区域中每行输入几个尺码选项。
6.  使用“保存设置”按钮提交表单，将返回到“管理字段”选项卡，此时该选项卡应类似于图 25-4。您可以使用相同的过程添加任何其他需要的字段，包括图像等其他类型的字段。

![images](img/2504.jpg)

**图 25-4。** 商品类型的“管理字段”选项卡显示了模块添加的默认字段以及您通过字段 UI 添加的任何其他字段。

#### 添加商品

一旦配置了商品类型，您就可以开始向站点添加商品了。返回到 *商店* ![图片](img/U001.jpg) *商品* 页面，您可以使用“添加商品”链接来选择商品类型并开始创建商品。当您列出具有单一值字段（如 T 恤商品类型）的商品时，您必须为您打算销售的每个变体添加一个独立的商品。

敏锐的观察者会意识到，对于具有多个属性字段和许多选项的商品类型的商店来说，这将产生多少商品。在 Drupal Commerce 中，您最终可能会创建几个到几十个仅在某个字段值上有所不同的商品。这在很大程度上是由于该项目优先考虑规范化商品数据模型，并且是其强调将 API 与 UI 分离（后文将讨论）的延伸。

Drupal Commerce 的诞生始于讨论如何更好地定义商品，以改善 Ubercart 商品 API 中糟糕的开发体验——这种糟糕体验源于属性数据的不一致、SKU 调整的不可靠性以及作为序列化数组存储的关键数据的不透明性。设计到 Commerce 商品系统中的主要纠正措施是强制定义商品的每一种可能的变体，包括唯一的商品 ID 和 SKU。这种方法结合了在字段中存储属性数据的方式，使得商品数据更易于处理，并简化了商品创建 API。

![images](img/square.jpg) **注意** SKU 代表库存量单位，指的是商家为每种商品变体或可计费实体定义的唯一标识符。SKU 通常包含有意义的缩写商品信息，但也可以只是简单的数值，特别是对于那些不依赖这些数据的商店，或者市场上其他跟踪已售商品的方法占主导地位的情况下。

这种为了简化核心 UI 而专注于 API 和数据模型的转变是有意为之的。商品策略的一个关键部分是在商品系统之上引入易用性层，以简化重复性任务。Drupal 的 2010 年谷歌编程之夏项目之一就专注于解决这个问题，从而产生了可从 `drupal.org/project/commerce_bpc` 获得的 Commerce 批量商品创建模块。如果您需要一次性创建同一商品的多个变体，该模块允许您通过一个表单来创建它们，您可以在该表单中选择要为其创建商品的属性字段选项，并指定一个基于令牌的模式用于为其生成 SKU。

至此，您仍然没有将您创建的商品展示给顾客进行购买的方式。商品定义仅存在于后端，并将通过利用商品引用字段的商品展示节点类型在前端进行显示，如“商品引用”部分所述。如前所述，将商品定义与其展示点分离，是 Drupal Commerce 强调将 API 与默认 UI 分离的延伸。这种分离允许同一商品在多个地方被引用，例如在特定语言的节点上或跨多个域，而无需手动数据同步来确保商品 SKU、价格和其他信息在所有展示中都保持一致。虽然设置初始商品展示可能需要更多工作，但更大的灵活性绝对物有所值。

最后，使用 Drupal 的实体系统来定义商品，使得模块能够使用特殊字段为不同的商品类型添加功能。物理商品模块正是这样做的，它定义了可用于描述商品尺寸、重量和包装信息的字段。这些数据随后在结账表单中可用，在那里它们可以被汇总，用于计算运费，并收集完成实体商品订单所需的用户额外信息。



### 订单项

Drupal Commerce 使用 `line items`（订单项）来表示订单上任何有助于计算订单总额或完成订单的内容。该模块定义的 `line item` 实体是可配置字段的，并且可以配置为任意数量的模块定义包，这些包被称为 `line item types`（订单项类型）。对 `line items` 的更改会通过 `line item revisions`（订单项修订）进行追踪，这与节点类似。除了由定义类型的模块添加的特定于 `line item type` 的字段之外，每个 `line item` 都包含以下默认属性和字段：

*   标签
*   标题
*   显示选项
*   数量
*   单价
*   总价

任何需要在订单上表示其他信息（例如来自优惠券代码的折扣）的模块，都可以使用 `hook_commerce_line_item_info()` 以相同的方式定义一个新的 `line item type`。在产品引用模块中可以看到一个示例实现，该模块定义了 `product line item type`，并使用默认的产品引用字段将 `line item` 与数据库中的实际产品关联起来。有关 `line item type` 数据结构的当前文档，请参阅 *Drupal Commerce 规范* 手册中“信息钩子”部分的“订单项”页面，网址为 [`www.drupalcommerce.org/specification`](http://www.drupalcommerce.org/specification)。

![images](img/square.jpg) **提示** 收藏规范手册（`drupalcommerce.org/specification`）并经常查阅，以获取最新的系统概览、钩子说明和 API 使用策略。

由订单项模块定义的 `line item reference` 字段可以使用其 `line_item_id` 值将任意数量的 `line items` 关联到另一个实体。该字段本身除了 ID 之外不存储任何数据，但它附带了一个非常强大的订单项管理器小部件，该小部件允许你通过由表单 API 中新的 `#ajax` 支持驱动的动态表单来添加、编辑和删除 `line items`（请参阅 图 25-5）。像购物车这样的模块希望通过 API 向订单添加 `line items`，它们负责创建 `line items` 并通过调整订单的 `line item reference` 字段的值将它们与订单关联起来。请参考 `commerce_cart.module` 中的 `commerce_cart_product_add()` 函数，以了解此过程的示例实现。

![images](img/2505.jpg)

***图 25-5.** 订单项管理器小部件可用于在订单上添加、编辑和删除订单项。*

`Line items` 可以通过 Views 显示，方法是使用 `line item` 关系或参数（例如在购物车区块和订单项视图显示格式化程序中），或者通过 API 基于 `line item type` 的字段显示配置构建内容数组。你可以通过商店 ![Image](img/U001.jpg) 配置菜单中的订单项选项来访问 `line item types` 列表并管理其字段显示设置。请注意，`line item types` 无法通过用户界面添加或编辑，因为它们依赖于模块特定的代码才能有效运行。如果你必须更改另一个模块的 `line item types`，可以使用 `hook_commerce_line_item_info_alter()` 来实现。

除了产品引用和购物车之外，税收是另一个与 `line item` 系统进行大量交互的核心模块。税收模块允许模块通过 `hook_commerce_tax_info()` 定义可能应用于订单中 `line items` 的税率。将征收的税款存储在 `line items` 中，你可以使用 Views 轻松创建税收报告，并通过引用用于计算 `line item` 价格的特定税率的字段来访问税率信息。

### 产品引用

产品引用模块定义了一个产品引用字段和 `product line item type`，后者使用该字段从 `line item` 引用产品数据。产品引用字段是你用于构建产品展示和“添加到购物车”表单的主要工具。当该字段放置在节点类型上时，产品引用模块会将所引用产品的字段拉取到节点中进行展示，同时使用产品类型和节点类型的字段显示设置来格式化产品字段并将其与节点字段一起排序。产品引用字段可以引用单个或多个产品，并且通过购物车模块提供的“添加到购物车”表单显示格式化程序，可以在节点展示中显示为“添加到购物车”表单。该字段还与 Views 集成，从展示节点到被引用产品的数据建立关系，用于产品目录 Views 和其他类型的展示。

如你所见，这个字段非常多功能，是 Drupal Commerce 产品策略的关键部分。通过将产品字段拉取到节点展示中，可以在单个产品实体上定义和维护图像和价格信息，然后从网站上的任意位置引用它们。对产品的任何更新都将自动出现在任何使用产品引用关系展示该产品的地方。



#### 构建产品展示节点类型

请按照以下说明为您的网站构建一个基本的产品展示节点类型：

1.  在管理菜单中浏览至 `结构` `![Image](img/U001.jpg)` `内容类型`，然后点击 `添加内容类型` 链接。输入 `产品展示` 作为内容类型名称，并根据需要调整垂直选项卡中的设置。
2.  使用 `保存并添加字段` 按钮提交表单，以创建新的内容类型并重定向到其 `管理字段` 选项卡。
3.  在字段表的 `添加新字段` 行中，输入 `产品` 作为新字段的标签，并在其名称文本框中输入 `product`。选择 `产品引用`，将小部件保留为 `自动完成文本框`，然后点击 `保存` 按钮创建新字段并重定向到其设置表单。
4.  保持 “可引用的产品类型” 复选框未选中状态，允许该字段引用任何类型的产品。使用 `保存字段设置` 按钮提交表单。
5.  您现在正在查看一个表单，其中包含 `产品展示` 内容类型上特定于产品字段的设置以及其他通用字段设置。在 `产品展示设置` 字段集中，勾选 `标签` 文本框下方的 `必填` 复选框，以要求每个 `产品展示` 节点都引用一个产品。在 `产品字段设置` 字段集中，将 `数值数量` 设置为 `1`，然后使用 `保存设置` 按钮提交表单，返回 `管理字段` 选项卡。
6.  点击 `管理显示` 选项卡，查看内容类型字段（`正文` 和 `产品`）以及引用的产品字段（`产品：尺寸` 和 `产品：价格`）将如何排序和显示。只有 T 恤产品才会有尺寸字段，您无需通过产品展示节点来显示该字段本身。您还需要将节点的产品字段显示为“加入购物车”表单，这需要在字段的 `格式` 选择框中选择该显示格式器。将您的默认字段显示设置更新为类似于 图 25-6 的样子，并对其他显示模式重复此过程。

![images](img/2506.jpg)

***图 25-6.** 如图所示，字段显示配置隐藏了不必要的字段，并调整了引用产品价格字段的顺序，使其显示在呈现为“加入购物车”表单的产品引用字段上方。*

7.  最后，您应该检查将在节点上下文中显示的产品字段的显示设置。这些设置通过每种产品类型的“管理显示”表单进行配置，这些表单位于 `管理` `![Image](img/U001.jpg)` `商店` `![Image](img/U001.jpg)` `产品` `![Image](img/U001.jpg)` `产品类型` 下。产品字段可以在每个节点视图模式中显示不同，因此您可能需要为多个产品视图模式配置字段显示设置。

完成这些步骤后，您将获得一个节点类型，现在可以用于在您的网站上列出待售产品。`自动完成文本框` 小部件会在 `节点编辑` 表单中添加一个文本字段，您可以在其中通过 SKU 输入产品以从节点引用，并且在数据输入过程中会根据产品 SKU 或标题自动完成。这种类型的节点将显示一个简单的“加入购物车”表单，将引用的产品添加到客户的购物车中。

虽然这种简单的产品展示适合一次销售单个产品，但对于从同一节点销售产品组合来说是不够的。例如，您可能想创建四件 T 恤产品，它们款式或设计相同但尺寸不同。Drupal Commerce 的产品数据模型要求将每种尺寸列为单独的产品，但客户只需看到一个单一的产品展示，在其中可以选择合适的尺寸。为了实现这一点，您需要使用一个包含产品引用字段的内容类型，该字段的 `数值数量` 设置大于 1 或设置为 `无限制`。自动完成文本框的功能相同，但支持以逗号分隔的 SKU 列表，并且“加入购物车”表单将自动调整，以允许客户选择要添加到购物车的合适产品。

在展示节点中包含产品字段是整个系统的关键功能，但这并非没有困难。对于单值产品引用字段，将所引用产品的字段提取到节点中进行显示非常容易，并且很明显，产品的最佳显示数据是存储在产品类型的字段中。最常见的例子是向产品类型添加一个图片字段，并将图片上传到产品本身，以便它们可以轻松地在产品展示节点或其他自定义显示和视图上显示。然而，对于多值产品引用字段，更难确定默认情况下应显示哪个产品的字段，以及某些数据是否应存储在产品上，尽管存在重复的可能性，例如仅作为单独产品列出以适应各种尺寸选项的 T 恤图片的情况。随着 Drupal Commerce 的成熟，您的方法可能会改变，但目前的最佳策略是将尽可能多的关于产品的原始数据（包括任何相关图片）存储在产品本身的字段中。



### 客户

`Customer`模块定义了一个客户配置实体以及相关的客户配置引用字段，其工作方式类似于订单项引用字段，用于将客户配置数据与订单关联。客户配置实体支持任意数量的、由模块定义的可字段化包，这些包被称为客户配置类型。对客户配置的更改会通过修订版本进行记录，并且特别注意确保客户配置被复制而非仅仅更新，以保留被先前订单引用的客户配置数据。每个客户配置类型可以拥有自己的一组字段来收集与该配置类型相关的数据，从而允许您在必要时为账单地址和收货地址配置收集不同的信息。像默认的账单信息类型这样的客户配置类型，会使用`地址字段`模块定义的国际地址字段来收集符合国际标准的姓名和地址信息。

客户配置旨在作为从客户处收集完成订单所需信息的主要方法。这些信息与常规的用户账户系统分开维护，以提供几个关键的灵活性点。首先，这种模式允许用户维护每种类型的多个配置，这类似于大多数主要电子商务网站常见的地址簿功能。其次，允许回头客引用以前使用过的配置，仅在信息发生变化时才创建新配置，从而减少数据重复。第三，允许团体购买的店铺可以授予多个用户访问同一客户配置信息的权限，并将创建新配置的能力委托给每个团体的管理员。第四，客户配置数据可以针对匿名用户的订单进行收集和存储，这样如果店铺不希望的话，就无需创建用户账户。如果客户数据直接与用户账户绑定，这种级别的匿名结账是不可能实现的。

客户配置类型通过 `hook_commerce_customer_profile_info()` 由模块定义。与订单项类型一样，模块可以在首次启用这些配置类型时执行配置步骤，以确保存在默认字段。`Customer`模块为每种客户配置类型提供了结账表单集成，为客户提供提供信息的地方，并且它会向订单对象添加客户配置引用字段，为管理员提供添加和编辑配置的位置。与订单项管理器小部件不同，客户配置管理器小部件一次只支持引用单个配置。最后，该模块在“商店”菜单中添加了一个“客户配置”项目，允许您创建、查看、更新和删除客户配置，并有一个单独的选项卡允许您查看所有客户配置类型的列表并管理它们的字段。您添加到客户配置类型的任何字段都将出现在结账表单、订单编辑表单以及客户配置显示页面上。

### 订单

订单系统由订单实体、订单状态和状态信息以及一个旨在帮助您处理订单数据和更新订单的 API 组成。订单实体定义了一个支持对订单数据的任何更改进行修订的单一可字段化包。从营销和安全角度来看，跟踪订单在其整个工作流程中的变化非常重要，这允许管理员跟踪客户在结账前与网站的交互，以及跟踪其他管理员之后对订单所做的更新。因此，特别注意确保订单及其关联的订单项和客户配置也在必要时进行修订。

除了前面提到的订单项和客户配置数据，订单还包含跟踪订单状态、创建和更新时间戳以及所有者信息的元数据。订单状态是订单生命周期中的当前步骤，为管理员提供有关订单已发生什么以及订单处理下一步将是什么的信息。订单状态的范围从与购物车相关和与结账相关的状态，到以完成状态结束的各种结账后状态。它们被组织到称为订单状态的容器中，这些容器代表了订单经历的更大阶段。所有者信息包括创建订单的用户 ID（无论该订单是通过购物车还是管理表单创建的）以及一个联系电子邮件地址，如果用户已登录，该地址默认为用户的电子邮件，但如果用户在结账表单上以匿名方式提供，则可能不同或由匿名用户提供。

订单管理的默认用户界面是一个视图，该视图列出所有订单，并在“商店”菜单的“订单”项中提供一个订单创建链接，以及在“商店”![图片](img/U001.jpg)“配置”菜单中有一个包含字段管理选项卡的设置区域。默认的订单视图首先显示最近的订单，并显示订单号，该订单号可以是任何字母数字值，即使它默认为订单 ID，以及包括订单总额和当前状态在内的其他元数据。

虽然默认的订单视图相当基本，但正是在这里，将 Drupal Commerce 的默认 UI 标准化到 Views 上的决定真正得到了回报。许多店铺需要定制的订单管理界面，以适应其独特的订单工作流程和履行需求。站点构建者可以使用熟悉的 Views 界面和主题系统来定制现有界面，添加额外的排序和过滤选项，并通过各种贡献的 Views 模块（例如用于批量更新的 Views Bulk Operations）对其进行扩展。

### 支付

`Payment`模块定义了一个支付交易实体，该实体记录任何由模块定义的支付方式的支付尝试，并通过结账流程或订单的支付表单将这些交易与订单关联起来。支付方式由贡献模块为任何给定支付服务提供的每种可能支付方式所定义。如前所述，核心项目中不包含真正的支付方式，以使支付服务集成代码能够独立于核心开发周期而成熟。`Payment`模块确实为常见的支付方式类型提供了可重用的代码，您应该在自己的集成模块中重用这些代码，例如一个标准化的信用卡数据输入表单。

每当进行支付尝试时，都会创建支付交易，记录尝试的时间和细节、支付服务返回的数据以及支付的结果或当前状态。支付交易实体为每个启用的支付方式定义了包，但它不允许向这些包附加字段。对支付交易的更新通过修订版本进行记录，但从支付服务接收的负载数据除外。该数据维护在一个序列化数组中，每个与交易相关的消息都有一个新值，并且仅对支付管理员可见，用于调试目的。

当为订单创建支付交易时，它们会显示在订单的“付款”选项卡页面上的一个付款选项卡中。此选项卡包含一个视图，该视图按时间顺序列出订单的所有交易，其页脚包含待支付的余额以及手动输入付款的表单。在开发和测试期间，您可以利用 `Payment Method Example` 模块来测试付款的接收和记录，从而产生完全支付的订单，如图 25-7 所示。

![images](img/2507.jpg)

***图 25-7.** 付款按时间顺序列在“付款”选项卡上，您还可以在此找到对付款执行任何必要操作的链接，以及手动向订单添加新付款的表单。*



#### 启用支付方式

“支付方式示例”模块也可作为开发自定义支付方式模块的模型。定义新的支付方式并非极其复杂的过程，主要涉及实现 `hook_commerce_payment_info()`，并定义用于收集和传递客户信息到支付服务的回调函数。对于需要重定向到第三方网站提交付款详情的支付方式，还有额外的适配措施，以确保这些支付方式能正常融入标准的结账流程。与订单项类型文档类似，请参考 *Drupal Commerce 规范*手册中“信息钩子”部分的“支付”页面，以获取关于正确集成支付系统的最新信息。

支付模块定义了两个用于在结账表单中处理客户付款的结账窗格。基本的“付款”窗格会在“审核结账”页面向客户展示所有可用的支付方式以供选择，并且会自动更新，以包含收集所选支付方式所需处理流程的附加表单元素。如果选择了站外支付方式，客户将通过“站外支付重定向”结账窗格从付款结账页面被重定向，该窗格也能处理从站外支付服务返回的客户。对于可以直接在您的网站上处理的支付方式，此页面将被跳过。

可用支付方式的列表通过与“规则”模块的集成来确定。每种支付方式都会收到一个默认的规则配置，您必须启用该配置，支付方式才会出现在结账表单中。每种支付方式通常还需要在启用它的规则配置的操作表单中进行额外设置。此外，如有必要，您可以在规则配置中添加客户使用该支付方式时必须满足的任何条件。

![images](img/2508.jpg)

***图 25–8.** 支付方式通过规则配置启用，此处是为示例支付方式进行的配置。*

请按照以下说明，为拥有“管理员”用户角色的用户启用“示例”支付方式，从而得到如图 25–8 所示的规则配置：

1.  浏览至“商店”![Image](img/U001.jpg)“配置”菜单中的“支付设置”页面。
2.  在“禁用的支付规则”表格中，点击“示例”支付规则配置对应的“启用”操作链接，并在后续的表单中确认该操作。
3.  点击该规则配置的“编辑”操作链接，查看总览表单，其中会列出触发此规则的事件（“为订单选择可用支付方式”）、执行时检查的条件，以及评估成功时要执行的操作。唯一的操作将是“启用支付方式：示例支付”操作。通过“编辑”操作链接查看其配置表单，了解通常应在何处输入支付方式设置。
4.  返回该规则配置的总览表单，点击“条件”表格底部的“添加条件”链接。选择“用户拥有（若干）角色”这一条件，然后通过“继续”按钮提交表单。
5.  现在您需要告诉规则引擎应检查哪个用户的哪个角色。在“用户”字段集中，在“数据选择器”文本框中选定或输入 `site:current-user`。此令牌告诉规则引擎使用当前登录用户来评估此条件。在“角色”字段集中，将“管理员”指定为“值”，然后通过“保存”按钮提交表单。
6.  您的总览表单现在应类似于图 25–8，这将使得拥有“管理员”角色的用户在结账表单上能够使用“示例”支付方式。

对于您希望在结账表单上启用的任何其他支付方式，都需要重复此过程。不过，管理支付表单不依赖于规则，它会显示管理员可以处理的任何可用支付方式。重定向支付方式通常无法在此工作，因为它们通常依赖于结账特定的信息或客户的用户名和密码。

![images](img/square.jpg) **提示** 大多数主流支付服务都有与之集成的 Drupal Commerce 模块，因此在开发自己的集成模块之前，请先在 `drupal.org` 上搜索电子商务模块。



### 结账

结账系统由一个可插拔的结账表单和一个管理端的结账表单构建器组成，后者允许你通过拖拽界面来排序和配置结账表单的组件。结账表单由包含一组模块定义的结账窗格的结账页面构成，这些窗格是用于显示订单详情以及收集客户与支付信息的字段集。

虽然该表单默认采用两步流程，并在独立页面上进行审核与支付，但只需点击几下鼠标，即可配置为单步流程：表单提交时即处理订单，成功时立即重定向至完成页面。这种配置是否适合你的商店，取决于你所使用的支付方式以及商店是否具有可能需要额外结账步骤的业务规则。

![images](img/square.jpg) **提示** 两步流程是默认设置，这样客户可以在提交支付信息之前，先查看其订单的完整详情。出于安全原因，某些支付方式必须在提交表单时立即处理，这意味着支付应在结账完成前的最后一步进行。在决定实施单步结账表单之前，请理解你所用支付方式的限制。

结账表单的目的是收集为正确履行订单所需的任何信息，包括在“支付”部分讨论过的支付处理。随着订单在结账表单中逐步推进，其状态会相应更新以反映当前所在的页面，这让你能够保留订单在结账过程中被放弃时的位置信息。订单上的数据对于挽回这些销售、简化结账流程以提高转化率可能至关重要。

由结账模块和支付模块定义的默认结账页面包括：

*   *结账*：显示购物车内容并收集客户资料信息。
*   *审核订单*：显示已输入数据的摘要并收集支付详情。
*   *支付*：是跳转到站外支付方式的转向点；如果不需要则跳过。
*   *结账完成*：是最终着陆页，显示订单摘要、相关的订单链接以及订单履行信息。

每个页面上出现的结账窗格是完全可定制的，任何没有窗格的页面都将在结账工作流中被跳过。你可以使用此前在《Drupal Commerce 规范》手册中“信息钩子”部分的“结账”页面上描述的钩子，向图 25-9 所示的结账表单构建器暴露额外的结账页面和窗格。

![images](img/2509.jpg)

***图 25-9.** 可以使用拖拽式结账表单构建器轻松重新配置结账表单。*

实际上，结账模块并不依赖购物车模块，这意味着你可以启用结账表单，但设计其他方法来根据客户行为创建订单，或者为客户提供链接以访问管理员创建的订单。基本的结账 URL 实际上由购物车模块定义，作为指向特定订单结账 URL（`checkout/#`）的路由。结账表单不使用 Drupal 的多步表单功能在单一 URL 上推进，因此随着客户在表单中推进，当前结账页面实际上会反映在 URL 中。

你大多需要自行配置 Drupal，以提供最佳的结账体验。有许多网站和文章讨论如何优化电子商务网站以提高转化率，而 Drupal 的灵活性将很好地帮助你优化结账页面。至少，你应该在结账页面上禁用不必要的区块和菜单，并使用主题功能来突出显示客户应使用的按钮，以便访问结账表单并在其中推进流程。

### 购物车

购物车模块启用了一个相当标准的购物车系统，允许客户将产品添加到购物车订单中，然后通过结账表单进行购买。一旦客户将商品添加到购物车，就会创建一个新订单，该订单将一直存在，直到通过结账完成。有一个默认的“购物车”订单状态和订单状态值，但其他状态值也可以告知购物车模块它们是购物车状态，就像表示结账表单中默认的结账和审核步骤的状态值一样。这使得客户可以在实际提交支付之前更新购物车的内容。

购物车通过一个 Drupal 区块表示，如图 25-10 所示，该区块由一个视图组成，列出购物车订单上的订单项，底部有汇总订单项并链接到购物车页面和结账表单的页脚。该区块相当动态，可以通过视图用户界面轻松定制，并使用主题使其与你的网站匹配。购物车页面提供了一个同样通过视图构建的表单，允许用户更新购物车内容并继续结账。

![images](img/2510.jpg)

***图 25-10.** 默认的购物车区块完全可以通过视图用户界面进行配置。*

购物车 API 包含用于加载和更新购物车订单的功能，以及一个多功能的“添加到购物车”表单功能。“添加到购物车”表单对产品引用字段的显示格式化程序会将引用的产品 ID 传递给此表单，该表单会检查传入的值以决定如何显示该表单。单个产品表单只会为隐藏表单值中指定的产品显示一个“添加到购物车”按钮，而多个产品表单的外观则会有所不同；它们可能显示为单一选择列表、单选按钮组或复选框组，允许客户根据产品标题选择要添加的产品，或者显示为一组动态生成的小部件，代表被引用产品上的通用属性字段。当客户更新所选的产品或属性时，该表单会使用表单 API 的 `#ajax` 属性，在将产品添加到购物车之前，相应地更新页面上的元素。



#### 主要组件概述

关于各系统如何架构，还有更多内容可以阐述，但通过快速审视核心模块，您应能对 Drupal Commerce 各主要部件如何协同工作有一个功能性的理解。您需要掌握的关键点是，尽管核心模块并未提供一个开箱即用的完整电子商务应用，但必要的系统已经就位，可供其他模块扩展，并在网站构建过程中充实完善，以提供您的网站所需的电子商务体验。

表格 25–1、25–2 和 25–3 总结了主要的 Commerce 组件，具体而言，是之前模块讨论中提到的所有实体和字段。

![images](img/t2501.jpg)

**表 25-2** . Drupal Commerce 核心字段

| **名称** | **小部件** | **显示格式器** |
| --- | --- | --- |
| 客户资料引用 | 客户资料管理器 | 客户资料显示 |
| 订单项引用 | 订单项管理器 | 订单项视图 |
| 价格 | 价格文本字段 |  |
| 带货币的价格 | 原始金额 |  |
| 格式化金额 |  |  |
| 产品引用 | 自动补全文本字段 |  |
| 选择列表 |  |  |
| 复选框/单选按钮 | 添加到购物车表单 |  |
| SKU（库存量单位） |  |  |
| 标题 |  |  |

**表 25-3** . 默认订单状态与状态值

| **订单状态** | **订单状态值** |
| --- | --- |
| 已取消 | 已取消 |
| 购物车 | 购物车 |
| 结账 | 结账：结账（功能同购物车） |
| 结账：审核（功能同购物车） |  |
| 结账：付款 |  |
| 结账：完成 |  |
| 待处理 | 待处理（允许访问完成页面） |
| 处理中（允许访问完成页面） |  |
| 已完成 | 已完成（允许访问完成页面） |

### 实施 Drupal Commerce

通读介绍各系统、实体和字段的讨论后，您应能体会到这种设计电子商务系统方法所提供的强大功能与灵活性。然而，您可能也推测到了，构成 Drupal Commerce 的松耦合组件需要您付出额外的用心和专业能力，以确保它们得到成功配置。

老话诚然不虚：能力越大，责任越大。现在，您需要确保在使用这些工具时能按预期行事，同时提供满足网站需求和客户期望的电子商务体验。搭建 Drupal Commerce 站点需要提前进行额外的规划工作，因此本章的最后部分提供了一些技巧，帮助您入门，并减少在开发多个站点时会遇到的一些重复性任务。

由于电子商务只是 Drupal 站点的一部分，您应确保遵循 Drupal 站点构建的最佳实践。提前规划您的内容类型、视图、角色和权限，以确保网站基础稳固且安全。配置 Commerce 组件只是您常规流程的延伸，除了内容类型之外，还需要您规划产品类型，并专门为商店管理添加角色。

此外，您应采取额外预防措施，以确保 Drupal 站点的安全性。计划定期维护以保持 Drupal 和您贡献模块的最新状态；全面测试您的支付系统和结账工作流，以确保支付数据不会泄露，且在支付完成前订单不会被履行。处理站点内信用卡支付时，确保您的结账配置允许包含支付结账窗格的页面在提交时，支付方式模块能够处理支付。您还应利用其他贡献模块来增强安全性，例如使用 Secure Pages 为您的站点添加 SSL 保护。

最后，从项目伊始，目标就是让站点构建者和开发者能够创建可重用的 Drupal Commerce 发行版和针对特定电子商务用例的功能模块。搭建 Drupal Commerce 安装所涉及的工作量，不是您在每个新站点上都愿意重头再来的，这需要您具备大多数新用户难以掌握的 Drupal 熟练程度。随着 Drupal 的定向发行版越来越流行，加之 `drupal.org` 本身自动安装配置文件打包功能的助力，Drupal Commerce 的愿景是看到许多这样的发行版面向电子商务站点，例如：

- 拥有现成产品类型和展示的服装服饰店。
- 销售用户角色、节点访问权限和其他类型权限的付费内容与会员站点，支持固定费用和定期付款。
- 非营利组织的外展与捐款收集站点。
- 基于社区的活动注册与支持站点。

基于这一策略和愿景，Commerce 组件也被设计为可供 Features 和其他支持可导出站点配置的模块使用。此外，为默认用户界面使用视图，为订单工作流自动化使用规则，使得您可以导出您的设置以及 Commerce 配置。对所有 Commerce 对象标准化使用实体和字段，为导入和导出电子商务数据提供了统一的数据模型。利用所有这些部件来开发可重用站点和定制发行版，需要您进行有意识的规划，但无论是在您自己的站点构建工作流中，还是在社区使用方面，其回报都值得付出这些努力。

### 开发历史

Drupal Commerce 的部分特性，最好从其基于之前 Drupal 电子商务项目的发展历程来理解。Drupal Commerce 的功能集和可用性目标源于 Ubercart，其开发哲学受到为 Ubercart 进行开发的社区经验以及 e-Commerce 模块最新版本实施的重大变革的双重影响。然而，Commerce 模块的代码和用户界面与之前的项目几乎毫无相似之处，因为这些新模块是专门为 Drupal 7 编写的，遵循本章后面概述的一套严格的开发标准。

Commerce 最初从 Ubercart 分离时的主要目标包括：

- 为贡献模块开发者建立一个文档更完善、更易于使用的 API。
- 将插件模块与核心系统分离，使每组模块能够独立于其他模块的关注点而成熟。
- 基于 Drupal 7 的实体和字段系统建立数据对象模型，依靠用户界面来减少可能随之而来的重复数据录入。
- 使用 Drupal 的测试框架为核心模块提供完整的测试覆盖。
- 利用安装配置文件和如 Features 等模块来提供默认配置，而不是仅关注模块本身的完整用户体验。
- 提供从 Ubercart 到 Drupal Commerce 的迁移路径。由于功能并非一一对应，因此无法从一个模块直接更新到另一个。

自项目启动以来，开发是分阶段完成的，每个系统在开发之前，都会在项目论坛、IRC 和实体代码冲刺中进行提议和讨论。要了解更多关于项目的历史，并随时了解开发提案和路线图，请参阅 `drupalcommerce.org` 上的论坛和文档。



#### 设计理念

初次浏览 Commerce 模块列表时，有两件事可能会引起你的注意，需要稍加解释。首先是大量可用的模块，以及部分模块被拆分为 API 模块和 UI 模块对。虽然有些用户会因为安装和配置各种模块所需的设置工作而却步，但这种划分是刻意为之的，目的是让同一组模块能够适应更多样化的电子商务网站。随着你的经验增长，你可以为特定网站只启用所需的模块，同时安装配置文件和其他模块可以为常见用例提供默认配置。

例如，通过将默认 UI 与实际定义数据对象和核心 API 的模块分离，网站构建者可以自由地为不适合其网站的 UI 部分提供完整的替代组件。此外，通过将某些系统拆分为其组成部分（例如将购物车和结账流程分离），同一套核心 Commerce 模块可以用于传统的基于购物车的商店、没有结账流程但使用购物车来提供报价的网站，以及由管理员创建订单并向客户发送结账链接以提供付款的发票网站。

![图片](img/square.jpg) **注意：** 对于 Drupal 6 的大多数用户来说，这种做法应该并不陌生，因为从那时起，Views 和 Rules 就已经将它们的默认用户界面分离到单独的模块中。Drupal 7 通过将 Field 系统划分为 Field 和 Field UI 模块，延续了这一做法。

第二件引人注意的事情是，缺少添加常见电子商务功能的模块，例如产品促销工具、折扣和优惠券，以及与第三方支付和履行服务的集成。在 Drupal Commerce 最早的规划阶段，项目的目标是在核心项目中只包含那些必要的系统和数据对象，没有它们项目就无法连贯运行，并且这些系统和数据对象是构建和支持所有必要的非核心功能所必需的。这并不意味着核心模块中未包含的功能就不重要，或者不值得特别关注和维护。相反，这是一项架构决策，旨在通过让核心系统能够迭代到新版本而无需首先更新每个插件模块，从而整体上改进开发过程。

一个很好的例子是，核心项目中包含了 Payment 模块，却没有实际集成主要支付服务的模块。在核心项目中包含集成了常见支付服务的模块可能并不方便，但将集成模块的开发与核心项目解耦，可以允许各个项目按照自己的时间表成熟并整合新功能。核心 Payment 模块可以在不要求先在新系统上测试和更新一批支付模块的情况下进行改进和发布；同时，集成模块也可以自由地响应所集成服务的 API 和功能变化，而无需等待 Drupal Commerce 的新版本发布。

项目设计理念的最后一个主要点是，依赖于安装配置文件和像 Features 这样的模块，以简化针对各种目标用例启动新 Drupal Commerce 网站的过程。核心模块不会对使用它们的网站的用例或业务需求做任何假设。相反，它们专注于保持灵活和可扩展，支持可导出的配置，并开发可以嵌入到任何地方的表单，而不是将其限制在默认的 UI 实现中。该项目的目标是，让发行版能够为利基市场（例如服装店、活动注册网站和非营利组织的捐赠网站）提供开箱即用的体验。

#### 开发标准

除了采用注重模块化和可重复配置的设计理念外，Drupal Commerce 还采用并强制执行了一套严格的开发标准，以简化开发和维护。主要目标是为开发者提供一个文档完善、一致的 API，以便他们在其贡献的模块和安装配置文件中进行集成。不符合 `drupalcommerce.org/development/standards` 列出的开发标准的补丁将不被接受。

虽然这些标准偶尔会因 Drupal 核心更新和需要采用新标准的问题而发生变化，但当前的标准列表涵盖了以下主题：

- 基于 Drupal 自身编码标准的代码语法和文档。
- 模块文件和目录结构。
- 模块 `.info` 文件的包命名。
- 函数和钩子命名，以避免常见的 PHP 不一致问题，并为钩子提供基于模式的命名空间。
- 正确使用 Drupal 的测试框架。
- 使用核心和贡献模块的 API。
- 采用细粒度的权限和可扩展的访问控制。
- 本地化和用户界面字符串存储。
- 模板文件和主题函数。
- 适当分离核心 API 与默认 UI。
- 覆盖数据库和内存使用情况的性能考量。
- 通过实体和字段数据表实现规范化的数据存储。

### 构建于 Drupal 7 之上

Commerce 模块的编写旨在利用 Drupal 7 的许多改进，最显著的是其可字段化的实体系统。它们的灵活性也很大程度上归功于 Views（提供包括购物车和产品列表在内的显示的查询构建器）和 Rules（用于在结账完成和订单更新等事件发生时执行条件操作的贡献系统）。本书的其余部分旨在涵盖 Drupal 7 的许多改进，但这里重点强调几个对 Drupal Commerce 具有特殊意义的内容。

#### 核心实体和字段

Drupal 7 的标志性新功能是其可字段化实体系统。现在，不再需要将所有内容都设为节点才能从像 Drupal 6 的内容构建工具包这样的模块中受益，这促使了对用户组和电子商务等主要系统的重新思考。贡献模块可以标准化实体系统，以定义新的数据对象，这些对象可以与任意数量的模块定义或用户可配置字段捆绑在一起。



### 产品实体

Drupal Commerce 全面拥抱了新系统，将所有自定义数据对象定义为可字段化的实体，并根据实体类型灵活使用包（Bundle）和修订。例如，以往 Drupal 的电商项目依赖节点系统来实现产品，而 Drupal Commerce 则定义了特定的产品实体。该实体拥有多个包，每个包构成一种不同的产品类型，可包含用于描述产品的字段，并支持从“添加到购物车”表单的一组产品中进行选择。

为了定义产品实体，产品模块使用了清单 25–1 中的代码。

**清单 25–1.** 定义产品实体

```
/**
  * 实现 hook_entity_info()。
  */
function commerce_product_entity_info() {
  $return = array(
    'commerce_product' => array(
    'label' => t('Product'),
    'controller class' => 'CommerceProductEntityController',
    'base table' => 'commerce_product',
    'fieldable' => TRUE,
    'entity keys' => array(
      'id' => 'product_id',
      'bundle' => 'type',
    ),
    'bundle keys' => array(
      'bundle' => 'type',
    ),
    'bundles' => array(),
    'load hook' => 'commerce_product_load',
    'view modes' => array(
      …
    ),
    …
  );

  foreach (commerce_product_type_get_name() as $type => $name) {
    $return['commerce_product']['bundles'][$type] = array(
      'label' => $name,
    );
  }

  return $return;
}
```

请注意，实体数组的结构包括与数据库中产品存储相关的键，以及用于通过保存、加载和删除函数对产品执行 CRUD 操作的控制器类 `CommerceProductEntityController`。这些值中的任何一个都可以通过 `hook_entity_info_alter()` 进行修改，以改变产品数据存储的性质，但在做出此类决定时应谨慎，因为这可能会中断其他模块（如视图模块）利用数据的能力。

上述代码片段中省略的部分处理了产品可用显示模式的定义，以及规则模块用于处理产品实体的各种回调函数的定义。`foreach` 循环使用 `commerce_product_type_get_name()` 中的数据填充产品实体的包数组，这是一个 API 函数，它调用 `hook_commerce_product_info()` 从已启用的模块中收集关于可用产品类型的信息。

### 订单实体

同样，订单对象被定义为一个实体，允许管理员根据站点的业务需求轻松地向默认订单包添加字段。然后，这些订单利用实体修订功能来跟踪订单数据的所有更改。

当一个实体未被定义支持多个包时，它默认使用单个包，其机器名称与实体本身相同。对于订单，显式定义了单个包，这使得默认行为变得多余，如清单 25–2 所示。

**清单 25–2.** 显式定义单个包

```
/**
 * 实现 hook_entity_info()。
 */
function commerce_order_entity_info() {
  $return = array(
    'commerce_order' => array(
      'label' => t('Order'),
      'controller class' => 'CommerceOrderEntityController',
      'base table' => 'commerce_order',
      'revision table' => 'commerce_order_revision',
      'fieldable' => TRUE,
      'entity keys' => array(
        'id' => 'order_id',
        'bundle' => 'type',
        'revision' => 'revision_id',
      ),
      'bundle keys' => array(
      'bundle' => 'type',
      ),
      'bundles' => array(
        'commerce_order' => array(
          'label' => t('Order'),
        ),
      ),
      'load hook' => 'commerce_order_load',
      'view modes' => array(
        …
      ),
      …
    ),
  );

  return $return;
}
```

注意，`bundles` 数组仅包含一个包。如果你出于某些原因需要启用多个订单包，则必须使用 `hook_entity_info_alter()` 修改此数组。订单模块的初始版本默认使用单个包，但在数据库和 API 层面支持多个包。

如果你遇到需要定义自己可修订实体的情况，可以将订单实体作为模型。请注意，必须同时指定修订表（`revision table`）和将订单链接到其当前修订版本的修订键（`revision key`）。你的控制器类必须添加对保存修订的支持，但你可以依赖 Drupal 的默认实体控制器来将修订信息正确加载到对象中。在你的控制器类的保存方法中，应在使用字段附加 API 添加或更新字段数据之前保存修订版本，以确保其与正确的修订版本一起保存。

本章后面将包含实体及其属性的完整列表。你也可以将 `drupalcommerce.org/specification/entities` 加入书签，作为开发期间的参考。

此外，各种 Commerce 模块也利用字段 API 来连接数据对象、向受益于用户可配置显示选项的实体添加数据，以及将“添加到购物车”表单嵌入站点中的任何实体。这些字段的运行方式与 Drupal 6 中的 CCK 字段非常相似，并将在下文与核心实体一同列出。你也可以将 `drupalcommerce.org/specification/fields` 加入书签，作为站点构建期间的参考。

### 表单 API 改进

表单 API 有几项改进，本节将涵盖其中两项。

#### 通过 `#ajax` 属性实现的动态表单

Commerce 模块利用的表单 API 最显著的新特性之一是表单元素的 `#ajax` 属性。借助 `#ajax` 属性，无需编写一行 JavaScript 代码，即可创建表单元素在更改或其他用户交互时自动验证和更新的表单。

在 Commerce 模块中，该功能在几个地方被使用。在订单编辑表单上，订单项表格使用 `#ajax` 属性，允许你在不刷新页面的情况下添加和更新订单项。客户模块与地址字段模块集成，后者使用此功能提供名称和地址元素，这些元素会自动更新以反映所选国家/地区的格式和词汇。

以下代码展示了使用 `#ajax` 实现动态表单是多么容易。代码取自支付模块的表单元素，当客户选择支付方式选项时，该元素会更新结账表单。完整实现位于 `commerce_payment.checkout_pane.inc` 文件中。

```
// 添加一个单选小部件来指定支付方式。
$pane_form['payment_method'] = array(
  '#type' => 'radios',
  '#options' => $options,
  '#ajax' => array(
    'callback' => 'commerce_payment_pane_checkout_form_details_refresh',
    'wrapper' => 'payment-details',
  ),
);
```

元素指定的 `callback` 函数只需返回表单数组的一部分，该部分应被渲染到由 `wrapper` 值（对应于 HTML ID）指定的 DOM 区域中。使用此功能的表单会在 JavaScript 未启用的情况下进行测试，以确保优雅降级，这依赖于表单提交处理程序能够根据用于提交表单的按钮请求重新构建表单的能力。



#### 自动文件包含

Commerce 模块重度依赖的 Forms API 的另一项创新，是在表单数组中指定 Drupal 在重建表单时应包含哪些文件。这使得模块可以将表单放入包含文件中，当提交处理表单时自动加载这些文件，而无需仅依赖活动菜单项的文件处理器。所有 Commerce 实体表单都使用了此功能，因此它们可以在默认 UI 模块特定的 URL 上实例化，或由其他贡献模块在必要时调用。实现此功能的代码相当简洁，如下例所示，摘自 `commerce_product.forms.inc`：

```
function commerce_product_product_form($form, &$form_state, $product) {
  // 确保从缓存重建表单时加载此包含文件。
  $form_state['build_info']['files']['form'] = drupal_get_path('module', 'commerce_product')
    . '/includes/commerce_product.forms.inc';
  …
}
```

Drupal 7 Forms API 的这两项新特性对 Drupal Commerce 至关重要。第一项使项目能够拥有动态表单，极大改善客户和管理员的用户体验。例如，结账表单可配置为在单页上运行，并能在不支持 JavaScript 的设备上优雅降级。第二项，由 Commerce 模块定义的表单可嵌入任何位置，完美契合了项目将 API 与 UI 严格分离的理念。

### 贡献模块依赖

作为在 Drupal 7 上从零开始构建 Drupal Commerce 策略的一部分，项目决定充分利用其他主要贡献模块，以避免代码和工作的重复。这包括引入对广泛使用的 Views 模块、Rules 模块及其依赖项的依赖。

Views 3 驱动了几乎所有 Commerce UI 模块提供的默认用户界面。这意味着每个列表页面、购物车区块以及嵌入其他表单中的某些表格显示，均可通过 Views UI 进行配置。Commerce 模块重度依赖 Views 3 新的可插件区域处理器功能，完全通过 Views 创建更强大的显示，例如购物车区块和订单上的“付款”标签。这两种显示都在其页脚中使用自定义区域处理器，向 Views 添加链接和表单。

对 Rules 的依赖同样重要。Entity API 模块（本身是 Rules 2 的依赖项）使得将自定义实体和字段数据暴露给 Rules 变得简单。Commerce 模块与 Rules 集成，允许管理员通过统一界面配置动态定价、结账表单的某些部分以及订单工作流。最新版的 Rules 还使得将 UI 部分嵌入各个位置变得容易，因此 Commerce 模块可以在默认 UI 的适当位置放置过滤后的配置列表。

Address Field（客户模块的依赖项）也作为一个独立的贡献模块维护。该项目定义了一个字段，允许用户通过根据所选国家更新的动态表单元素集输入姓名和地址信息。其目标是实现 xNAL 标准中关于姓名和邮政地址的子集，并保持独立性，以便它能独立于核心 Commerce 模块成熟，并允许其他项目或网站使用它。

### 总结

Drupal Commerce 项目仍在成熟发展中，核心代码和贡献模块生态系统中的创新可能会在后续版本中快速变化。请务必关注项目主页（`drupalcommerce.org`）和问题追踪器（`drupal.org/project/issues/commerce`），了解此处概述的系统如何成熟，并寻找您可以贡献文档、测试、代码等的地方。当处理问题或开发扩展 Drupal Commerce 功能的贡献模块时，您通常也可以在 `irc.freenode.net` 的 `#drupalcommerce` IRC 频道中找到帮助。

