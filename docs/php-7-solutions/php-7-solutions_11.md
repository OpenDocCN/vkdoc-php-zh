# 提示

如果使用 `http://localhost/` 时出现问题，请改用 `http://127.0.0.1/`。`127.0.0.1` 是所有计算机用来指向本机的回环 IP 地址。



### 使用虚拟主机

将 PHP 文件存储在 Web 服务器文档根目录以外的另一种选择是使用虚拟主机。虚拟主机为每个站点创建唯一的地址，这也是托管公司管理共享主机的方式。MAMP PRO 通过其控制面板简化了虚拟主机的设置。EasyPHP 也提供了一个用于管理虚拟主机的插件模块。

手动设置虚拟主机需要编辑计算机的一个系统文件，在本地机器上注册主机名。你还需要告知本地测试环境中的 Web 服务器文件所在的位置。这个过程并不困难，但每次设置新的虚拟主机时都需要操作一次。

在虚拟主机中设置每个站点的优势在于，它能更精确地匹配实际网站的结构。不过，在学习 PHP 时，使用测试服务器文档根目录的子文件夹可能更方便。当你积累了 PHP 经验后，就可以进阶使用虚拟主机。在 Apache 中手动设置虚拟主机的说明请参考我网站上的以下地址：

- **Windows**: [`http://foundationphp.com/tutorials/apache_vhosts.php`](http://foundationphp.com/tutorials/apache_vhosts.php)
- **MAMP**: [`http://foundationphp.com/tutorials/vhosts_mamp.php`](http://foundationphp.com/tutorials/vhosts_mamp.php)

