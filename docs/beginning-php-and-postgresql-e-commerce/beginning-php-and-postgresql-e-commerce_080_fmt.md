# 产品推荐

## 添加产品推荐

请确保您理解前面解释的数据层逻辑，因为您将在`catalog_get_recommendations`数据库函数中实现它。与之前显示的查询唯一重要的区别是，您还需要获取产品描述，该描述将被截断为指定的字符数。

`catalog_get_recommendations`数据库函数在显示与所选产品一起订购了哪些产品时被调用。按照下一个练习中的步骤，将`catalog_get_recommendations`函数添加到`hatshop`数据库中。

### 练习：添加`catalog_get_recommendations`函数

1. 加载`pgAdmin III`，并连接到`hatshop`数据库。

2. 点击工具 ➤ 查询工具（或点击工具栏上的`SQL`按钮）。将出现一个新的查询窗口。

3. 使用查询工具执行以下代码，该代码将在您的`hatshop`数据库中创建`product_recommendation`类型和`catalog_get_recommendations`函数：

```sql
-- Create product_recommendation type
CREATE TYPE product_recommendation AS
(
  product_id INTEGER,
  name VARCHAR(50),
  description VARCHAR(1000)
);

-- Create catalog_get_recommendations function
CREATE FUNCTION catalog_get_recommendations(INTEGER, INTEGER) RETURNS SETOF product_recommendation LANGUAGE plpgsql AS $$
DECLARE
  inProductId ALIAS FOR $1;
  inShortProductDescriptionLength ALIAS FOR $2;
  outProductRecommendationRow product_recommendation;
BEGIN
  FOR outProductRecommendationRow IN
    SELECT product_id, name, description
    FROM product
    WHERE product_id IN
    (SELECT od2.product_id
     FROM order_detail od1
     JOIN order_detail od2
       ON od1.order_id = od2.order_id
     WHERE od1.product_id = inProductId
       AND od2.product_id != inProductId
     GROUP BY od2.product_id
     ORDER BY COUNT(od2.product_id) DESC
     LIMIT 5)
  LOOP
    IF char_length(outProductRecommendationRow.description) > inShortProductDescriptionLength THEN
      outProductRecommendationRow.description :=
        substring(outProductRecommendationRow.description, 1, inShortProductDescriptionLength) || '...';
    END IF;
    RETURN NEXT outProductRecommendationRow;
  END LOOP;
END;
$$;
```

### 使用子查询的替代方案

由于`SQL`非常灵活，`catalog_get_recommendations`可以用多种方式编写。在我们的案例中，一种流行的替代表联接的方法是使用子查询。以下是一个使用子查询代替联接的`catalog_get_recommendations`版本。带注释的代码不言自明：

```sql
-- Create catalog_get_recommendations function
CREATE OR REPLACE FUNCTION catalog_get_recommendations(INTEGER, INTEGER) RETURNS SETOF product_recommendation LANGUAGE plpgsql AS $$
DECLARE
  inProductId ALIAS FOR $1;
  inShortProductDescriptionLength ALIAS FOR $2;
  outProductRecommendationRow product_recommendation;
BEGIN
  FOR outProductRecommendationRow IN
    -- 返回产品推荐
    SELECT product_id, name, description
    FROM product
    WHERE product_id IN
      (-- 返回与 inProductId 一起被订购的产品
       SELECT product_id
       FROM order_detail
       WHERE order_id IN
         (-- 返回包含 inProductId 的订单
          SELECT DISTINCT order_id
          FROM order_detail
          WHERE product_id = inProductId
          LIMIT 5)
         -- 不能包含访客购物车中已有的产品
         AND product_id != inProductId
         -- 对 product_id 进行分组，以便计算排序
         GROUP BY product_id
         -- 按排序降序排列
         ORDER BY COUNT(product_id) DESC
         LIMIT 5)
  LOOP
    IF char_length(outProductRecommendationRow.description) > inShortProductDescriptionLength THEN
      outProductRecommendationRow.description :=
        substring(outProductRecommendationRow.description, 1, inShortProductDescriptionLength) || '...';
    END IF;
    RETURN NEXT outProductRecommendationRow;
  END LOOP;
END;
$$;
```

## 添加购物车推荐

显示购物车推荐的逻辑与你之前所做的非常相似，只不过现在需要考虑购物车中的所有产品，而不是单个产品。按照下一个练习中的步骤，将`shopping_cart_get_recommendations`函数添加到`hatshop`数据库中。

### 练习：添加`shopping_cart_get_recommendations`函数

1. 加载`pgAdmin III`，并连接到`hatshop`数据库。

2. 单击工具 ➤ 查询工具（或单击工具栏上的`SQL`按钮）。此时会出现一个新的查询窗口。

3. 使用查询工具执行以下代码，该代码将在你的`hatshop`数据库中创建`shopping_cart_get_recommendations`函数：

```sql
-- 创建 shopping_cart_get_recommendations 函数

CREATE FUNCTION shopping_cart_get_recommendations(CHAR(32), INTEGER) RETURNS SETOF product_recommendation LANGUAGE plpgsql AS $$

DECLARE

inCartId ALIAS FOR $1;

inShortProductDescriptionLength ALIAS FOR $2;

outProductRecommendationRow product_recommendation;

BEGIN

FOR outProductRecommendationRow IN

-- 返回产品推荐

SELECT product_id, name, description

FROM product

WHERE product_id IN

(-- 返回存在于某个订单列表中的产品

SELECT od1.product_id

FROM order_detail od1

JOIN order_detail od2

ON od1.order_id = od2.order_id

JOIN shopping_cart

ON od2.product_id = shopping_cart.product_id

WHERE shopping_cart.cart_id = inCartId

-- 不能包含访客购物车中已有的产品

AND od1.product_id NOT IN

(-- 返回指定购物车中的产品

SELECT product_id

FROM shopping_cart

WHERE cart_id = inCartId)

-- 对 product_id 进行分组，以便计算排序

GROUP BY od1.product_id

-- 按排序降序排列

ORDER BY COUNT(od1.product_id) DESC

LIMIT 5)

LOOP

IF char_length(outProductRecommendationRow.description) > inShortProductDescriptionLength THEN

outProductRecommendationRow.description :=

substring(outProductRecommendationRow.description, 1,

inShortProductDescriptionLength) || '...';

END IF;

RETURN NEXT outProductRecommendationRow;

END LOOP;

END;

$$;
```

此函数的替代版本（使用子查询而不是表连接）如下所示：

```sql
-- 创建 shopping_cart_get_recommendations 函数

CREATE OR REPLACE FUNCTION shopping_cart_get_recommendations(CHAR(32), INTEGER) RETURNS SETOF product_recommendation LANGUAGE plpgsql AS $$

DECLARE

inCartId ALIAS FOR $1;

inShortProductDescriptionLength ALIAS FOR $2;

outProductRecommendationRow product_recommendation;

BEGIN

FOR outProductRecommendationRow IN

-- 返回产品推荐

SELECT product_id, name, description

FROM product

WHERE product_id IN

(-- 返回存在于订单列表中的商品

SELECT product_id

FROM order_detail

WHERE order_id IN

(-- 返回包含特定商品的订单

SELECT DISTINCT order_id

FROM order_detail

WHERE product_id IN

(-- 返回指定购物车中的商品

SELECT product_id

FROM shopping_cart

WHERE cart_id = inCartId))

-- 不得包含已存在于访客购物车中的商品

AND product_id NOT IN

(-- 返回指定购物车中的商品

SELECT product_id

FROM shopping_cart

WHERE cart_id = inCartId)

-- 按 product_id 分组，以便计算排名

GROUP BY product_id

-- 按排名降序排列

ORDER BY COUNT(product_id) DESC

LIMIT 5)

LOOP

IF char_length(outProductRecommendationRow.description) > inShortProductDescriptionLength THEN

outProductRecommendationRow.description :=

substring(outProductRecommendationRow.description, 1,

inShortProductDescriptionLength) || '...';

END IF;

RETURN NEXT outProductRecommendationRow;

END LOOP;

END;

$$;
```

## 实现业务层

商品推荐系统的业务层包含两个方法，均名为 `GetRecommendations`。其中一个位于 `Catalog` 类中，用于获取商品详情页的推荐；另一个位于 `ShoppingCart` 类中，用于获取将显示在访客购物车中的推荐。

## 练习：实现业务逻辑

**1.** 将以下代码添加到 `business/catalog.php` 文件中：

```php
// 获取商品推荐
public static function GetRecommendations($productId)
{
    // 构建 SQL 查询
    $sql = 'SELECT * FROM catalog_get_recommendations(
        :product_id, :short_product_description_length);';
    // 构建参数数组
    $params = array (':product_id' => $productId,
                    ':short_product_description_length' =>
                    SHORT_PRODUCT_DESCRIPTION_LENGTH);
    // 使用 PDO 特定功能准备语句
    $result = DatabaseHandler::Prepare($sql);
    // 执行查询并返回结果
    return DatabaseHandler::GetAll($result, $params);
}
```

**2.** 打开位于 `business` 文件夹中的 `shopping_cart.php` 文件，并添加以下代码：

```php
// 获取购物车的商品推荐
public static function GetRecommendations()
{
    // 构建 SQL 查询
    $sql = 'SELECT * FROM shopping_cart_get_recommendations(
        :cart_id, :short_product_description_length);';
    // 构建参数数组
    $params = array (':cart_id' => self::GetCartId(),
                    ':short_product_description_length' =>
                    SHORT_PRODUCT_DESCRIPTION_LENGTH);
    // 使用 PDO 特定功能准备语句
    $result = DatabaseHandler::Prepare($sql);
    // 执行查询并返回结果
    return DatabaseHandler::GetAll($result, $params);
}
```

## 实现表示层

下一个练习将演示如何更新 `product` 和 `cart_details` 组件化模板，以显示商品推荐。

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH10.qxd 11/19/06 1:59 PM Page 348

**348** 第 10 章 ■ 商品推荐

### 练习：更新 product 和 cart_details 组件化模板

**1.** 打开 `presentation/smarty_plugins/function.load_product.php` 文件，并向 `Product` 类添加 `$mRecommendation` 成员：

```php
// 用于 Smarty 模板的公共变量
public $mProduct;
public $mPageLink = 'index.php';
public $mAddToCartLink;
public $mRecommendations;
```

**2.** 现在，你需要在 `$mRecommendations` 中获取推荐商品数据，并创建指向其主页的链接。按如下高亮显示的代码修改 `Product` 类的 `init()` 方法：

```php
$this->mAddToCartLink = 'index.php?ProductID=' . $this->_mProductId .
                        '&CartAction=' . ADD_PRODUCT;
// 获取商品推荐
```

`$this->mRecommendations =`

`Catalog::GetRecommendations($this->_mProductId);`

`// 创建推荐产品链接`

`for ($i = 0; $i < count($this->mRecommendations); $i++)` `$this->mRecommendations[$i]['link'] = 'index.php?ProductID=' .`

`$this->mRecommendations[$i]['product_id'];`

3. 完善产品详情页推荐系统的最后一步是更新产品模板，以显示推荐列表。请在`presentation/templates/product.tpl`文件末尾添加以下代码行：

```
{if $product->mRecommendations}

<br /><br />

<span class="description">购买了此商品的顾客也购买了：</span>

{section name=m loop=$product->mRecommendations}

<br /><br />

{strip}

<a class="product_recommendation"

href="{$product->mRecommendations[m].link|prepare_link:"http"}">

{$product->mRecommendations[m].name}

</a>

{/strip}

-

<span>{$product->mRecommendations[m].description}</span>

{/section}

{/if}
```

4. 打开`hatshop.css`，并添加以下样式：

```
.product_recommendation

{

color: #0000ff;

text-decoration: underline;

}
```

5. 现在，让我们修改`cart_details`组件化模板以显示产品推荐。打开位于`presentation/smarty_plugins`文件夹中的`function.load_cart_details.php`，为`CartDetails`类添加`$mRecommendation`成员：

```
public $mCartReferrer = 'index.php';
public $mCartDetailsTarget;
public $mRecommendations;
```

6. 接下来，你需要将推荐产品数据存入`$mRecommendations`，并为它们创建指向其主页的链接。请按高亮部分所示修改`CartDetails`类的`init()`方法：

```
$this->mSavedCartProducts[$i]['remove'] = 'index.php?ProductID=' .
$this->mSavedCartProducts[$i]['product_id'] .
'&CartAction=' . REMOVE_PRODUCT;

}

// 获取购物车的产品推荐
$this->mRecommendations =
ShoppingCart::GetRecommendations();

// 创建推荐产品链接
for ($i = 0; $i < count($this->mRecommendations); $i++)
$this->mRecommendations[$i]['link'] = 'index.php?ProductID=' .
$this->mRecommendations[$i]['product_id'];

}
}
?>
```

7. 最后一步是更新`cart_details`模板以显示推荐列表。请在`presentation/templates/cart_details.tpl`文件末尾添加以下代码行：

```
{if $cart_details->mRecommendations}

<br /><br />

<span class="description">购买了此商品的顾客也购买了：</span>

{section name=m loop=$cart_details->mRecommendations}

<br /><br />

{strip}

<a class="product_recommendation"

href="{$cart_details->mRecommendations[m].link|prepare_link:"http"}">

{$cart_details->mRecommendations[m].name}

</a>

{/strip}

-

<span>{$cart_details->mRecommendations[m].description}</span>

{/section}

{/if}
```

8. 启动 HatShop 应用，下几笔订单，然后检查产品详情页和购物车详情页是否根据订购的产品显示了推荐信息！结果应该类似于本章前面展示的图 10-1 和图 10-2。

## 总结

展示产品推荐是促进销售的好方法，我们在这简短的一章中成功地实现了这一功能。最大的挑战是构建获取推荐产品列表的 SQL 查询，我们逐步分析了如何创建它。

在下一章中，你将进入开发的第三阶段，即添加客户账户功能。

## 开发的第三阶段

### 管理客户详细信息