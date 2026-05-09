# 第 9 章 ■ 处理客户订单

**321**

```
<input type="submit" name="submitBetweenDates" value="Go!" />
```

```
<br /><br />
```

```
<span class="admin_page_text">按状态显示订单</span>
```

```
{html_options name="status" options=$admin_orders->mOrderStatusOptions selected=$admin_orders->mSelectedStatus}
```

```
<input type="submit" name="submitOrdersByStatus" value="Go!" />
```

```
<br /><br />
```

```
</form>
```

```
<br />
```

```
{if $admin_orders->mOrders}
```

```
<table>
```

```
<tr>
<th>订单 ID</th>
<th>创建日期</th>
<th>发货日期</th>
<th>状态</th>
<th>客户</th>
<th> </th>
</tr>
```

```
{section name=cOrders loop=$admin_orders->mOrders}
{assign var=status value=$admin_orders->mOrders[cOrders].status}
```

```
<tr>
<td>{$admin_orders->mOrders[cOrders].order_id}</td>
<td>
{$admin_orders->mOrders[cOrders].created_on|date_format:"%Y-%m-%d %T"}
</td>
<td>
{$admin_orders->mOrders[cOrders].shipped_on|date_format:"%Y-%m-%d %T"}
</td>
<td>{$admin_orders->mOrderStatusOptions[$status]}</td>
<td>{$admin_orders->mOrders[cOrders].customer_name}</td>
<td align="right">
<input type="button" value="查看详情"
onclick="window.location='{
$admin_orders->mOrders[cOrders].onclick|prepare_link:"https"}';" />
</td>
</tr>
```

```
{/section}
```

```
</table>
```

```
{/if}
```

2. 创建一个名为 `presentation/smarty_plugins/function.load_admin_orders.php` 的新文件，并向其中添加以下代码：

```php
<?php

// 插件文件内的插件函数必须命名为：smarty_type_name
function smarty_function_load_admin_orders($params, $smarty)
{
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH09.qxd 11/17/06 3:39 PM 第 322 页

**322**

