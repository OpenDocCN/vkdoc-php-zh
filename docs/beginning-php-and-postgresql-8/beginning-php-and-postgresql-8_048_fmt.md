# 文件上传处理

进入`"/www/htdocs/classnotes/" . $_FILES['classnotes']['name']`，当然，您可以在移动文件时将其重命名为任何您想要的名称。然而，正确引用文件在第一个（源）参数中的临时名称非常重要。

## 上传错误信息

像任何涉及用户交互的应用程序组件一样，您需要一种评估结果（成功或失败）的方法。您如何确切地知道文件上传过程成功了？如果在上传过程中出现错误，您如何知道是什么原因导致的？幸好，`$_FILES['userfile']['error']`中提供了足够的信息来确定结果，并且在出现错误时，还能了解错误原因。

`UPLOAD_ERR_OK`（值 = 0）

如果上传成功，则返回值 0。

`UPLOAD_ERR_INI_SIZE`（值 = 1）

如果尝试上传的文件大小超出了`upload_max_filesize`指令指定的值，则返回值 1。

`UPLOAD_ERR_FORM_SIZE`（值 = 2）

如果尝试上传的文件大小超出了`MAX_FILE_SIZE`指令（该指令可以嵌入到 HTML 表单中）指定的值，则返回值 2。

> **注意**：因为`MAX_FILE_SIZE`指令嵌入在 HTML 表单中，很容易被一个富于进取的攻击者修改。因此，始终使用 PHP 的服务器端设置（`upload_max_filesize`、`post_max_filesize`）来确保不会超过这些预定的绝对限制。

[www.it-ebooks.info](http://www.it-ebooks.info/)