# 谨慎选择字体

有些设计师喜欢在站点的每个区域都使用独特字体（非网页安全字体）。我们建议将其限制在标题和特色版块，并避免使用 CSS 背景图片作为标题，尤其是对于全球性站点。这不仅关乎可用性，也影响页面加载时间。由于 Drupal 依赖动态更改内容的能力，使用 CSS 背景图片作为标题会使站点维护者难以进行必要的修改。此外，这些图片可能会导致屏幕阅读器或视障用户出现可访问性问题。

如果你确实想使用非标准网页字体，请考虑使用已获得 `@font-face` 许可的字体。`@font-face` 是 CSS3 规范，允许浏览器以网页安全格式渲染字体。请查看 `www.css3.info/preview/web-fonts-with-font-face` 获取关于使用 `@font-face` 的更多信息。像 Typekit (`typekit.com`) 和 FontDeck (`fontdeck.com`) 这样的字体托管服务可以让你轻松地在 Drupal 站点中引入支持 `@font-face` 的字体，而 FontSquirrel (`fontsquirrel.com`) 等站点则提供免费的支持 `@font-face` 的字体，以及为你生成 `@font-face` 样式的功能。

