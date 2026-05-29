# 脚注

## 26. 解耦式 Drupal 的未来

正如我们在这些章节中所看到的，解耦式 Drupal 在许多方面都有可能决定 Drupal 未来多年的发展轨迹。它能够为各种消费者提供 Drupal 内容，而无需考虑这些消费者是如何构建的，这既是解耦式 Drupal 的关键优势之一，也是 API 优先架构的普遍优势。Drupal 并非处于这一范式转变的孤例，许多其他长期存在的软件项目（如 WordPress）也在经历着自身的重大演进。

Drupal 正处于其历史中一个独特的十字路口。迄今为止，组织选择 Drupal 大多是基于其构建网站的能力以及提供丰富功能编辑体验的能力。如今，组织对 Drupal 的应用更进一步，用其内容服务于整个数字生态系统，将所有数据集中到一个 CMS 中。此外，Drupal 的用户体验正在经历全面改革，不仅涉及设计和用户界面，还包括前端开发体验，即对 JavaScript 进行现代化改造。

在本章中，我们将简要介绍一些正在进行的工作，旨在持续为 Drupal 做好准备迎接充满希望的未来，以及一些将对 Drupal 作为 CMS 的中长期未来产生直接影响的问题——其视野已然远远超出简单的网站范畴。

### 管理界面与 JavaScript 现代化计划

2017 年 9 月，在维也纳举行的 DrupalCon 上，一群 Drupal 核心贡献者和 Drupal 社区的 JavaScript 维护者一致同意，在 Drupal 核心的管理界面中采用 React 库进行实验。¹¹⁰ 这一漫长过程始于作者在 2016 年初提议采用 JavaScript 框架，¹¹¹ React 的采用为 Drupal 管理界面的运作方式带来了根本性变革。其他被考虑的备选方案包括 Angular、Ember、Vue.js、Elm 等框架，社区中仍有人支持将 Vue.js 作为备选，原因在于它被 Laravel 社区采用且具有渐进式采用特性（参见第 20 章）。

管理界面与 JavaScript 现代化计划是由 Drupal 社区的用户体验专家、设计师与 JavaScript 维护者共同发起的一项联合行动，旨在重新构想 Drupal 的内部用户体验，使其符合那些由单页应用和无缝交互构成的其他编辑体验的期望。

自 Drupal 8 发布以来，Drupal 的管理体验基本保持不变。一个相关的问题是：这种体验是否已经过时到足以威胁 Drupal 作为 CMS 的主导地位？

管理界面与 JavaScript 现代化计划由 Angie Byron (webchick)、Cristina Chumillas (ckrina)、Matt Grill (drpal) 和 Sally Young (justafish) 领导，根据 Drupal.org 计划页面所述，其目标如下：¹¹²

1.  为 Drupal 的编辑和管理界面创建一套新的“设计系统”，并逐步实施。

2.  创建一个解耦式的单页 React 应用，用于管理 Drupal 后台。

3.  现代化底层 JavaScript 代码，并增强 Drupal 的 API，以更好地支持所有类型的解耦应用。