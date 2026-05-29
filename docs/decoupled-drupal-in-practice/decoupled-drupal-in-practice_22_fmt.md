# 第 V 部分 与消费者的集成

与消费者的集成

在第 4 部分中，我们对新兴的解耦 Drupal 生态系统及其最常用的组件进行了简要调查，包括像 `Contenta`、`Reservoir` 和 `Headless Lightning` 这样的 API 优先发行版，以及帮助开发人员为自身实现提供起点的 SDK 和参考构建。

在这些章节中，我们将深入探讨一些最常用的、驱动基于 Drupal 应用程序的 JavaScript 技术，即 `React`、`React Native`、`Angular`、`Vue.js` 和 `Ember`。在此过程中，我们探索了所有这些技术的观点倾向范围，并构建了与 Drupal Web 服务交互的应用程序。当然，我们不可能详尽地涵盖这些项目以解释其所有概念，因为每个项目都值得为其撰写一本甚至更多的书。

我们在这些章节中审视的 JavaScript 技术涵盖了从有限的、专注于视图的库到高度固执己见的 MV*（模型-视图-任何东西）框架的整个范围。这两种类别的 JavaScript 技术都在前端开发社区中被大规模采用。尽管如此，这些工具在多个维度上存在显著差异。

例如，尽管 `React` 处于观点性较弱的端，但它也将许多关于其他工具的必要决策留给了开发人员，这可能导致具有挑战性的学习曲线。其他工具，即 `Angular` 和 `Ember`，则对它们期望开发者如何使用其功能抱有很强的观点性。例如，`Ember` 默认包含 JSON API 支持，而 `Angular` 则强制使用 `TypeScript`。与此同时，JavaScript 社区中的许多人通常认为 `React` 和 `Vue.js` 更灵活，并允许采用不同的解决方案。例如，虽然我们可以通过将其作为依赖项包含在 `React` 中来自由地利用 `Waterwheel.js`，但由于 `Ember` 有一个用于 JSON API 的默认适配器，`Waterwheel.js` 就变得多余了。

由于 `React` 是 JavaScript 社区中使用的最流行的 JavaScript 项目之一，并且在当今的解耦 Drupal 架构中得到了很好的体现，因此我们从它开始我们的旅程。之后，我们将介绍基于 Drupal 的应用程序在 `React Native`、`Angular`、`Vue.js` 和 `Ember` 中的实现。