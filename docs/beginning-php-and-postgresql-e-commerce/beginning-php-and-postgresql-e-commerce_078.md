# 第 9 章 ■ 处理客户订单

**323**

```php
        /* 如果"显示在 date_1 和 date_2 之间创建的所有记录"
        过滤器正在生效…… */
        if (isset ($_GET['submitBetweenDates']))
        {
            $this->mStartDate = $_GET['startDate'];
            $this->mEndDate = $_GET['endDate'];

            // 检查开始日期的格式是否可接受
            if (($this->mStartDate == '') ||
                ($timestamp = strtotime($this->mStartDate)) == -1)
                $this->mErrorMessage = '开始日期无效。 ';
            else
                // 将日期转换为 YYYY/MM/DD HH:MM:SS 格式
                $this->mStartDate =
                    strftime('%Y/%m/%d %H:%M:%S', strtotime($this->mStartDate));

            // 检查结束日期的格式是否可接受
            if (($this->mEndDate == '') ||
                ($timestamp = strtotime($this->mEndDate)) == -1)
                $this->mErrorMessage .= '结束日期无效。';
            else
                // 将日期转换为 YYYY/MM/DD HH:MM:SS 格式
                $this->mEndDate =
                    strftime('%Y/%m/%d %H:%M:%S', strtotime($this->mEndDate));
```



```php
// 检查开始日期是否晚于结束日期
if ((empty($this->mErrorMessage)) &&
    (strtotime($this->mStartDate) > strtotime($this->mEndDate)))
    $this->mErrorMessage .= '开始日期应晚于结束日期。';

// 如果没有错误，则获取两个日期之间的订单
if (empty($this->mErrorMessage))
    $this->mOrders = Orders::GetOrdersBetweenDates(
        $this->mStartDate, $this->mEndDate);
}

// 如果“按状态显示订单”筛选器处于激活状态……
if (isset($_GET['submitOrdersByStatus']))
{
    $this->mSelectedStatus = $_GET['status'];
    $this->mOrders = Orders::GetOrdersByStatus($this->mSelectedStatus);
}

// 构建“查看详情”链接
for ($i = 0; $i < count($this->mOrders); $i++)
{
    $this->mOrders[$i]['onclick'] =
        'admin.php?Page=OrderDetails&OrderId=' .
        $this->mOrders[$i]['order_id'];
}
```

3. 在浏览器中加载 `admin.php`，如果已注销，请重新输入用户名/密码组合。点击“订单管理”菜单链接，然后点击某个“执行！”按钮，查看结果，结果应与之前图 9-4 中显示的类似。

**工作原理：admin_orders 组件化模板**

每个“执行！”按钮都会调用业务层方法（位于 `Orders` 类中），并用返回的订单信息填充表格。

在处理请求时，我们会测试访问者输入的数据，以确保其有效。当点击第一个“执行！”按钮时，我们会验证输入的值是否为数字（要显示的记录数）。我们还会验证“开始日期”和“结束日期”文本框中输入的日期是否有效。我们首先用 `strtotime` 函数处理它们，该函数解析字符串并将其转换为 Unix 时间戳。这个函数很有用，因为它也接受诸如“now”、“tomorrow”、“last week”等输入值。生成的时间戳随后会由 `strftime` 函数处理，将其转换为 `YYYY/MM/DD HH:MM:SS` 格式。来看看这些日期/时间值是如何解析的：

```php
// 检查开始日期的格式是否可接受
if (($this->mStartDate == '') ||
    ($timestamp = strtotime($this->mStartDate)) == -1)
    $this->mErrorMessage = '开始日期无效。';
else
    // 将日期转换为 YYYY/MM/DD HH:MM:SS 格式
    $this->mStartDate =
        strftime('%Y/%m/%d %H:%M:%S', strtotime($this->mStartDate));
```

> **注意** 请查阅 [`www.php.net/strtotime`](http://www.php.net/strtotime) 了解 `strtotime` 函数支持的输入格式，以及 [`www.php.net/strftime`](http://www.php.net/strftime) 获取关于 `strftime` 的更多细节。

除此之外，`admin_orders.tpl` 模板文件相当简单，没有引入任何新的理论元素给你。

### 显示订单详情

在本节中，你将创建 `admin_order_details` 组件化模板，允许管理员编辑特定订单的详细信息。最常见的任务是将已下的订单标记为“已验证”或“已取消”，以及在发货时将已验证的订单标记为“已完成”。请查看图 9-5（前面已展示）以了解 `admin_order_details` 模板的实际效果。

当 PayPal 确认了某笔订单的付款时，站点管理员会将该订单标记为“已验证”；当订单完成组装、填写地址并邮寄给购买者时，则将其标记为“已完成”。例如，如果 PayPal 未能在合理时间（“合理”的确切含义由管理员决定）内确认付款，管理员可以将订单标记为“已取消”。

其他按钮——“编辑”、“更新”和“取消”——允许管理员手动编辑订单的任何详细信息。点击“编辑”按钮时，选择框和文本框将变为可编辑状态。



现在您已经对这个控件的功能有了大致了解，接下来我们将按照常规方式从数据层开始实现它。

## 实现数据层

在此部分，您将实现支持 UI 所需功能的数据层逻辑。

您需要让管理员能够执行三个操作，并将通过以下函数来实现它们：

- `orders_get_order_info`：获取填充表单所需的一般订单信息数据，例如总金额、创建日期、发货日期等。您可以在前面图 9-6 中看到完整列表。
- `orders_get_order_details`：返回属于所选订单的所有产品，其返回数据用于填充表单底部的网格。
- `orders_update_order`：当管理员在编辑模式下更新订单时调用。

现在，请按照下一步练习中的步骤来实现每个函数。

### 练习：实现相关函数

**1.** 加载 `pgAdmin III`，并连接到 `hatshop` 数据库。

**2.** 点击 `Tools` ➤ `Query tool`（或点击工具栏上的 `SQL` 按钮）。将会出现一个新的查询窗口。

**3.** 使用查询工具执行以下代码，该代码会在 `hatshop` 数据库中创建 `orders_get_order_info` 函数：

```sql
-- Create orders_get_order_info function

CREATE FUNCTION orders_get_order_info(INTEGER)

RETURNS orders LANGUAGE plpgsql AS $$

DECLARE

inOrderId ALIAS FOR $1;

outOrdersRow orders;

BEGIN

SELECT INTO outOrdersRow

order_id, total_amount, created_on, shipped_on, status, comments, customer_name, shipping_address, customer_email FROM orders

WHERE order_id = inOrderId;

RETURN outOrdersRow;

END;

$$;
```

该函数返回填充 `admin_order_details` 组件化模板中表单所需的信息。

**4.** 使用查询工具执行以下代码，该代码会在 `hatshop` 数据库中创建 `order_details` 类型和 `orders_get_order_details` 函数：

```sql
-- Create order_details type

CREATE TYPE order_details AS

(

order_id INTEGER,

product_id INTEGER,

product_name VARCHAR(50),

quantity INTEGER,

unit_cost NUMERIC(10, 2),

subtotal NUMERIC(10, 2)

);

-- Create orders_get_order_details function

CREATE FUNCTION orders_get_order_details(INTEGER)

RETURNS SETOF order_details LANGUAGE plpgsql AS $$

DECLARE

inOrderId ALIAS FOR $1;

outOrderDetailsRow order_details;

BEGIN

FOR outOrderDetailsRow IN

SELECT order_id, product_id, product_name, quantity,

unit_cost, (quantity * unit_cost) AS subtotal

FROM order_detail

WHERE order_id = inOrderId

LOOP

RETURN NEXT outOrderDetailsRow;

END LOOP;

END;

$$;
```

`orders_get_order_details` 函数返回属于特定订单的产品列表。该列表将用于填充页面底部的订单详情表格。

**5.** 使用查询工具执行以下代码，该代码会在 `hatshop` 数据库中创建 `orders_update_order` 函数：

```sql
-- Create orders_update_order function

CREATE FUNCTION orders_update_order(INTEGER, INTEGER, VARCHAR(255), VARCHAR(50), VARCHAR(255), VARCHAR(50))

RETURNS VOID LANGUAGE plpgsql AS $$

DECLARE

inOrderId ALIAS FOR $1;

inStatus ALIAS FOR $2;

inComments ALIAS FOR $3;

inCustomerName ALIAS FOR $4;

inShippingAddress ALIAS FOR $5;

inCustomerEmail ALIAS FOR $6;

currentStatus INTEGER;

BEGIN

SELECT INTO currentStatus

status

FROM orders

WHERE order_id = inOrderId;

IF inStatus != currentStatus AND (inStatus = 0 OR inStatus = 1) THEN

UPDATE orders SET shipped_on = NULL WHERE order_id = inOrderId;

ELSEIF inStatus != currentStatus AND inStatus = 2 THEN

UPDATE orders SET shipped_on = NOW() WHERE order_id = inOrderId;

END IF;

UPDATE orders

SET status = inStatus, comments = inComments,

customer_name = inCustomerName,

shipping_address = inShippingAddress,

customer_email = inCustomerEmail;
```



```sql
WHERE order_id = inOrderId;

END;

$$;
```

`orders_update_order` 函数用于更新订单的详细信息。

### 实现业务层

`admin_order_details` 组件化模板的业务层部分非常简单，由以下方法组成，你需要将这些方法添加到 `business/orders.php` 文件中的 `Orders` 类中：

```php
// 获取特定订单的详细信息
public static function GetOrderInfo($orderId)
{
    // 构建 SQL 查询语句
    $sql = 'SELECT * FROM orders_get_order_info(:order_id);';
    // 构建参数数组
    $params = array (':order_id' => $orderId);
    // 使用 PDO 特定功能准备语句
    $result = DatabaseHandler::Prepare($sql);
    // 执行查询并返回结果
    return DatabaseHandler::GetRow($result, $params);
}

// 获取属于特定订单的产品
public static function GetOrderDetails($orderId)
{
    // 构建 SQL 查询语句
    $sql = 'SELECT * FROM orders_get_order_details(:order_id);';
    // 构建参数数组
    $params = array (':order_id' => $orderId);
    // 使用 PDO 特定功能准备语句
    $result = DatabaseHandler::Prepare($sql);
    // 执行查询并返回结果
    return DatabaseHandler::GetAll($result, $params);
}

// 更新订单详细信息
public static function UpdateOrder($orderId, $status, $comments, $customerName, $shippingAddress, $customerEmail)
{
    // 构建 SQL 查询语句
    $sql = 'SELECT orders_update_order(:order_id, :status, :comments,
            :customer_name, :shipping_address, :customer_email);';
    // 构建参数数组
    $params = array (':order_id' => $orderId,
                     ':status' => $status,
                     ':comments' => $comments,
                     ':customer_name' => $customerName,
                     ':shipping_address' => $shippingAddress,
                     ':customer_email' => $customerEmail);
    // 使用 PDO 特定功能准备语句
    $result = DatabaseHandler::Prepare($sql);
    // 执行查询
    return DatabaseHandler::Execute($result, $params);
}
```

### 实现表示层

再次，你已到达将所有数据层和业务层功能包装并打包成美观用户界面的阶段。表示层由 `admin_order_details` 组件化模板组成。让我们在以下练习中创建这个组件化模板。

**练习：创建 admin_order_details 组件化模板**

1. 在 `presentation/templates` 文件夹中创建一个名为 `admin_order_details.tpl` 的新模板文件，并向其中添加以下代码：

```smarty
{* admin_order_details.tpl *}
{load_admin_order_details assign="admin_order_details"}
<span class="admin_page_text">
正在编辑订单 ID 的详细信息:
{$admin_order_details->mOrderInfo.order_id} [
{strip}
<a href="{$admin_order_details->mAdminOrdersPageLink|prepare_link:"https"}"> 返回管理员订单...
</a>
{/strip}
]
</span>
<br /><br />
<form action="{"admin.php"|prepare_link:"https"}" method="get">
<input type="hidden" name="Page" value="OrderDetails" />
<input type="hidden" name="OrderId"
       value="{$admin_order_details->mOrderInfo.order_id}" />
<table class="edit">
  <tr>
    <td class="admin_page_text">总金额: </td>
    <td class="price">
      ${$admin_order_details->mOrderInfo.total_amount}
    </td>
  </tr>
  <tr>
    <td class="admin_page_text">创建日期: </td>
    <td>
      {$admin_order_details->mOrderInfo.created_on|date_format:"%Y-%m-%d %T"}
    </td>
  </tr>
  <tr>
    <td class="admin_page_text">发货日期: </td>
    <td>
      {$admin_order_details->mOrderInfo.shipped_on|date_format:"%Y-%m-%d %T"}
    </td>
  </tr>
  <tr>
    <td class="admin_page_text">状态: </td>
    <td>
      <select name="status"
              {if ! $admin_order_details->mEditEnabled}
              disabled="disabled"
              {/if} >
```



`{html_options options=$admin_order_details->mOrderStatusOptions selected=$admin_order_details->mOrderInfo.status}`

</select>

</td>

</tr>

<tr>

<td class="admin_page_text">备注：</td>

<td>

<input name="comments" type="text" size="50"

value="{$admin_order_details->mOrderInfo.comments}"

{if ! $admin_order_details->mEditEnabled}

disabled="disabled"

{/if} />

</td>

</tr>

<tr>

<td class="admin_page_text">客户名称：</td>

<td>

<input name="customerName" type="text" size="50"

value="{$admin_order_details->mOrderInfo.customer_name}"

{if ! $admin_order_details->mEditEnabled}

disabled="disabled"

{/if} />

</td>

</tr>

<tr>

<td class="admin_page_text">收货地址：</td>

<td>

<input name="shippingAddress" type="text" size="50"

value="{$admin_order_details->mOrderInfo.shipping_address}"

{if ! $admin_order_details->mEditEnabled}

disabled="disabled"

{/if} />

</td>

</tr>

<tr>

<td class="admin_page_text">客户邮箱：</td>

<td>

<input name="customerEmail" type="text" size="50"

value="{$admin_order_details->mOrderInfo.customer_email}"

{if ! $admin_order_details->mEditEnabled}

disabled="disabled"

{/if} />

</td>

</tr>

</table>

---

**2.** 在 `presentation/smarty_plugins` 文件夹中创建一个新文件，命名为 `function.load_admin_order_details.php`，并在其中编写以下代码：

```php
<?php
// 插件文件中的插件函数必须命名为：smarty_type_name
function smarty_function_load_admin_order_details($params, $smarty)
{
  // 创建 AdminOrderDetils 对象
  $admin_order_details = new AdminOrderDetails();
  $admin_order_details->init();

  // 分配模板变量
  $smarty->assign($params['assign'], $admin_order_details);
}

// 处理订单详情管理的表示层类
class AdminOrderDetails
{
  // 在 Smarty 模板中可用的公共变量
  public $mOrderId;
  public $mOrderInfo;
  public $mOrderDetails;
  public $mEditEnabled;
  public $mOrderStatusOptions;
  public $mAdminOrdersPageLink;

  // 类构造函数
  public function __construct()
  {
    // 从会话中获取返回链接
    $this->mAdminOrdersPageLink = $_SESSION['admin_orders_page_link'];

    // 从查询字符串中接收订单 ID
    if (isset ($_GET['OrderId']))
      $this->mOrderId = (int) $_GET['OrderId'];
    else
      trigger_error('必须提供 OrderId 参数');

    $this->mOrderStatusOptions = Orders::$mOrderStatusOptions;
  }

  // 初始化类成员
  public function init()
  {
    if (isset ($_GET['submitUpdate']))
    {
      Orders::UpdateOrder($this->mOrderId, $_GET['status'], $_GET['comments'], $_GET['customerName'], $_GET['shippingAddress'], $_GET['customerEmail']);
    }

    $this->mOrderInfo = Orders::GetOrderInfo($this->mOrderId);
    $this->mOrderDetails = Orders::GetOrderDetails($this->mOrderId);

    // 指定启用或禁用编辑模式的值
    if (isset ($_GET['submitEdit']))
      $this->mEditEnabled = true;
    else
      $this->mEditEnabled = false;
  }
}
?>
```

**3.** 打开 `hatshop.css`，并添加以下样式：

```css
.edit tr td
{
  background: #ffffff;
  border: none;
}
```

**4.** 向数据库中添加一些模拟订单，然后在浏览器中加载 `admin.php` 文件。点击“订单管理”菜单链接，点击某个“Go！”按钮显示订单，然后点击其中一个订单的“查看详情”按钮。订单详情管理页面将会显示，允许你编辑订单详情，正如本章前面所述。

## 工作原理：`admin_order_details` 组件化模板

你刚刚编写的三个文件 `admin_order_details.tpl`、`function.load_admin_order_details.php` 和 `admin_order_details.php` 允许你查看和更新特定订单的详细信息。

函数插件通过常规机制从模板文件中加载。`AdminOrderDetails` 类的构造函数（`__construct` 方法）确保查询字符串中存在 `OrderId` 参数，因为如果没有这个参数，该组件化模板就没有意义：

```php
// 类构造函数
public function __construct()
{
  // 从会话中获取返回链接
  $this->mAdminOrdersPageLink = $_SESSION['admin_orders_page_link'];

  // 从查询字符串中接收订单 ID
  if (isset ($_GET['OrderId']))
    $this->mOrderId = (int) $_GET['OrderId'];
  else
    trigger_error('必须提供 OrderId 参数');

  $this->mOrderStatusOptions = Orders::$mOrderStatusOptions;
}
```

`init()` 方法响应用户操作，并调用各种业务层方法来完成用户请求。它通过调用业务层方法 `Orders::GetOrderInfo` 和 `Orders::GetOrderDetails` 获取数据来填充表单。

`mEditEnabled` 类成员根据查询字符串中 `submitEdit` 参数是否设置来决定进入或退出编辑模式。当进入编辑模式时，所有文本框以及“更新”和“取消”按钮变为可用，但“编辑”按钮被禁用。当退出编辑模式时（点击“取消”或“更新”按钮时发生），情况则相反。

## 总结

本章我们涉及了广泛的内容。通过两个独立的阶段，你实现了一个用于接收订单和手动管理订单的系统。你在购物车控件中添加了一个“下订单”按钮，以便访客能够下单购物车中的商品。你还实现了一个简单的订单管理页面，网站管理员可以在其中查看和处理待处理订单。

由于订单数据现在存储在数据库中，你可以根据已售商品进行各种统计和计算。在下一章中，你将学习如何实现“购买了此商品的顾客也购买了……”功能，如果没有数据库中存储的订单数据，这个功能是不可能实现的。

---

## 第 10 章：商品推荐

与实体店相比，互联网商店最重要的优势之一就是能够根据每位访客的偏好，或根据从具有相似偏好的其他访客那里收集的数据，来自定义网站。如果你的网站能够以巧妙的方式向访客推荐更多商品，那么他们最终可能会购买超出最初计划的商品。



