# 排版后内容

此时，谁购买了你的产品并不重要，你只需要知道哪些产品被售出以及何时售出。当你在后续章节中添加用户自定义功能时，你的任务会变得相当简单：当访客进行身份验证时，该访客的临时（匿名）购物车将与其账户相关联。由于你使用的是临时购物车，即使在实现客户账户系统之后，访客也无需在必要时刻之前提前提供额外信息（登录）。

存储购物车信息的最佳方式可能是为每个购物车生成一个唯一的 `cart ID`，并将其作为 cookie 保存在访客的计算机上。当访客点击“添加到购物车”按钮时，服务器首先会验证客户端计算机上是否存在该 cookie。如果存在，则将指定产品添加到现有购物车中。否则，服务器会生成另一个 `cart ID`，将其保存到客户端的 cookie 中，然后将该产品添加到新生成的购物车中。

在上一章中，你通过从表示层组件开始，创建了组件化的模板。然而，这种方法在此处行不通，因为现在你需要事先做一些设计工作，所以我们将采用更常见的方法，从数据库层开始。

## 存储购物车信息

你将把购物车中的所有信息存储在一个名为 `shopping_cart` 的单一数据表中。按照下面的练习来创建 `shopping_cart` 表。

### 练习：创建 `shopping_cart` 表

**1.** 加载 pgAdmin III，并连接到 `hatshop` 数据库。

**2.** 点击“工具”➤“查询工具”（或点击工具栏上的 SQL 按钮）。此时应出现一个新的查询窗口。

**3.** 使用查询工具执行以下代码，该代码将在你的 `hatshop` 数据库中创建 `shopping_cart` 表：

```sql
-- 创建 shopping_cart 表

CREATE TABLE shopping_cart
(
  cart_id CHAR(32) NOT NULL,
  product_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL,
  buy_now BOOLEAN NOT NULL DEFAULT true,
  added_on TIMESTAMP NOT NULL,
  CONSTRAINT pk_cart_id_product_id PRIMARY KEY (cart_id, product_id),
  CONSTRAINT fk_product_id FOREIGN KEY (product_id) REFERENCES product (product_id)
    ON UPDATE RESTRICT ON DELETE RESTRICT
);
```

### 工作原理：`shopping_cart` 表

我们来查看 `shopping_cart` 表中的每个字段：

- `cart_id` 存储一个唯一的 ID，你将为你为每个购物车生成。它不像之前创建的其他 ID 列那样是整数字段。它是一个 `char` 字段，将填充一个唯一 ID 的 MD5 哈希值，该哈希值是一个 32 字符的字符串。
- `product_id` 引用一个现有产品的 ID。
- `quantity` 存储产品在购物车中的数量。
- `buy_now` 帮助你实现“稍后购买”功能。`buy_now` 字段是布尔类型，默认值为 `true`。当客户进行结账时，只有该值设置为 `true` 的产品才会被添加到订单中，而标记为“稍后购买”的产品则会保留在购物车中。这个功能很有用，因为它允许访客在购物车中保留比他或她当前能负担得起（或想要购买）的更多产品，并且允许访客只订购购物车中的部分产品。
- `added_on` 会在新产品添加到购物车时填入当前日期，并且在从数据库中删除旧购物车时非常有用。

> **注意** `shopping_cart` 表有一个由 `cart_id` 和 `product_id` 字段组成的复合主键。这是合理的，因为某个特定产品在特定购物车中只能存在一次，所以一个 `(cart_id, product_id)` 对在表中不应出现多次。

## 实现数据层

在本节中，你将创建用于查询购物车操作的常规函数。在进一步编写代码之前，让我们回顾一下你将添加到 `hatshop` 数据库中的函数：

- `shopping_cart_add_product` 将产品添加到购物车。
- `shopping_cart_update` 修改购物车中产品的数量。
- `shopping_cart_remove_product` 从访客的购物车中删除一条记录。
- `shopping_cart_get_products` 获取指定购物车中的产品列表，当你想向用户展示其购物车时调用。
- `shopping_cart_get_saved_products` 获取购物车中保存以备稍后购买的产品列表，当用户请求查看购物车详情页面时调用。
- `shopping_cart_get_total_amount` 返回指定产品购物车中产品的总价。
- `shopping_cart_save_product_for_later` 将产品保存到购物车以供稍后购买。
- `shopping_cart_move_product_to_cart` 将产品从“稍后购买”列表移回“主”购物车。

现在，在下面的练习中，我们逐一创建每个方法。

### 练习：实现函数

**1.** 加载 pgAdmin III，并连接到 `hatshop` 数据库。

**2.** 点击“工具”➤“查询工具”（或点击工具栏上的 SQL 按钮）。此时应出现一个新的查询窗口。

**3.** 使用查询工具执行以下代码，该代码将在你的 `hatshop` 数据库中创建 `shopping_cart_add_product` 函数：

```sql
-- 创建 shopping_cart_add_product 函数

CREATE FUNCTION shopping_cart_add_product(CHAR(32), INTEGER) RETURNS VOID
LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  inProductId ALIAS FOR $2;
  productQuantity INTEGER;
BEGIN
  SELECT INTO productQuantity
    quantity
  FROM shopping_cart
  WHERE cart_id = inCartId AND product_id = inProductId;
  IF productQuantity IS NULL THEN
    INSERT INTO shopping_cart(cart_id, product_id, quantity, added_on)
    VALUES (inCartId, inProductId, 1, NOW());
  ELSE
    UPDATE shopping_cart
    SET quantity = quantity + 1, buy_now = true
    WHERE cart_id = inCartId AND product_id = inProductId;
  END IF;
END;
$$;
```

当访客点击某个产品的“添加到购物车”按钮时，会调用 `shopping_cart_add_product` 函数。如果选定的产品已存在于购物车中，则其数量增加一；如果该产品不存在，则向购物车中添加一个单位（创建一条新的 `shopping_cart` 记录）。

不出所料，`shopping_cart_add_product` 接收两个参数，即 `inCartId` 和 `inProductId`。

该函数首先判断 `inProductId` 所指定的产品是否存在于 `inCartId` 所指的购物车中。它通过测试 `(inCartId, inProductId)` 对是否存在于 `shopping_cart` 表中来实现这一点。如果产品已在购物车中，`shopping_cart_add_product` 会通过增加一个单位来更新当前产品数量。否则，`shopping_cart_add_product` 会在 `shopping_cart` 中为该产品创建一个新记录，默认数量为 1，但在此之前会检查所提到的 `inProductId` 是否有效。

`NOW()` PostgreSQL 函数用于获取当前日期，并手动填充 `added_on` 字段。

**4.** 使用查询工具执行以下代码，该代码将在你的 `hatshop` 数据库中创建 `shopping_cart_update` 函数：

```sql
-- 创建 shopping_cart_update 函数

CREATE FUNCTION shopping_cart_update(CHAR(32), INTEGER[], INTEGER[])
RETURNS VOID
LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  inProductIds ALIAS FOR $2;
  inQuantities ALIAS FOR $3;
BEGIN
  FOR i IN array_lower(inQuantities, 1)..array_upper(inQuantities, 1) LOOP
    IF inQuantities[i] > 0 THEN
      UPDATE shopping_cart
      SET quantity = inQuantities[i], added_on = NOW()
```


```sql
WHERE cart_id = inCartId AND product_id = inProductIds[i]; ELSE
PERFORM shopping_cart_remove_product(inCartId, inProductIds[i]); END IF;
END LOOP;
END;
$$;
```

当你想要更新一个或多个现有购物车商品的数量时，可以使用 `shopping_cart_update` 函数。当访问者点击“更新”按钮时，会调用此函数。

`shopping_cart_update` 接收两个数组值作为参数：`inProductIds` 和 `inQuantities`。

`inQuantities[i]` 的值表示 `inProductIDs[i]` 所指定商品的新数量。

如果 `inQuantities[i]` 为零或负数，`shopping_cart_update` 会从购物车中移除该商品。否则，它会更新购物车中该商品的数量，并同时更新 `added_on` 字段，以准确反映记录上次被修改的时间。

更新 `added_on` 字段对于管理页面特别有用，因为您需要移除长时间未更新的购物车。

**5.** 使用查询工具执行以下代码，以便在您的 hatshop 数据库中创建 `shopping_cart_remove_product` 函数：

```sql
-- 创建 shopping_cart_remove_product 函数
CREATE FUNCTION shopping_cart_remove_product(CHAR(32), INTEGER)
RETURNS VOID LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  inProductId ALIAS FOR $2;
BEGIN
  DELETE FROM shopping_cart
  WHERE cart_id = inCartId AND product_id = inProductId;
END;
$$;
```

当访问者点击购物车中某个商品的“移除”按钮时，`shopping_cart_remove_product` 函数会将该商品从购物车中移除。

**6.** 使用查询工具执行以下代码，以便在您的 hatshop 数据库中创建 `cart_product` 类型和 `shopping_cart_get_products` 函数：

```sql
-- 创建 cart_product 类型
CREATE TYPE cart_product AS
(
  product_id INTEGER,
  name VARCHAR(50),
  price NUMERIC(10, 2),
  quantity INTEGER,
  subtotal NUMERIC(10, 2)
);

-- 创建 shopping_cart_get_products 函数
CREATE FUNCTION shopping_cart_get_products(CHAR(32))
RETURNS SETOF cart_product LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  outCartProductRow cart_product;
BEGIN
  FOR outCartProductRow IN
    SELECT p.product_id, p.name,
           COALESCE(NULLIF(p.discounted_price, 0), p.price) AS price, sc.quantity,
           COALESCE(NULLIF(p.discounted_price, 0), p.price) * sc.quantity AS subtotal
    FROM shopping_cart sc
    INNER JOIN product p
    ON sc.product_id = p.product_id
    WHERE sc.cart_id = inCartId AND buy_now
  LOOP
    RETURN NEXT outCartProductRow;
  END LOOP;
END;
$$;
```

`shopping_cart_get_products` 函数返回由 `inCartId` 参数指定的购物车中的商品。由于 `shopping_cart` 表仅存储每个商品的 `product_id`，因此您需要连接 `shopping_cart` 和 `product` 表来获取所需信息。

请注意，部分商品可能有折扣价格。当商品有折扣价格时（即其 `discounted_price` 值不为 0），应使用折扣价格进行计算。否则，应使用其标价。以下表达式在 `discounted_price` 不为 0 时返回该值；否则返回 `price`：

```sql
COALESCE(NULLIF(p.discounted_price, 0), p.price)
```

> **注意：** 这是您第一次使用 `COALESCE` 和 `NULLIF` 这两个 PostgreSQL 条件表达式，因此我们来了解一下它们的作用。`COALESCE` 可以接受任意数量的参数，并返回第一个不为 `NULL` 的参数。`NULLIF` 接受两个参数，如果它们相等则返回 `NULL`；否则返回第一个参数。在我们的示例中，我们使用 `NULLIF` 来测试 `discounted_price` 是否为 0；如果条件成立，`NULLIF` 返回 `NULL`，那么 `COALESCE` 函数将返回 `p.price`。如果 `discounted_price` 不为 0，则整个表达式返回 `discounted_price`。

**7.** 使用查询工具执行以下代码，以便在您的 hatshop 数据库中创建 `cart_saved_product` 类型和 `shopping_cart_get_saved_products` 函数：

```sql
-- 创建 cart_saved_product 类型
CREATE TYPE cart_saved_product AS
(
  product_id INTEGER,
  name VARCHAR(50),
  price NUMERIC(10, 2)
);

-- 创建 shopping_cart_get_saved_products 函数
CREATE FUNCTION shopping_cart_get_saved_products(CHAR(32)) RETURNS SETOF cart_saved_product LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  outCartSavedProductRow cart_saved_product;
BEGIN
  FOR outCartSavedProductRow IN
    SELECT p.product_id, p.name,
           COALESCE(NULLIF(p.discounted_price, 0), p.price) AS price
    FROM shopping_cart sc
    INNER JOIN product p
    ON sc.product_id = p.product_id
    WHERE sc.cart_id = inCartId AND NOT buy_now
  LOOP
    RETURN NEXT outCartSavedProductRow;
  END LOOP;
END;
$$;
```

`shopping_cart_get_saved_products` 函数返回由 `inCartId` 参数指定的购物车中保存以供稍后购买的商品。

**8.** 使用查询工具执行以下代码，以便在您的 hatshop 数据库中创建 `shopping_cart_get_total_amount` 函数：

```sql
-- 创建 shopping_cart_get_total_amount 函数
CREATE FUNCTION shopping_cart_get_total_amount(CHAR(32)) RETURNS NUMERIC(10, 2) LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  outTotalAmount NUMERIC(10, 2);
BEGIN
  SELECT INTO outTotalAmount
         SUM(COALESCE(NULLIF(p.discounted_price, 0), p.price) * sc.quantity)
  FROM shopping_cart sc
  INNER JOIN product p
  ON sc.product_id = p.product_id
  WHERE sc.cart_id = inCartId AND sc.buy_now;
  RETURN outTotalAmount;
END;
$$;
```

`shopping_cart_get_total_amount` 函数返回购物车中商品的总价值。在显示购物车总金额时会调用此函数。如果购物车为空，`total_amount` 将为 0。

**9.** 使用查询工具执行以下代码，以便在您的 hatshop 数据库中创建 `shopping_cart_save_product_for_later` 函数：

```sql
-- 创建 shopping_cart_save_product_for_later 函数
CREATE FUNCTION shopping_cart_save_product_for_later(CHAR(32), INTEGER) RETURNS VOID LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  inProductId ALIAS FOR $2;
BEGIN
  UPDATE shopping_cart
  SET buy_now = false, quantity = 1
  WHERE cart_id = inCartId AND product_id = inProductId;
END;
$$;
```

`shopping_cart_save_product_for_later` 函数将购物车中的商品保存到“稍后购买”列表中，以便访问者日后购买（在下单时该商品不会进入结算流程）。这是通过将 `buy_now` 字段的值设置为 `false` 来实现的。

**10.** 使用查询工具执行以下代码，以便在您的 hatshop 数据库中创建 `shopping_cart_move_product_to_cart` 函数：

```sql
-- 创建 shopping_cart_move_product_to_cart 函数
CREATE FUNCTION shopping_cart_move_product_to_cart(CHAR(32), INTEGER) RETURNS VOID LANGUAGE plpgsql AS $$
DECLARE
  inCartId ALIAS FOR $1;
  inProductId ALIAS FOR $2;
BEGIN
  UPDATE shopping_cart
  SET buy_now = true, added_on = NOW()
  WHERE cart_id = inCartId AND product_id = inProductId;
END;
$$;
```

`shopping_cart_move_product_to_cart` 函数将商品的 `buy_now` 状态设置为 `true`，以便访问者在下单时购买该商品。


