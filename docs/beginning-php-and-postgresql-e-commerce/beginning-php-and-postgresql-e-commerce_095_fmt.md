# 工作原理：在 HatShop 中展示亚马逊商品

在本练习中，您只需使用本章第一部分学习的技术，更新 `HatShop` 以展示亚马逊的商品。这项新功能并不特别复杂，但其带来的可能性令人兴奋。

若要更改访问方式，请修改 `include/config.php` 中的以下内容：

```
// Amazon E-Commerce Service
define('AMAZON_METHOD', 'REST');
//define('AMAZON_METHOD', 'SOAP');
```

当点击“从亚马逊购买”链接时，亚马逊会将客户及其购买的商品关联到您的合作伙伴 ID（该 ID 在链接中提及）。在 `AmazonProductsList` 类的 `init` 方法中，会调用 `Amazon` 类的 `GetProducts` 方法来获取数据，从而填充产品列表。这些数据会被读取，用于构建指向检索到产品的亚马逊链接：

```
public function init()
{
    $amazon = new Amazon();
    $this->mProducts = $amazon->GetProducts();
    for ($i = 0;$i < count($this->mProducts); $i++)
        $this->mProducts[$i]['link'] =
            'http://www.amazon.com/exec/obidos/ASIN/' .
            $this->mProducts[$i]['asin'] .
            '/ref=nosim/' . AMAZON_ASSOCIATES_ID;
}
```

不过，您必须知道，亚马逊提供了许多方式，让您可以允许访客购买他们的产品。

如果您登录到合作伙伴页面，您会看到多种可以构建并集成到您网站中的链接类型。

或许最有趣且最强大的功能是，通过使用亚马逊 API，您可以从 PHP 代码中创建和管理亚马逊购物车。如果您真的希望将亚马逊整合到您的网站中，您应该仔细研究 ECS 文档并充分利用它。

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH17a.qxd 11/22/06 5:42 AM Page 569

第 17 章 ■ 连接 Web 服务

**569**

## 总结

在本章中，您学习了如何使用 `REST` 和 `SOAP` 访问亚马逊电子商务服务。在通过这些协议访问任何类型的外部功能时，您都可以使用相同的技术。

恭喜您，您刚刚完成了学习使用 PHP 和 PostgreSQL 构建电子商务网站的旅程。您已经具备了构建自己定制解决方案的知识，这些方案甚至可能比本书展示的更令人兴奋、更强大。我们希望您喜欢阅读这本书，并祝您在自己的 PHP 和 PostgreSQL 项目中好运！

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH17a.qxd 11/22/06 5:42 AM Page 570

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XAppA.qxd 11/19/06 1:45 PM Page 571

## 附录 A：安装 Apache、PHP 和 PostgreSQL

在本附录中，您将学习如何安装：

- Apache 2.2

- PHP 5.1 以及本书所需的额外模块

- PostgreSQL 8.1

这些是本附录测试过的软件版本。不过，您应该安装所有软件包的最新版本，并理解安装步骤可能会略有不同。我们将分别讨论在 Windows 和 Linux 下的安装。

您在本书中开发的 `HatShop` 应用程序也可能在旧版本的软件上运行。非常重要的是，PHP 版本应为 PHP 5.0 或更高版本；代码使用了旧版 PHP 无法识别的 OOP 语法。

由于信用卡信息等高度敏感数据必须在网络上安全传输，因此将您的应用程序托管在支持 SSL 的 Web 服务器上至关重要。此外，PHP 安装必须包含以下模块：`CURL`、`mcrypt`、`mhash`、`SOAP`、`PDO` 以及适用于 PostgreSQL 的 `PDO` 驱动程序。

## 准备您的 Windows 开发环境

在本节中，您将学习如何在您的开发机上安装 Apache 2.2、PHP 5.1 和 PostgreSQL 8.1。

在 Windows 中，安装支持 SSL 的 Apache 比安装不带 SSL 的 Apache 要稍微复杂一些。即使您没有支持 SSL 的 Apache，也可以继续学习本书；如果选择这样做，您可以跳到接下来的“安装 Apache（无 SSL）”部分。