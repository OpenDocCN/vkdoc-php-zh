# 实现业务层

要实现业务层，您需要向`Catalog`类添加以下方法：

- `DeleteProduct`：从目录中彻底删除一个产品。
- `RemoveProductFromCategory`：当点击“从分类中移除”按钮时调用，用于将产品从分类中取消分配。
- `GetCategories`：返回目录中的所有分类。
- `GetProductInfo`：返回产品的详细信息。
- `GetCategoriesForProduct`：用于获取与指定产品相关的分类列表。
- `SetProductDisplayOption`：设置产品的显示选项。
- `AssignProductToCategory`：将产品分配到一个分类。
- `MoveProductToCategory`：将产品从一个分类移动到另一个分类。
- `SetImage`：更改数据库中特定产品的图片文件名。
- `SetThumbnail`：更改特定产品的第二个图片文件名。

## 练习：实现业务层方法

由于这些功能通过数据层方法调用能更好地表达，我们将在实现数据层时进一步讨论它们。请将以下代码添加到`business/catalog.php`中的`Catalog`类：

```php
// Removes a product from the product catalog
public static function DeleteProduct($productId)
{
    // Build the SQL query
    $sql = 'SELECT catalog_delete_product(:product_id);';

    // Build the parameters array
    $params = array (':product_id' => $productId);

    // Prepare the statement with PDO-specific functionality
    $result = DatabaseHandler::Prepare($sql);

    // Execute the query
    return DatabaseHandler::Execute($result, $params);
}

// Unassigns a product from a category
public static function RemoveProductFromCategory($productId, $categoryId)
{
    // Build the SQL query
    $sql = 'SELECT catalog_remove_product_from_category(
        :product_id, :category_id);';

    // Build the parameters array
    $params = array (':product_id' => $productId,
                     ':category_id' => $categoryId);

    // Prepare the statement with PDO-specific functionality
    $result = DatabaseHandler::Prepare($sql);

    // Execute the query and return the results
    return DatabaseHandler::GetOne($result, $params);
}

// Retrieves the list of categories a product belongs to
public static function GetCategories()
{
    // Build the SQL query
    $sql = 'SELECT * FROM catalog_get_categories();';

    // Prepare the statement with PDO-specific functionality
    $result = DatabaseHandler::Prepare($sql);

    // Execute the query and return the results
    return DatabaseHandler::GetAll($result);
}

// Retrieves product info
public static function GetProductInfo($productId)
{
    // Build the SQL query
    $sql = 'SELECT * FROM catalog_get_product_info(:product_id);';

    // Build the parameters array
    $params = array (':product_id' => $productId);

    // Prepare the statement with PDO-specific functionality
    $result = DatabaseHandler::Prepare($sql);

    // Execute the query and return the results
    return DatabaseHandler::GetRow($result, $params);
}

// Retrieves the list of categories a product belongs to
public static function GetCategoriesForProduct($productId)
{
    // Build the SQL query
    $sql = 'SELECT * FROM catalog_get_categories_for_product(:product_id);';

    // Build the parameters array
    $params = array (':product_id' => $productId);

    // Prepare the statement with PDO-specific functionality
    $result = DatabaseHandler::Prepare($sql);

    // Execute the query and return the results
    return DatabaseHandler::GetAll($result, $params);
}

// Assigns a product to a category
public static function SetProductDisplayOption($productId, $display)
{
    // Build the SQL query
    $sql = 'SELECT catalog_set_product_display_option(
        :product_id, :display);';

    // Build the parameters array
    $params = array (':product_id' => $productId,
                     ':display' => $display);

    // Prepare the statement with PDO-specific functionality
    $result = DatabaseHandler::Prepare($sql);
}
```

```markdown
// 执行查询

return `DatabaseHandler::Execute($result, $params)`;

}

// 将产品分配给一个类别

public static function `AssignProductToCategory($productId, $categoryId)`

{

// 构建 SQL 查询语句

$sql = 'SELECT catalog_assign_product_to_category(:product_id, :category_id);';

// 构建参数数组

$params = array (':product_id' => $productId, ':category_id' => $categoryId);

// 使用 PDO 特定功能准备语句 `$result = DatabaseHandler::Prepare($sql)`;

// 执行查询

return `DatabaseHandler::Execute($result, $params)`;

}

// 将产品从一个类别移动到另一个类别

public static function `MoveProductToCategory($productId, $sourceCategoryId, $targetCategoryId)`

{

// 构建 SQL 查询语句

$sql = 'SELECT catalog_move_product_to_category(:product_id, :source_category_id, :target_category_id);';

// 构建参数数组

$params = array (':product_id' => $productId, ':source_category_id' => $sourceCategoryId, ':target_category_id' => $targetCategoryId);

// 使用 PDO 特定功能准备语句 `$result = DatabaseHandler::Prepare($sql)`;

// 执行查询

return `DatabaseHandler::Execute($result, $params)`;

}

// 更改数据库中产品图片文件的名称

public static function `SetImage($productId, $imageName)`

{

// 构建 SQL 查询语句

$sql = 'SELECT catalog_set_image(:product_id, :image_name);';

// 构建参数数组

$params = array (':product_id' => $productId, ':image_name' => $imageName);

// 使用 PDO 特定功能准备语句 `$result = DatabaseHandler::Prepare($sql)`;

// 执行查询

return `DatabaseHandler::Execute($result, $params)`;

}

// 更改数据库中产品缩略图文件的名称

public static function `SetThumbnail($productId, $thumbnailName)`

{

// 构建 SQL 查询语句

$sql = 'SELECT catalog_set_thumbnail(:product_id, :thumbnail_name);';

// 构建参数数组

$params = array (':product_id' => $productId, ':thumbnail_name' => $thumbnailName);

// 使用 PDO 特定功能准备语句 `$result = DatabaseHandler::Prepare($sql)`;

// 执行查询

return `DatabaseHandler::Execute($result, $params)`;

}

## 实现数据层

在数据层中，您需要在 `Catalog` 类中添加与您刚才看到的业务层方法相对应的方法。

## 练习：添加数据层函数

**1.** 加载 `pgAdmin III`，并连接到 `hatshop` 数据库。

**2.** 点击**工具** ➤ **查询工具**（或点击工具栏上的 SQL 按钮）。此时会弹出一个新的查询窗口。

**3.** 使用查询工具执行以下代码，该代码将在您的 `hatshop` 数据库中创建 `catalog_delete_product` 函数：

```sql
-- 创建 catalog_delete_product 函数
CREATE FUNCTION catalog_delete_product(INTEGER)
RETURNS VOID LANGUAGE plpgsql AS $$
DECLARE
inProductId ALIAS FOR $1;
BEGIN
DELETE FROM product_category WHERE product_id = inProductId;
DELETE FROM product WHERE product_id = inProductId;
END;
$$;
```

`catalog_delete_product` 函数通过删除 `product_category` 和 `product` 表中的相关条目，将产品从目录中完全移除。

**4.** 使用查询工具执行以下代码，该代码将在您的 `hatshop` 数据库中创建 `catalog_remove_product_from_category` 函数：

```sql
-- 创建 catalog_remove_product_from_category 函数
CREATE FUNCTION catalog_remove_product_from_category(INTEGER, INTEGER)
RETURNS SMALLINT LANGUAGE plpgsql AS $$
DECLARE
inProductId ALIAS FOR $1;
inCategoryId ALIAS FOR $2;
productCategoryRowsCount INTEGER;
BEGIN
SELECT INTO productCategoryRowsCount
count(*)
FROM product_category
WHERE product_id = inProductId;
IF productCategoryRowsCount = 1 THEN
PERFORM catalog_delete_product(inProductId);
RETURN 0;
END IF;
DELETE FROM product_category
```