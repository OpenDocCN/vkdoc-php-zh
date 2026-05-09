# 访问密钥 ID 无法做到的事

访问密钥 ID 无法做到的是，让你通过网站销售的亚马逊产品获得佣金。要获得收入，你需要申请一个`关联者 ID`。这个`关联者 ID`将用于你在亚马逊专区展示的“从亚马逊购买”链接，亚马逊通过它来识别你是该笔销售的来源。

`关联者 ID`甚至可用于包含亚马逊产品链接的静态网页，并且无需你同时拥有用途不同的 ECS 访问密钥 ID。

因此，在进一步操作之前，如果你想通过“亚马逊超级帽子”专区赚钱，请前往 `http://associates.amazon.com/gp/associates/apply/main.html` 获取你的`关联者 ID`。否则，如果你目前只对学习 ECS 感兴趣，可以跳过这一步。

## 使用 REST 访问亚马逊电子商务服务

访问 REST Web 服务需要请求一个格式正确的 URL。请在浏览器中尝试以下链接（别忘了将字符串 `[Your Access Key ID]` 替换为你之前获得的真实访问密钥 ID）：

```
http://webservices.amazon.com/onca/xml?Service=AWSECommerceService
&AWSAccessKeyId=[Your Access Key ID]
&Operation=ItemLookup
&IdType=ASIN
&ItemId=159059648X
```

**提示：** 请确保将整个 URL 输入为单行；这里将其拆分为独立元素是为了便于阅读。

你的浏览器将显示一个包含你正在阅读的书籍信息的 XML 结构。图 17-3 展示了 Firefox 中显示的 XML 结构，它清晰地呈现了 XML 文档树。

---

**图 17-3.** *Web 服务请求的 XML 响应*

很酷吧？你刚刚见证了 REST 的实际应用。亚马逊数据库中的每个产品都有一个唯一的标识符，称为 ASIN（亚马逊标准商品编号）。对于图书，ASIN 就是图书的 ISBN（本书的 ASIN 为 `159059648X`）。

你刚刚发出的 Web 服务请求向 ECS 传达了以下信息：我有一个访问密钥 ID（`AWSAccessKeyId=[Your Access Key ID]`），我想执行一项商品查询操作（`&Operation=ItemLookup`），以了解更多关于 ASIN 为 `159059648X` 的产品信息（`&IdType=ASIN&ItemId=159059648X`）。

在这个例子中，你并没有获得关于这本书的详细信息——没有价格或库存信息，也没有封面图片或客户评论的链接。ECS 4.0 引入了更精细的数据控制方式，即使用响应组（响应组是一组关于产品的信息集合）。

**注意：** 在撰写本文时，ECS 提供了超过 35 个可能的响应组列表。在本书中，我们只会解释 HatShop 所用响应组的作用；完整的列表请查阅 ECS 文档。

---

那么，让我们通过使用响应组来请求更多数据。在你之前组合的链接末尾，添加以下字符串以获取关于本书的更具体信息：

`&ResponseGroup=Request,SalesRank,Small,Images,OfferSummary`

完整的链接应如下所示：

```
http://webservices.amazon.com/onca/xml?Service=AWSECommerceService
&AWSAccessKeyId=[Your Access Key ID]
&Operation=ItemLookup
&IdType=ASIN
&ItemId=159059648X
```



`&ResponseGroup=Request,SalesRank,Small,Images,OfferSummary` 来自亚马逊网站（Amazon.com）的新 XML 响应包含了关于该商品更详细的信息，如图 17-4 所示。

**图 17-4.** *Web 服务请求的 XML 响应*  
我们刚刚混合使用了五个响应组：`Request`、`SalesRank`、`Small`、`Images` 和 `OfferSummary`。要了解更多关于响应组的信息，请访问 `http://developer.amazonwebservices.com/connect/kbcategory.jspa?categoryID=5`，并点击“最新技术文档”（Latest Tech Docs）按钮。或者，你也可以点击“技术文档”（Technical Documentation）链接，然后点击最新版本文档的链接。你可以下载 PDF 格式的文档，也可以在线阅读：`http://docs.amazonwebservices.com/AWSEcommerceService/2006-09-13/`。

在 ECS 文档中，可以在“API 参考”（API Reference）的“响应组”（Response Groups）部分找到响应组的详细信息。以下是前一个示例中使用的五个响应组的描述：

- `Request` 响应组是每种操作中的默认响应组，它会返回你用于发起请求的名称-值对列表。
- `SalesRank` 响应组返回有关商品当前在亚马逊网站（Amazon.com）上销售排名的数据。
- `Small` 响应组返回响应中包含的商品的一般数据（如 ASIN、商品名称、URL 等）。这是 `ItemLookup` 操作（如本示例所示）的默认响应组。
- `Images` 响应组提供响应中每个商品的三张图片（小图、中图和大图）的地址。
- `OfferSummary` 响应组返回响应中每个商品的价格信息。

接下来，我们继续学习如何从 PHP 发起 REST 请求。为了充实未来的 Amazon Super Hats 部门，你将在亚马逊网站（Amazon.com）的服装（Apparel）部门中搜索关键词“super hats”。一种简单的方法是使用 PHP 的 `file_get_contents` 函数，如下面的脚本所示。

要测试使用 REST 访问 Web 服务，请在 `hatshop` 目录中创建一个名为 `test_rest.php` 的新文件，并在其中写入以下代码：

```php
<?php
// 告知浏览器即将接收一个 XML 文档。
header('Content-type: text/xml');

/* 请务必在下面这行代码中，将字符串 '[Your Access Key ID]' 替换为你的访问密钥 ID */
$url = 'http://webservices.amazon.com/onca/xml?Service=AWSECommerceService' .
    '&AWSAccessKeyId=[Your Access Key ID]' .
    '&Operation=ItemSearch' .
    '&Keywords=super+hats' .
    '&SearchIndex=Apparel' .
    '&ResponseGroup=Request,Medium';

echo file_get_contents($url);
?>
```

**注意：** 某些 PHP 安装环境和网站托管提供商可能默认不允许运行此代码。在这种情况下，你可以在 `php.ini` 中更改此设置：

```
allow_url_fopen = On
```

或者，你可以将以下行添加到 `include/config.php` 中。第二种方案是首选，因为它只影响你的应用程序，并且在需要将应用程序迁移到其他服务器时，该设置仍然有效。

```
ini_set('allow_url_fopen', 'On');
```

加载 `http://localhost/hatshop/test_rest.php` 将会显示关于亚马逊超级帽子（Super Hats）的 XML 数据（见图 17-5）。

**图 17-5.** *来自亚马逊的超级帽子*

为了练习并构建更多 XML 链接，只需研究“API 参考”（API Reference）中的示例即可。



