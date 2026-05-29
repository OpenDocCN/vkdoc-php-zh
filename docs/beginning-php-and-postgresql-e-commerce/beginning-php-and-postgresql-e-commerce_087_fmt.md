# 第 14 章 ■ II 实现订单管道：第二部分 **493**

**图 14-5.** *已完成订单的审计条目*

**工作原理：订单管道**

我们已经介绍了订单管道的工作原理，现在只需要解释添加到 `OrderProcessor` 中的新代码。我们修改了 `GetCurrentPipelineSection()` 方法中的代码，该方法负责选择需要执行的管道部分。

修改内容只是一个 `switch` 块，它将管道部分赋值给 `$_mCurrentPipelineSection` 成员：

```php
// 获取当前管道部分

private function GetCurrentPipelineSection()

{

switch($this->mOrderInfo['status'])

{

case 0:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsInitialNotification(); break;

case 1:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsCheckFunds(); break;

case 2:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsCheckStock(); break;

case 3:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsStockOk();

break;

case 4:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsTakePayment(); break;

case 5:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsShipGoods(); break;

case 6:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsShipOk();

break;

case 7:

$this->_mOrderProcessStage = $this->mOrderInfo['status']; $this->_mCurrentPipelineSection = new PsFinalNotification(); break;

case 8:

$this->_mOrderProcessStage = 100;

throw new Exception('订单已完成。');

break;

default:

$this->_mOrderProcessStage = 100;

throw new Exception('请求了未知的管道部分。');

}

}
```

如果订单已完成或请求了未知部分，则会生成异常。

测试代码为您提供了额外测试此异常生成的机会，因为如果您再次运行它，您将处理一个已完成的订单。针对已完成（状态为 8）的订单单击“处理订单”按钮，您应该会收到如图 14-6 所示的错误邮件。

**图 14-6.** *订单完成错误邮件*

发送给管理员的错误消息应足以帮助您开始排查问题。

**更新结账页面**

在前面的示例中，您被迫从订单详情管理页面连续三次调用 `OrderProcessor::Process()` 方法。在实践中，这种情况不会发生——当客户下单时，它会由 `presentation/smarty_plugins/function.load_checkout_info.php` 调用一次，再由供应商在 `presentation/smarty_plugin/function.load_admin_order_details.php` 中调用两次。您需要相应地修改这些网页。

您还需要在 `index.php` 中添加对新类的引用。按照本练习中的步骤，使 `function.load_checkout_info.php` 能与新的订单管道协同工作。

**练习：更新结账流程**

**1.** 修改 `presentation/smarty_plugins/function.load_checkout_info.php` 中 `CheckoutInfo` 类的 `init()` 方法，添加高亮显示的代码：

```php
public function init()

{

// 如果点击了“下单”按钮，将订单保存到数据库 if ($this->_mPlaceOrder == 1)

{

$this->mCustomerData = Customer::Get();

$tax_id = '';

switch ($this->mCustomerData['shipping_region_id'])

{

case 2:

$tax_id = 1;

break;

default:

$tax_id = 2;

}

$order_id = ShoppingCart::CreateOrder(

$this->mCustomerData['customer_id'],

(int)$_POST['shipping'], $tax_id);

$redirect_page = '';

// 创建新的 OrderProcessor 实例

$processor = new OrderProcessor($order_id);

try

{

$processor->Process();

}

catch (Exception $e)

{

// 如果发生错误，跳转到错误页面

$redirect_page = 'index.php?OrderError';

}

// 成功时跳转到订单成功页面

$redirect_page = 'index.php?OrderDone';

// 重定向到 index.php

$redirect_link = 'http://' . getenv('SERVER_NAME');

// 如果 HTTP_SERVER_PORT 已定义且不同于默认值 if (defined('HTTP_SERVER_PORT') && HTTP_SERVER_PORT != '80')

{

// 追加服务器端口

$redirect_link .= ':' . HTTP_SERVER_PORT;

}

$redirect_link .= VIRTUAL_LOCATION . $redirect_page; header('Location:' . $redirect_link);

exit;

}

...
```

**2.** 在 `presentation/templates` 文件夹中创建一个名为 `order_done.tpl` 的新文件，并在其主体中添加以下代码：

```smarty
{* order_done.tpl *}

<br />

<span class="description">感谢您的订单！</span>

<br /><br />

<strong>确认邮件很快就会发送到您的邮箱。</strong>
```

**3.** 如果下单时发生错误，则重定向到另一个页面。创建 `presentation/templates/order_error.tpl`，内容如下：

```smarty
{* order_error.tpl *}

<br />

<span class="description">

处理您的订单时发生错误。</span>

<br /><br />

<strong>

如果您对此消息有疑问，请发送电子邮件至

<a class="mail" href="mailto:CustomerService@example.com"> CustomerService@example.com</a>

</strong>
```

**4.** 在 `hatshop.css` 中添加以下样式：

```css
a.mail

{

color: #0000ff;

font-size: 11px;

text-decoration: underline;

}
```

**5.** 修改 `index.php`，添加高亮显示的代码，以便根据订单处理是否成功来加载 `order_done.tpl` 或 `order_error.tpl`：

```
if (isset($_GET['RegisterCustomer']) || isset($_GET['UpdateAccountDetails'])) $pageContentsCell = 'customer_details.tpl';

elseif (isset($_GET['UpdateAddressDetails']))

$pageContentsCell = 'customer_address.tpl';

elseif (isset($_GET['UpdateCreditCardDetails']))

$pageContentsCell = 'customer_credit_card.tpl';

if (isset($_GET['OrderDone']))

$pageContentsCell = 'order_done.tpl';

elseif (isset($_GET['OrderError']))

$pageContentsCell = 'order_error.tpl';

$page->assign('hide_boxes', $hide_boxes);

$page->assign('customerLoginOrLogged', $customerLoginOrLogged);
```

现在您可以使用 HatShop 网络商店下单，但订单在进入库存确认阶段时会暂停。要继续进行，您将实现供供应商和管理员使用的接口，以强制订单继续处理。

## 更新订单管理页面

此页面的基本功能是允许供应商和管理员查看需要关注的订单列表，并手动推进管道中的订单。这只需按前面所述调用 `OrderProcess::Process()` 方法即可。

此页面可以通过多种方式实现。实际上，在某些设置中，将其作为独立应用程序实现可能更好，例如，如果您的供应商在内部且位于同一网络上。或者，将这种方法与 Web 服务结合使用可能会更好。

为了简化本节内容，您将为管理员和供应商提供一个单一页面。这可能并非在所有情况下都是理想的选择，因为您可能不希望将所有的订单详情和审计信息暴露给外部供应商。但是，出于演示目的，这可以减少您需要编写的代码量。您还将此页面的安全性与本书前面使用的基于管理员表单的安全性相结合，假设有权编辑站点数据的人员也有权管理订单。在更高级的设置中，您可以稍作修改，为不同类型的用户提供角色，并限制不同角色用户可用的功能。

## 实现数据层

我们需要向 `hatshop` 数据库添加一个新的数据层函数 `orders_get_audit_trail`，并更新一个现有函数 `orders_update_order`，以考虑新的状态代码。

使用 `pgAdmin III`，连接到 `hatshop` 数据库，并使用查询工具执行以下代码，该代码将在您的 `hatshop` 数据库中创建 `orders_update_order` 函数：

```sql
-- Update orders_update_order function

CREATE OR REPLACE FUNCTION orders_update_order(INTEGER, INTEGER, VARCHAR(255), VARCHAR(50), VARCHAR(50))

RETURNS VOID LANGUAGE plpgsql AS $$

DECLARE

inOrderId ALIAS FOR $1;

inStatus ALIAS FOR $2;

inComments ALIAS FOR $3;

inAuthCode ALIAS FOR $4;

inReference ALIAS FOR $5;

currentDateShipped TIMESTAMP;

BEGIN

SELECT INTO currentDateShipped

shipped_on

FROM orders

WHERE order_id = inOrderId;

UPDATE orders

SET status = inStatus, comments = inComments,

auth_code = inAuthCode, reference = inReference

WHERE order_id = inOrderId;

IF inStatus < 7 AND currentDateShipped IS NOT NULL THEN

UPDATE orders SET shipped_on = NULL WHERE order_id = inOrderId; ELSEIF inStatus > 6 AND currentDateShipped IS NULL THEN

UPDATE orders SET shipped_on = NOW() WHERE order_id = inOrderId; END IF;

END;

$$;
```

然后，使用查询工具执行以下代码，该代码将在您的 `hatshop` 数据库中创建 `orders_get_audit_trail` 函数：

```sql
-- Create orders_get_audit_trail function

CREATE FUNCTION orders_get_audit_trail(INTEGER)

RETURNS SETOF audit LANGUAGE plpgsql AS $$

DECLARE

inOrderId ALIAS FOR $1;

outAuditRow audit;

BEGIN

FOR outAuditRow IN

SELECT audit_id, order_id, created_on, message, message_number FROM audit

WHERE order_id = inOrderId

LOOP

RETURN NEXT outAuditRow;

END LOOP;

END;

$$;
```

## 实现业务层

您还必须向 `business/orders.php` 中的 `Orders` 类添加一个新方法，以配合上一节中添加的新数据层函数。

将 `GetAuditTrail` 方法添加到 `business/orders.php` 的 `Orders` 类中，如下所示：

```php
// Gets the audit table entries associated with a specific order public static function GetAuditTrail($orderId)
{
    // Build the SQL query
    $sql = 'SELECT * FROM orders_get_audit_trail(:order_id);';

    // Build the parameters array
    $params = array (':order_id' => $orderId);

    // Prepare the statement with PDO-specific functionality $result = DatabaseHandler::Prepare($sql);

    // Execute the query and return the results
    return DatabaseHandler::GetAll($result, $params);
}
```

## 实现表示层

您需要更新 `admin_order_details` 组合模板，该模板用于显示订单详情。在本书前面，此组合模板还包括测试订单流程的功能，但我们在此将其移除。取而代之的是，您将提供当订单卡在“等待库存确认”和“等待发货确认”阶段时，推动订单沿管线前进的能力。

现在，您还可以在另一个新表格中显示订单的所有审计信息。

让我们看看您将要实现的目标，如图 14-7 所示。

您可以将订单管理页面分为三个部分：

*   在第一部分，我们将“处理”按钮改为供应商的“确认”按钮。

*   在第二部分，一个表格填充了订单中的商品数据。

*   在第三部分，一个表格显示了订单的审计跟踪。

**图 14-7.** *新的订单详情管理页面* 您将在下一个练习中实现新功能。

## 练习：修改订单详情管理部分

**1.** 从 `presentation/templates/admin_order_details.tpl` 中删除以下行：

```smarty
{if $admin_order_details->mOrderProcessMessage}
<strong>{$admin_order_details->mOrderProcessMessage}</strong>
<br /><br />
{/if}
```

**2.** 同样在 `presentation/templates/admin_order_details.tpl` 中，将“处理订单”按钮代码替换为突出显示的代码：

### 3. 在 `presentation/templates/admin_order_details.tpl` 文件中，添加以下突出显示的代码：

```smarty
{section name=cOrder loop=$admin_order_details->mOrderDetails}
<tr>
<td>{$admin_order_details->mOrderDetails[cOrder].product_id}</td>
<td>{$admin_order_details->mOrderDetails[cOrder].product_name}</td>
<td>{$admin_order_details->mOrderDetails[cOrder].quantity}</td>
<td>${$admin_order_details->mOrderDetails[cOrder].unit_cost}</td>
<td>${$admin_order_details->mOrderDetails[cOrder].subtotal}</td>
</tr>
{/section}
</table>
<br /><br />
<span class="admin_page_text">Order audit trail:</span>
<br /><br />
<table>
<tr>
<th>Audit ID</th>
<th>Created On</th>
<th>Message Number</th>
<th>Message</th>
</tr>
{section name=cOrder loop=$admin_order_details->mAuditTrail}
<tr>
<td>{$admin_order_details->mAuditTrail[cOrder].audit_id}</td>
<td>{$admin_order_details->mAuditTrail[cOrder].created_on}</td>
<td>{$admin_order_details->mAuditTrail[cOrder].message_number}</td>
<td>{$admin_order_details->mAuditTrail[cOrder].message}</td>
</tr>
{/section}
</table>
</form>
```

### 4. 打开 `presentation/smarty_plugins/function.load_admin_order_details.php` 文件，并删除此处所示的 `AdminOrderDetails` 类的 `$mOrderProcessMessage` 成员的定义：

```php
public $mOrderProcessMessage;
```

### 5. 同样在 `function.load_admin_order_details.php` 文件中，向 `AdminOrderDetails` 类添加两个新成员：

```php
public $mProcessButtonText;
public $mAuditTrail;
```

### 6. 在同一文件中，通过添加此处突出显示的代码来修改 `AdminOrderDetails` 类的 `init` 方法：

```php
if (isset ($_GET['submitUpdate']))
{
    Orders::UpdateOrder($this->mOrderId, $_GET['status'], $_GET['comments'], $_GET['authCode'], $_GET['reference']);
}
if (isset ($_GET['submitProcessOrder']))
{
    $processor = new OrderProcessor($this->mOrderId);
    $processor->Process();
}
```

`$this->mOrderInfo = Orders::GetOrderInfo($this->mOrderId);`

`$this->mOrderDetails = Orders::GetOrderDetails($this->mOrderId);`

`$this->mAuditTrail = Orders::GetAuditTrail($this->mOrderId);`

`$this->mCustomerInfo = Customer::Get($this->mOrderInfo['customer_id']);`

`$this->mTotalCost = $this->mOrderInfo['total_amount'];`

```php
// 格式化数值
$this->mTotalCost = number_format($this->mTotalCost, 2, '.', '');
$this->mTaxCost = number_format($this->mTaxCost, 2, '.', '');
if ($this->mOrderInfo['status'] == 3)
    $this->mProcessButtonText = '确认订单库存';
elseif ($this->mOrderInfo['status'] == 6)
    $this->mProcessButtonText = '确认订单发货';
```

```php
// 用于指定是否启用或禁用编辑模式的值
if (isset ($_GET['submitEdit']))
    $this->mEditEnabled = true;
else
    $this->mEditEnabled = false;
```

### 6. 加载 HatShop，创建一个新订单，然后加载订单详情管理页面以测试新更改。

## 工作原理：订单详情管理

`AdminOrderDetails` 中定义的 `init` 方法，当点击“处理”按钮时，会将管道推进到下一阶段；该按钮在页面上的显示与否取决于 `mProcessButtonText` 成员的值。

如果当前管道阶段为 3（等待库存确认），则该值设置为“确认库存”；如果当前管道阶段为 6（等待发货确认），则设置为“确认发货”。如果当前管道阶段既不是 3 也不是 6，则表示订单已成功完成，此时不会显示该按钮。管理员可以随时通过查看页面上显示的审计轨迹来了解订单的处理情况。

现在剩下要做的就是检查一切是否正常运行。为此，请通过 Web 界面下一个订单，然后通过订单详情管理部分进行查看。您应该会看到订单正在等待库存确认，如图 14-7 所示。

点击**确认订单库存**按钮，订单即被处理。由于此过程非常迅速，您很快会进入下一阶段，此时**确认订单库存**按钮会被替换为一个名为**确认发货**的新按钮，并且审计轨迹会显示一组新数据。

点击**确认发货**按钮即可完成订单。如果您向下滚动页面，可以查看数据库中存储的所有与此订单相关的审计轨迹消息。

## 本章小结

在本章中，您在完成 HatShop 电子商务应用程序的道路上取得了巨大进展。现在，您已经为应用程序构建了一个经过全面审计、安全的骨架。

具体来说，我们涵盖了以下内容：

- 对 HatShop 应用程序进行的修改，以启用您自己的管道处理

- 订单管道的基本框架

- 用于审计数据以及在订单表中存储额外所需数据的数据库新增内容

- 除处理信用卡部分外，大部分订单管道的实现

在将本应用程序交付给外部世界之前，唯一缺少且需要添加的是信用卡处理功能，我们将在下一章中对此进行探讨。