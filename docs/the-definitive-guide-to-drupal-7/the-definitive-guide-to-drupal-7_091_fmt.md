# 引入 WAI-ARIA

无障碍富互联网应用（WAI-ARIA）标准的部分元素已被引入 Drupal 7 核心。WAI-ARIA 目前仍为草案文档，因此其应用范围有限。它被添加在无法以其他方式向屏幕阅读器传达重要信息的地方。WAI-ARIA 可以提供更多工具，用于向你的网站添加语义信息。

WAI-ARIA 地标角色定义了特定 HTML 区块的关键词，以向屏幕阅读器传达更多含义。地标允许网页开发者划分网页，使其内容更易于浏览。Juice Studio 的 Firefox 插件支持识别你网站中定义的地标角色（`juicystudio.com/article/examining-wai-aria-document-andmark-roles.php`）。

Drupal 7 和 jQuery 提供了许多交互元素。使用的动态元素越多，就越需要为 ARIA 的实时区域添加支持，以便其信息能够有效传达给屏幕阅读器。通过将交互元素的重要性定义为“礼貌”、“强烈”或“粗鲁”，可以指示屏幕阅读器是打断当前正在朗读的文本，还是等待其朗读完毕。