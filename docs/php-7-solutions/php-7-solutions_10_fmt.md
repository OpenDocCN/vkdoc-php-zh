# 提示

如果端口 80 已被其他程序占用，Apache 将无法重启。如果你找不到是什么程序占用了端口 80，请打开 MAMP 偏好设置面板，点击“将 MAMP 端口设置为默认值”。

1.  当两个指示灯再次变为绿色时，MAMP 欢迎页面会重新加载到你的浏览器中。此时，URL 中 `localhost` 之后不应再出现冒号加数字的格式，因为 Apache 现在正在默认端口上监听。

## PHP 文件应放置在何处（Windows 和 Mac）

你需要将文件创建在 Web 服务器可以处理它们的位置。通常，这意味着文件应位于服务器的文档根目录或文档根目录的子文件夹中。最常见配置下文档根目录的默认位置如下：

- **XAMPP**: `C:\xampp\htdocs`
- **WampServer**: `C:\wamp\www`
- **EasyPHP**: `C:\EasyPHP\www`
- **IIS**: `C:\inetpub\wwwroot`
- **MAMP**: `/Applications/MAMP/htdocs`

要查看一个 PHP 页面，你需要使用 URL 在浏览器中加载它。在你的本地测试环境中，Web 服务器文档根目录的 URL 是 `http://localhost/`。

### 警告

如果你需要将 MAMP 重置回默认端口，则必须使用 `http://localhost:8888` 而不是 `http://localhost`。

如果你将本书的示例文件存放在文档根目录下名为 `phpsols-4e` 的子文件夹中，那么 URL 将是 `http://localhost/phpsols-4e/` 后跟文件夹名称（如果有）和文件名。