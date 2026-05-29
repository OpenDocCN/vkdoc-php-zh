# 第 9 章 ■ 处理客户订单

```php
// 创建 AdminOrders 对象
$admin_orders = new AdminOrders();
$admin_orders->init();

// 分配模板变量
$smarty->assign($params['assign'], $admin_orders);
}

/* 支持订单管理功能的表示层类 */
class AdminOrders
{
    // 在 Smarty 模板中可用的公共变量
    public $mOrders;
    public $mStartDate;
    public $mEndDate;
    public $mRecordCount = 20;
    public $mOrderStatusOptions;
    public $mSelectedStatus = 0;
    public $mErrorMessage = '';

    // 类构造函数
    public function __construct()
    {
        /* 将当前页面的链接保存到 AdminOrdersPageLink 会话变量中；它将用于在后台订单详情页面中创建
        "返回后台订单..." 链接 */
        $_SESSION['admin_orders_page_link'] =
            str_replace(VIRTUAL_LOCATION, '', getenv('REQUEST_URI'));
        $this->mOrderStatusOptions = Orders::$mOrderStatusOptions;
    }

    public function init()
    {
        // 如果"显示最近的 x 个订单"过滤器正在生效……
        if (isset ($_GET['submitMostRecent']))
        {
            // 如果记录数值不是有效的整数，则显示错误
            if ((string)(int)$_GET['recordCount'] == (string)$_GET['recordCount'])
            {
                $this->mRecordCount = (int)$_GET['recordCount'];
                $this->mOrders = Orders::GetMostRecentOrders($this->mRecordCount);
            }
            else
                $this->mErrorMessage = $_GET['recordCount'] . ' 不是一个数字。';
        }
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH09.qxd 11/17/06 3:39 PM 第 323 页