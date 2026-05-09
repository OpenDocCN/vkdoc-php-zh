# 第二部分 解耦 Drupal

解耦 Drupal

在第一部分中，我们从概念和历史的角度审视了解耦的 Drupal，追溯了其在服务端和客户端的发展轨迹，并分析了常见的解耦 Drupal 方法以及选择特定范式的动机。在这些章节中，我们将深入探讨 Drupal 核心如何为消费者提供 Web 服务，Drupal 的贡献生态系统如何围绕核心功能培育出一个庞大的生态系统，以及如何对针对 Drupal Web 服务的请求进行身份验证。

在此过程中，我们将列举大量技术，包括 Drupal 核心中的模块（`Serialization`、`RESTful Web Services`、`HAL`）、Drupal 贡献模块（`JSON API`、`RELAXed Web Services`、`GraphQL`、`REST UI`）以及身份验证方法（`OAuth 2.0`、`JSON Web Tokens`）。

从开箱即用的 Web 服务角度来看，Drupal 8 核心提供了多种强大的功能来处理消费者所需的 API 响应的编码和序列化。在 Drupal 8 核心中，启用四个关键模块后，可以共同促成开箱即用地提供 RESTful API：`HAL`、`Serialization`、`REST` 和 `Basic Authentication`。其中一些模块为同样对 Drupal 有用的重要贡献模块（例如 `JSON API`、`RELAXed Web Services` 和 `GraphQL`）提供了基础支撑。例如，虽然 `JSON API` 贡献模块依赖于 `Serialization`，但它并不依赖于 `REST`。

此外，`RESTful Web Services` 模块与 Drupal 的 `Entity Access` 系统紧密集成，以提供对资源的细粒度权限访问。最后，随着最近的发展，例如对 `CORS` 的支持以及未来将 `JSON API` 纳入 Drupal 核心，直接使用 Drupal 来提供 Web 服务的优势变得更加引人注目。

在审视这些模块并将 Drupal 8 设置为 Web 服务提供商之后，我们将配置 Drupal 8 使其成为一个有效的 Web 服务提供商和后端，然后再转向 Web 服务模块的贡献生态系统。许多开发者可能会发现，Drupal 的贡献生态系统在提供 Web 服务、用户界面以及增强现有 Web 服务的附加功能方面，提供了更灵活、可扩展性更强的解决方案。

例如，`JSON API`、`CouchDB` 和 `GraphQL` 都是被广泛理解的标准，由于它们强调积极的开发者体验，因此得到了广泛采用。就 Drupal 而言，`REST UI` 模块为 `REST` 资源配置提供了更友好的用户体验。

这个模块和其他有用的贡献模块为繁琐的配置任务提供了便捷的用户界面。

最后，在任何解耦的 Drupal 架构中，最关键的要素之一是身份验证，以确保数据的私密性和安全性。虽然 Drupal 在核心中提供了几种内置机制，我们可以在解耦的 Drupal 架构中使用，即 `Basic Authentication` 和基于 cookie 的身份验证，但这些机制的安全性远不如贡献的替代方案。在贡献的领域，`Simple OAuth`（Drupal 的 `OAuth 2.0` 实现）和 `JSON Web Tokens` 模块实现了更现代的标准。

由于 Drupal 8 核心是那些尝试将 Drupal 作为解耦后端的人最常见的切入点，我们将从这里开始。

