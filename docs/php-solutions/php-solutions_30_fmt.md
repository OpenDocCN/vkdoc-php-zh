# PHP 解决方案 7-4：创建通用文件选择器

前面的 PHP 解决方案依赖于对正则表达式的理解。将其调整为适用于其他文件扩展名并不困难，但您需要小心，以免意外删除关键字符。除非正则表达式是您的专长，否则将代码包装在一个函数中可能更简单，该函数可用于检查特定文件夹并创建特定类型文件名的数组。例如，您可能想创建一个 PDF 文档文件名的数组，或者一个同时包含 PDF 和 Word 文档的数组。操作方法如下。

在 `filesystem` 文件夹中创建一个名为 `buildlist.php` 的新文件。该文件将只包含 PHP 代码，因此请删除编辑程序插入的任何 HTML。将以下代码添加到文件中：

```php
function buildFileList($dir, $extensions) {
    if (!is_dir($dir) || !is_readable($dir)) {
        return false;
    } else {
        if (is_array($extensions)) {
            $extensions = implode('|', $extensions);
        }
    }
}
```

这里定义了一个名为 `buildFileList()` 的函数，它接收两个参数：

*   `$dir`：您希望从中获取文件名列表的文件夹路径。

*   `$extensions`：可以是一个包含单个文件扩展名的字符串，也可以是一个文件扩展名数组。为保持代码简洁，文件扩展名不应包含前导句点。

该函数首先检查 `$dir` 是否是一个文件夹并且是否可读。如果不是，函数返回 `false`，并且不再执行后续代码。

如果 `$dir` 没问题，则执行 `else` 代码块。它同样以一个条件语句开始，检查 `$extensions` 是否是一个数组。如果是，则将数组传递给 `implode()`，该函数将数组元素用竖线（`|`）连接起来。竖线在正则表达式中用于表示“或”关系。假设以下数组作为第二个参数传递给函数：

`['jpg', 'png', 'gif']`

条件语句会将其转换为 `jpg|png|gif`。因此，这将查找 `jpg`、`png` 或 `gif`。然而，如果参数是一个字符串，则保持不变。

现在您可以构建正则表达式搜索模式，并将两个参数传递给 `FilesystemIterator` 和 `RegexIterator`，如下所示：

```php
function buildFileList($dir, $extensions) {
    if (!is_dir($dir) || !is_readable($dir)) {
        return false;
    } else {
        if (is_array($extensions)) {
            $extensions = implode('|', $extensions);
        }
        $pattern = "/\.(?:{$extensions})$/i";
        $folder = new FilesystemIterator($dir);
        $files = new RegexIterator($folder, $pattern);
    }
}
```

正则表达式模式是通过使用双引号字符串构建的，并将 `$extensions` 包裹在花括号中，以确保 PHP 引擎能正确解析它。复制代码时请小心。它确实不太容易阅读。

代码的最后一部分提取文件名以构建一个数组，该数组经过排序后被返回。完成的函数定义如下所示：

```php
function buildFileList($dir, $extensions) {
    if (!is_dir($dir) || !is_readable($dir)) {
        return false;
    } else {
        if (is_array($extensions)) {
            $extensions = implode('|', $extensions);
        }
        $pattern = "/\.(?:{$extensions})$/i";
        $folder = new FilesystemIterator($dir);
        $files = new RegexIterator($folder, $pattern);
        $filenames = [];
        foreach ($files as $file) {
            $filenames[] = $file->getFilename();
        }
        natcasesort($filenames);
        return $filenames;
    }
}
```

这段代码初始化了一个数组，并使用 `foreach` 循环通过 `getFilename()` 方法将文件名赋值给该数组。最后，将数组传递给 `natcasesort()`，该函数以自然、不区分大小写的顺序对其进行排序。所谓“自然”排序，是指包含数字的字符串会按照人类习惯的方式排序。例如，计算机通常会将 `img12.jpg` 排在 `img2.jpg` 之前，因为 12 中的 1 小于 2。而使用 `natcasesort()` 则会使 `img2.jpg` 排在 `img12.jpg` 之前。

要使用该函数，将文件夹路径和您想要查找的文件的扩展名作为参数传入。例如，您可以像这样从文件夹中获取所有 Word 和 PDF 文档：

```php
$docs = buildFileList('文件夹名称', ['doc', 'docx', 'pdf']);
```

`buildFileList()` 函数的代码位于 `ch07` 文件夹的 `buildlist.php` 中。