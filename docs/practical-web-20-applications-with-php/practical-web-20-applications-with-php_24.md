# `ls`

`_documentation.html` `fckeditor.afp` `fckeditor.php` `fckstyles.xml`

`_samples/` `fckeditor.asp` `fckeditor.pl` `fcktemplates.xml` `_upgrade.html` `fckeditor.cfc` `fckeditor.py` `htaccess.txt`

`_whatsnew.html` `fckeditor.cfm` `fckeditor_php4.php` `license.txt`

`editor/` `fckeditor.js` `fckeditor_php5.php`

`fckconfig.js` `fckeditor.lasso` `fckpackager.xml`

[www.it-ebooks.info](http://www.it-ebooks.info/)

9063Ch08CMP2 11/11/07 12:35 PM Page 293

## 第 8 章 ■ 扩展博客管理器

我通常喜欢做的第一件事是检查并清理发行版中不必要的文件。目前我将保留所有这些项目，但你可以考虑删除以下内容：

*   其他语言的加载器类（主目录中的`fckeditor.*`文件，除了我们稍后将使用的`fckeditor_php5.php`文件）。
*   未使用的文件浏览器和上传连接器。这些文件可以在`./htdocs/js/fckeditor/editor/filemanager`目录中找到。

#### 配置 FCKeditor

接下来，我们必须配置 FCKeditor 的工作方式。我们通过修改主目录中的`fckconfig.js`来实现。大部分设置我们无需改动，但需要自定义工具栏，然后禁用默认启用的连接器。

首先，我们将定义一个新工具栏，其中只包含我们在第 7 章中定义的标签列表对应的按钮。这些标签是`<a>`、`<img>`、`<b>`、`<strong>`、`<em>`、`<i>`、`<ul>`、`<li>`、`<ol>`、`<p>`和`<br>`。

在`fckconfig.js`的第 94 行，定义了一个名为`Default`的工具栏，它包含大量按钮，其后直接跟有一个名为`Basic`的简化工具栏。我们将在该文件中保留这两个工具栏，并定义一个名为`phpweb20`的新工具栏，它是这些工具栏的组合。保留它们的主要原因是可以将其作为可添加的其他按钮的参考。

清单 8-23 展示了我们用于创建新工具栏的 JavaScript 数组。可以将其放置在`fckconfig.js`中其他工具栏之后。请注意，`'-'`元素会在工具栏中渲染一个分隔符。

**清单 8-23.** *自定义 FCKeditor 工具栏 (fckconfig.js)*

```
FCKConfig.ToolbarSets["phpweb20"] = [

['Bold','Italic','-','OrderedList','UnorderedList','-',

'Link','Unlink','-','Image']

];
```

> **注意** 严格来说，清单 8-23 实际上定义的是一个*工具栏集*，而非工具栏。换句话说，一个或多个工具栏组成一个工具栏集。此代码创建了一个数组的数组，其中内部数组才是实际的工具栏。

我们在这个配置文件中还需要做的唯一改动是禁用文件管理器和上传连接器，因为我们不允许用户上传文件。禁用它们会从用户界面中移除相应的选项。

清单 8-24 展示了`fckconfig.js`的新行，所有行都将列出的值设置为`false`。你可以在`fckconfig.js`文件底部找到每个变量被定义为`true`的位置，并相应地进行更新。

**清单 8-24.** *禁用文件浏览器和上传连接器 (fckconfig.js)*

```
FCKConfig.LinkBrowser = false;
FCKConfig.ImageBrowser = false;
FCKConfig.FlashBrowser = false;
FCKConfig.LinkUpload = false;
FCKConfig.ImageUpload = false;
FCKConfig.FlashUpload = false;
```

#### 在博客编辑页面加载 FCKeditor

最后，我们需要在博客文章的编辑表单中加载编辑器。首先，我们将编写一个 Smarty 插件，用于输出要加载的 HTML 代码。FCKeditor 附带了一个 PHP 类，用于简化 HTML 的生成。

FCKeditor 类位于 FCKeditor 主目录（`./htdocs/js/fckeditor`）的`fckeditor_php5.php`文件中。为了保持我们自己的代码整洁，我们将把这个类复制到应用程序的包含目录中。此外，为了与我们的应用程序文件命名保持一致，我们将文件重命名为`FCKeditor.php`。这也意味着它可以通过`Zend_Loader`自动加载。

```
# cd /var/www/phpweb20/htdocs/js/fckeditor
```



