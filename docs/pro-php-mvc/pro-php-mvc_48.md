# 由于我们需要在用户注册和设置更新的地方添加上传功能，因此应该添加一个核心函数来执行上传操作。我们将把这个函数添加到`Users`模型中，如代码清单 18-4 所示。

#### 代码清单 18-4. `_upload()`方法

```php
use Shared\Controller as Controller;

class Users extends Controller
{
    protected function _upload($name, $user)
    {
        if (isset($_FILES[$name]))
        {
            $file = $_FILES[$name];
            $path = APP_PATH."/public/uploads/";
            $time = time();
            $extension = pathinfo($file["name"], PATHINFO_EXTENSION);
            $filename = "{$user}-{$time}.{$extension}";

            if (move_uploaded_file($file["tmp_name"], $path.$filename))
            {
                $meta = getimagesize($path.$filename);
                if ($meta)
                {
                    $width = $meta[0];
                    $height = $meta[1];

                    $file = new File(array(
                        "name" => $filename,
                        "mime" => $file["type"],
                        "size" => $file["size"],
                        "width" => $width,
                        "height" => $height,
                        "user" => $user
                    ));

                    $file->save();
                }
            }
        }
    }
}
```

关于这个方法，有几个重要事项需要注意。首先，文件上传可能只在两个方面有所不同，具体来说是表单输入的名称和上传文件的用户 ID。其次需要注意的是，该方法在`public/uploads`文件夹内为新文件定义了一个路径。这将允许文件通过 URL 直接访问。

与我们的示例代码一样，该方法检查是否有文件被上传。如果有，该方法会尝试将文件移动到永久位置。

**注意：** 当文件无法移动到永久存储时，这可能是上传不安全或无效的迹象。此时正好可以引发异常，让开发者意识到出现了问题。

文件移动到永久存储后，会创建一个新的`File`实例，并将一行记录保存到数据库中。我们还会确定上传文件的宽度/高度。既然我们有了一个可重用的上传方法，现在是时候将其集成到`Users`控制器的`register()`和`settings()`操作中了。

这些方法应类似于代码清单 18-5 和 18-6 所示。

#### 代码清单 18-5. `register()`操作

```php
use Framework\RequestMethods as RequestMethods;
use Shared\Controller as Controller;

class Users extends Controller
{
    public function register()
    {
        if (RequestMethods::post("save"))
        {
            $user = new User(array(
                "first" => RequestMethods::post("first"),
                "last" => RequestMethods::post("last"),
                "email" => RequestMethods::post("email"),
                "password" => RequestMethods::post("password")
            ));

            if ($user->validate())
            {
                $user->save();
                $this->_upload("photo", $user->id);
                $this->actionView->set("success", true);
            }

            $this->actionView->set("errors", $user->errors);
        }
    }
}
```

#### 代码清单 18-6. `settings()`操作

```php
use Framework\RequestMethods as RequestMethods;
use Shared\Controller as Controller;

class Users extends Controller
{
    /**
     * @before _secure
     */
    public function settings()
    {
        $errors = array();

        if (RequestMethods::post("save"))
        {
            $this->user->first = RequestMethods::post("first");
            $this->user->last = RequestMethods::post("last");
            $this->user->email = RequestMethods::post("email");

            if (RequestMethods::post("password"))
            {
                $this->user->password = RequestMethods::post("password");
            }

            if ($this->user->validate())
            {
                $this->user->save();
                $this->_upload("photo", $this->user->id);
                $this->actionView->set("success", true);
            }

            $errors = $this->user->errors;
        }

        $this->actionView->set("errors", $errors);
    }
}
```

集成我们可重用的`_upload()`方法相当简单。在每种情况下，我们只需提供正确的表单输入名称和用户 ID 即可执行文件上传。我们还需要将表单输入添加到`register.html`和`settings.html`视图中，如代码清单 18-7 所示。

#### 代码清单 18-7. 文件输入

```html
<li>
    <label>
        照片：
        <input type="file" name="photo" />
    </label>
</li>
```



- 确认已完成所有步骤：创建了 `uploads` 文件夹（并设置了上传所需的正确文件权限），且已提交注册或设置表单。此时，你应该能看到该文件夹中开始生成文件。同时，你需要在 `settings.html` 和 `register.html` 表单中添加 `enctype = "multipart/form-data"` 属性。

> **注意**：如果要为文件上传机制添加验证，你需要在文件输入控件下方添加一些 `echo` 逻辑，以显示验证错误信息。

**稍加展示**

既然文件上传功能已可正常工作，接下来应将上传的文件整合到个人资料页面和搜索页面中。首先，我们需要在 `User` 模型上创建一个新方法，用于返回最新的 `File` 行。请参见清单 18-8，了解具体实现方式。

***清单 18-8.*** `getFile()` 方法

```php
use Shared\Model as Model;

class User extends Model
{
    public function getFile()
    {
        return File::first(array(
            "user = ?" => $this->id,
            "live = ?" => true,
            "deleted = ?" => false
        ), array("*"), "id", "DESC");
    }
}
```

该方法仅从数据库中返回与当前用户行关联的最新文件。接下来，我们需要更新 `search.html` 模板以包含照片，如清单 18-9 所示。

***清单 18-9.*** `search.html` 模板（节选）

```html
{foreach $_user in $users}
{script $file = $_user->file}
<tr>
    <td>{if $file} <img src="/uploads/{echo $file->name}" />{/if}</td>
    <td>{echo $_user->first} {echo $_user->last}</td>
    <td>
        {if $_user->isFriend($user->id)}
            <a href="/unfriend/{echo $_user->id}.html">unfriend</a>
        {/if}
        {else}
            <a href="/friend/{echo $_user->id}.html">friend</a>
        {/else}
    </td>
</tr>
{/foreach}
```

由于 `getFile()` 方法符合 `Base` 类中 `__get()` 方法的模式，我们可以直接通过 `$_user->file` 引用它。如果找到了文件，则将文件名回显到 `img` 元素的 `src` 属性中，从而在搜索结果列表中渲染该图片。最后一步是修改 `profile.html` 视图，如清单 18-10 所示。

***清单 18-10.*** `profile.html` 模板（节选）

```html
{script $file = $user->file}
{if $file} <img src="/uploads/{echo $file->name}" />{/if}
<h1>{echo $user->first} {echo $user->last}</h1>
```

这是一个个人资料页面！

至此，我们已完成允许用户上传文件，并让这些文件在搜索页面和个人资料页面上显示的所有步骤。

**问题**

1. 为什么有必要在数据库中保存对上传文件的引用？
2. 我们将文件上传到了 `public/uploads` 文件夹，并且没有检查它们是否为有效的图片。是什么阻止了用户上传恶意文件到服务器？
3. 确定并存储文件的宽度和高度的目的是什么？
4. 如何修改此功能以支持上传到像 Amazon CloudFront 或 Rackspace Cloud Files 这样的云服务？

**答案**

1. 这并不是绝对必要的，但强烈建议这样做。任何 PHP 应用中最大的瓶颈之一就是文件的读写操作。如果我们不在数据库中保存这些文件引用，就必须遍历 `uploads` 文件夹中的所有文件，匹配那些文件名中包含用户 ID 的文件。这种方法效率太低，不实用。而通过数据库行，我们可以简单地查询与用户关联的所有行，并通过引用返回对应的文件。
2. 我们没有进行验证，因此无法阻止此类事情发生。
3. 在保护应用程序安全时，你应该采取一系列步骤——从拒绝执行 `uploads` 目录中的任何文件，到验证允许用户上传的文件的 MIME 类型、大小和文件名。这些过程应在文件被移动到永久存储之前完成，并且不会妨碍其他步骤的执行。



### 3\. 我们谈论了很多关于文件的内容，但实际上指的都是照片。

其核心思想是用户可以上传多张照片，并且这些照片会在数据库中被引用。`Width` 和 `height` 是用于确定照片如何渲染和/或修改的有用属性。我们在此并未使用它们，但它们可以应用于多种实用场景，例如生成缩略图或在幻灯片中渲染。

### 4\. 我们首先要检查文件是否已上传。

在执行任何必要的文件验证后，我们会将文件上传到云服务器，而不是将其移动到（我们自己的服务器上的）永久存储位置。确定元数据并将引用保存到数据库的步骤仍然适用，但在获取信息的具体方式上可能会有细微差别。

### 练习

1.  我们的代码在文件验证方面相当简略。请尝试添加一些检查，例如文件大小、MIME 类型和文件名，以确保上传的文件确实是有效的图像。

2.  利用存储的 `width` 和 `height` 属性，尝试为上传的照片创建缩略图，这样当上传大尺寸照片时，搜索页面和个人资料页面就不会出错。一个可能有帮助的 PHP 函数是 `imagecopyresized()`。

3.  我们简单讨论了一下使用基于云的存储服务器。如果你特别有冒险精神，可以尝试连接并使用这些服务之一来存储上传的文件。

> **注意** 你可以在 [www.php.net/manual/en/function.imagecopyresized.php](http://www.php.net/manual/en/function.imagecopyresized.php) 上阅读更多关于 `imagecopyresized()` 方法的信息。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 19 章：扩展

流行框架的最大区别之一在于它们能多么轻松地通过第三方库进行扩展。到目前为止，我们开发的所有类要么属于框架，要么属于应用程序。

### 目标

-   我们需要了解如何创建自己的库。
-   我们需要了解如何使用其他开发者编写的库。
-   我们需要理解什么是观察者设计模式，以及它为何与我们的应用程序相关。
-   我们需要在我们的应用程序中构建一种监听和触发事件的方法。

### Foxy

我们要介绍的第一种库是我们自己创建的那一类。虽然这些库严格来说并非第三方，但我们有可能在多个项目中重用它们。它们的“应用”属性不够强，我们不想将其与其余应用程序代码捆绑在一起；但它们的“框架”属性也不够强，我们不想将其添加到框架代码中。

几年前，我创建了一个库，其明确目的是确定用户的浏览器，并提供针对性的样式表，以便在 CSS 中使用自定义字体。这类似于 Google 的 Web Fonts（[www.google.com/webfonts](http://www.google.com/webfonts)）；事实上，我认为那正是最初启发我编写这个库的原因。

我决定将这个库命名为 `Foxy`（是 *font* 和 *proxy* 两个词的组合词）。这个库将启发我们构建自己的字体代理库，该库会更简单一些，但本质上是相同的。

### 自定义 CSS 字体

CSS 虽然不是本书的重点，但却是现代网络不可或缺的一部分。如果你不熟悉，可以将其视为 HTML 页面中元素的着装规范。样式声明（即关于元素某些方面外观的指令）通过 CSS 选择器定位到特定元素。CSS 选择器是结构化元素标记的字符串表示形式。

现代浏览器对自定义字体的支持相当不错，而且这个概念也绝非新鲜事物。这个库也有一些很好的替代方案，它们各自在其领域内表现出色。我们构建这个库并非因为我们能做得更好，而是为了获得如何扩展框架的基本理解。

[www.it-ebooks.info](http://www.it-ebooks.info/)



