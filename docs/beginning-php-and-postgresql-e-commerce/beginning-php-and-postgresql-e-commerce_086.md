# 第 14 章 ■ 实现订单管道：第二部分

```php
$processor->CreateAudit('发货邮件已发送给客户。', 20702);

// 更新订单状态
$processor->UpdateOrderStatus(8);

// 审计
$processor->CreateAudit('PsFinalNotification 处理完成。', 20701);
```

它使用一个熟悉的 `GetMailBody` 方法来构建邮件正文：

```php
private function GetMailBody()
{
    $body = '您的订单现已发货！' .
            '已发货的商品包括：';
    $body .= "\n\n";
    $body .= $this->_mProcessor->GetOrderAsString(false);
    $body .= "\n\n";
    $body .= '您的订单已寄送至：';
    $body .= "\n\n";
    $body .= $this->_mProcessor->GetCustomerAddressAsString();
    $body .= "\n\n";
    $body .= '订单参考编号：';
    $body .= $this->_mProcessor->mOrderInfo['order_id'];
    $body .= "\n\n";
    $body .= '感谢您在 HatShop.com 购物！';
    return $body;
}
```

当此管道段完成时，订单状态将更改为 `8`，表示订单已完成。若再尝试使用 `OrderProcessor` 处理该订单，将引发异常。

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH14.qxd 11/17/06 3:47 PM Page 487

**487**

## 测试管道

现在我们来做一个简单的测试，以确保刚刚编写的代码按预期工作。

### 练习：测试管道

1. 在 `include/app_top.php` 文件中添加以下高亮显示的行（同时可以随意移除对 `ps_dummy.php` 的引用，该文件已不再需要）。

```php
require_once BUSINESS_DIR . 'order_processor.php';
require_once BUSINESS_DIR . 'ps_initial_notification.php';
require_once BUSINESS_DIR . 'ps_check_funds.php';
require_once BUSINESS_DIR . 'ps_check_stock.php';
require_once BUSINESS_DIR . 'ps_stock_ok.php';
require_once BUSINESS_DIR . 'ps_take_payment.php';
require_once BUSINESS_DIR . 'ps_ship_goods.php';
require_once BUSINESS_DIR . 'ps_ship_ok.php';
require_once BUSINESS_DIR . 'ps_final_notification.php';
```

2. 修改 `OrderProcessor`（位于 `business/order_processor.php`）中 `GetCurrentPipelineSection` 方法的代码如下：

```php
// 获取当前管道段
private function GetCurrentPipelineSection()
{
    switch($this->mOrderInfo['status'])
    {
        case 0:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsInitialNotification();
            break;
        case 1:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsCheckFunds();
            break;
        case 2:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsCheckStock();
            break;
        case 3:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsStockOk();
            break;
        // 以下省略部分 case，但保留原文结构
        [www.it-ebooks.info](http://www.it-ebooks.info/)

        648XCH14.qxd 11/17/06 3:47 PM Page 488

        **488**

        case 4:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsTakePayment();
            break;
        case 5:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsShipGoods();
            break;
        case 6:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsShipOk();
            break;
        case 7:
            $this->_mOrderProcessStage = $this->mOrderInfo['status'];
            $this->_mCurrentPipelineSection = new PsFinalNotification();
            break;
        case 8:
            $this->_mOrderProcessStage = 100;
            throw new Exception('订单已完成。');
            break;
        default:
            $this->_mOrderProcessStage = 100;
            throw new Exception('请求了未知的管道段。');
    }
}
```


**3.** 打开`business/orders.php`并修改`Orders`类的`$mOrdersStatusOptions`成员以管理新的订单状态码。注意，此更改会影响之前使用不同状态码的旧订单：

```php
public static $mOrderStatusOptions = array (

'Order placed, notifying customer', // 0

'Awaiting confirmation of funds', // 1

'Notifying supplier-stock check', // 2

'Awaiting stock confirmation', // 3

'Awaiting credit card payment', // 4

'Notifying supplier-shipping', // 5

'Awaiting shipment confirmation', // 6

'Sending final notification', // 7

'Order completed', // 8

'Order canceled'); // 9
```

**4.** 打开`presentation/smarty_plugins/function.load_admin_order_details.php`，并向`AdminOrderDetails`类添加高亮显示的新成员：

```php
public $mTaxCost = 0.0;

public $mOrderProcessMessage;
```

**5.** 如高亮所示，修改位于`presentation/smarty_plugins/function.load_admin_order_details.php`中`AdminOrderDetails`类的`init`方法代码。这将处理当访问者点击“Process”按钮时的必要功能。

```php
if (isset ($_GET['submitProcessOrder']))

{

$processor = new OrderProcessor($this->mOrderId);

try

{

$processor->Process();

$this->mOrderProcessMessage = 'Order processed, status now: ' .

$processor->mOrderInfo['status'];

}

catch (Exception $e)

{

$this->mOrderProcessMessage = 'Processing error, status now: ' .

$processor->mOrderInfo['status'];

}

}
```

**6.** 打开`presentation/templates/admin_order_details.tpl`文件，并添加高亮显示的代码：

```smarty
<span class="admin_page_text">

Editing details for order ID:

{$admin_order_details->mOrderInfo.order_id} [

{strip}

<a href="{$admin_order_details->mAdminOrdersPageLink|prepare_link:"https"}"> back to admin orders...

</a>

{/strip}

]

</span>

<br /><br />

{if $admin_order_details->mOrderProcessMessage}

<strong>{$admin_order_details->mOrderProcessMessage}</strong>

<br /><br />

{/if}

<form action="{"admin.php"|prepare_link:"https"}" method="get">
```

**7.** 执行代码，创建一个新订单，然后在订单管理页面中打开该订单。在订单管理页面中，点击“Process Order”按钮。

**8.** 您应该会收到一封客户通知邮件（参见图 14-1）。

**Figure 14-1.** *客户订单确认邮件*

**9.** 检查您的供应商邮箱，查找库存检查邮件（参见图 14-2）。

**Figure 14-2.** *库存检查邮件*

**10.** 在管理订单详情页面中，再次点击“Process Order”按钮继续处理，第二次调用`OrderProcessor`类的`Process`方法。

**11.** 检查您的邮箱，查找发货邮件（参见图 14-3）。

**Figure 14-3.** *发货邮件*

**12.** 在管理订单详情页面中，点击“Process”并第三次（也是最后一次）调用`OrderProcessor`类的`Process`方法，继续处理。

**13.** 检查您的邮箱，查找配送确认邮件（参见图 14-4）。

**Figure 14-4.** *客户配送通知邮件 (dispatched.png)*

**14.** 检查订单的新审计条目，如图 14-5 所示。



