# 第 15 章 处理文件上传

**351**

`UPLOAD_ERR_PARTIAL`（值 = 3）

如果文件未完全上传，则返回值 3。这可能是由于网络错误导致上传过程中断而发生的。

`UPLOAD_ERR_NO_FILE`（值 = 4）

如果用户提交表单时没有指定要上传的文件，则返回值 4。

### 文件上传示例

现在我们已经打下了基本概念的基础，是时候考虑一些实际示例了。

#### 第一个文件上传示例

第一个示例实际实现了本章中一直引用的课堂笔记示例。为了形式化这个场景，假设一位教授邀请学生将课堂笔记发布到他的网站上，想法是每个人都可能从这种合作中有所收获。当然，功劳仍然应该归于应得之人，因此每个上传的文件都应重命名为学生的姓氏。此外，只接受 PDF 文件。

清单 15-1（`uploadmanager.php`）给出了一个示例。

**清单 15-1.** 一个简单的文件上传示例

```html
<form action="uploadmanager.php" enctype="multipart/form-data" method="post">
  Last Name:<br />
  <input type="text" name="name" value="" /><br />
  Class Notes:<br />
  <input type="file" name="classnotes" value="" /><br />
  <p><input type="submit" name="submit" value="Submit Notes" /></p>
</form>
```

```php
<?php
/* Set a few constants */
define ("FILEREPOSITORY","/home/www/htdocs/class/classnotes/");

/* Make sure that the file was POSTed. */
if (is_uploaded_file($_FILES['classnotes']['tmp_name'])) {
    
    /* Was the file a PDF? */
    if ($_FILES['classnotes']['type'] != "application/pdf") {
        echo "<p>Class notes must be uploaded in PDF format.</p>";
    } else {
        /* move uploaded file to final destination. */
        $name = $_POST['name'];
        $result = move_uploaded_file($_FILES['classnotes']['tmp_name'],
                                     FILEREPOSITORY."/$name.pdf");
        
        if ($result == 1) echo "<p>File successfully uploaded.</p>";
        else echo "<p>There was a problem uploading the file.</p>";
    } #endIF
} #endIF
?>
```

> **警告**：请记住，文件的上传和移动都是在 Web 服务器守护进程所有者的身份下进行的。如果未能为该用户分配临时上传目录和最终目的目录足够的权限，将导致文件上传过程无法正确执行。

#### 按日期列出上传的文件

教授对学生在课堂笔记项目中的参与感到高兴，决定将所有课程通信转移到线上。他当前的项目涉及提供一个接口，允许学生通过 Web 提交他们的每日家庭作业。与课堂笔记一样，家庭作业需要以 PDF 格式提交，并存放在服务器上时以学生的姓氏作为文件名。由于作业是每日提交的，教授希望有一种方法可以按日期自动组织作业提交，并确保班级里的懒虫不能在截止日期（每天 23:59:59）之后偷偷提交作业。

清单 15-2 中的脚本自动完成了所有这些工作，最大限度地减少了教授的管理负担。除了确保文件是 PDF 格式并自动分配学生指定的姓氏外，该脚本还会每天创建新文件夹，每个文件夹遵循`MM-DD-YYYY`的命名约定。

**清单 15-2.** 按日期对文件进行分类

```html
<form action="homework.php" enctype="multipart/form-data" method="post">
  Last Name:<br />
  <input type="text" name="name" value="" /><br />
  Homework:<br />
  <input type="file" name="homework" value="" /><br />
  <p><input type="submit" name="submit" value="Submit Notes" /></p>
</form>
```

```php
<?php
```