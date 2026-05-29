# 搜索引擎目录

请注意，向量保留了单词的位置（尽管我们并不真正需要这一点），并且“A”相关性因子被添加到来自产品名称的单词中。同时要注意，同一单词的各种形式都会被识别（例如“fit”），而停用词不会被考虑。

**表 5-1.** *产品的搜索向量*

| **字段** | **值** |
| --- | --- |
| `product_id` | |
| `name` | Santa Jester Hat |
| `description` | This three-prong velvet jester is one size fits all and has an adjustable touch fastener back for perfect fitting. |
| `search_string` | `'fit':13,24 'hat':3A 'one':11 'back':21 'size':12 'prong':7 'santa':1A 'three':6 'touch':19 'adjust':18 'fasten':20 'jester':2A,9 'velvet':8 'perfect':23 'three-prong':5` |

■**注意**  当向表中添加新产品或更新现有产品时，需要确保同时（重新）创建它们的搜索向量。负责解析这些向量的索引会自动完成其工作，但向量本身必须手动创建。你将在第 7 章中处理这个问题，届时将添加目录管理功能。在此之前，如果你手动更改了产品，只需执行之前的 SQL 命令来更新搜索向量。

## 搜索数据库

现在你已经为每个产品构建了搜索向量，让我们看看如何用它进行搜索。

要执行搜索，同样需要三个步骤：

1. **构建一个*搜索字符串***，表达你究竟在查找什么。它可以包含布尔运算符；我们在全词搜索时使用 `&`（AND），在任意词搜索时使用 `|`（OR）。

2. **对查询字符串应用 `to_tsquery` 函数**。这将把查询字符串准备成可用于搜索的形式。

3. **执行搜索时**，使用条件 `search_vector @@ prepared_search_string`，如果匹配则返回 `TRUE`，否则返回 `FALSE`。这里，`search_vector` 是之前计算的结果（上一部分的步骤 3），而 `prepared_search_string` 是步骤 2 的结果。

让我们看看这在实践中如何应用。以下查询对“yankee war”搜索字符串执行全词搜索：

```sql
SELECT product_id, name
FROM product
WHERE search_vector @@ to_tsquery('yankee & war')
ORDER BY product_id;
```

对于示例产品数据库，此查询应得到表 5-2 所示的结果。

**表 5-2.** *匹配“yankee & war”的帽子*

| `product_id` | `name` |
| --- | --- |
| | Civil War Union Slouch Hat |
| | Union Civil War Kepi Cap |

要执行任意词搜索，应在搜索字符串中使用 `|` 代替 `&`：

```sql
SELECT product_id, name
FROM product
WHERE search_vector @@ to_tsquery('yankee | war')
ORDER BY product_id;
```

如预期，这次将会有更多匹配的产品，如表 5-3 所示（因为列表未排序，结果顺序可能不同）。

**表 5-3.** *匹配“yankee | war”的帽子*

| `product_id` | `name` |
| --- | --- |
| | Military Beret |
| | Confederate Civil War Kepi |
| | Uncle Sam Top Hat |
| | Confederate Slouch Hat |
| | Civil War Union Slouch Hat |
| | Civil War Leather Kepi Cap |
| | Union Civil War Kepi Cap |

## 按相关性排序结果

前面的查询显示了匹配的产品，但未按特定顺序排序。数据库引擎会以其认为简单的方式返回结果。对于搜索，我们更希望首先显示相关性更高的匹配项。请记住，我们为来自产品标题的匹配赋予了更高优先级，因此一切已准备就绪。

`tsearch2` 引擎提供了 `rank` 函数，可用于对结果进行排序。默认顺序是先显示排名较低的匹配项，因此还需要使用 `ORDER BY` 的 `DESC` 选项将更好的匹配项置于顶部。

```sql
-- 以下查询对“yankee war”执行了带排名的任意词搜索：
SELECT rank(search_vector, to_tsquery('yankee | war')) as rank, product_id, name FROM product
WHERE search_vector @@ to_tsquery('yankee | war')
ORDER BY rank DESC;
```

这次，结果将按顺序排列。您还可以看到搜索排名。在`name`字段中有匹配项的产品的排名明显更高。

**表 5-4.** *按排名排序的搜索结果*

| 排名      | product_id | 名称                           |
|----------|------------|--------------------------------|
| 0.341959 |            | 内战联邦宽松帽                 |
| 0.33436  |            | 联邦内战凯皮帽                 |
| 0.31684  |            | 内战皮革凯皮帽                 |
| 0.303964 |            | 邦联内战凯皮帽                 |
| 0.0379954|            | 军用贝雷帽                     |
| 0.0303964|            | 邦联宽松帽                     |
| 0.0303964|            | 山姆大叔高顶礼帽               |

现在，您应该已经准备好实现网站的搜索功能了。要了解有关`tsearch2`引擎内部工作原理的更多信息，请查阅其官方文档。

## 练习：编写数据库搜索代码

1.  加载 `pgAdmin III`，并连接到 `hatshop` 数据库。

2.  点击工具 ➤ 查询工具（或点击工具栏上的 SQL 按钮）。一个新的查询窗口应该会出现。

3.  在查询工具中编写以下代码，然后按 `F5` 键执行它。此命令将 `product` 表配置为使用 `tsearch2` 引擎进行搜索，正如本章前面所述。

    ```sql
    -- 修改 product 表，添加 search_vector 字段
    ALTER TABLE product ADD COLUMN search_vector tsvector;

    -- 为 product 表中的 search_vector 字段创建索引
    CREATE INDEX idx_search_vector ON product USING gist(search_vector);

    -- 更新 product 表中新添加的 search_vector 字段
    UPDATE product
    SET search_vector =
    setweight(to_tsvector(name), 'A') || to_tsvector(description);
    ```

    [www.it-ebooks.info](http://www.it-ebooks.info/)

    648XCH05.qxd 10/31/06 10:04 PM 第 176 页

    **176**

    第 5 章 ■ 搜索产品目录

4.  使用查询工具执行以下代码，该代码将在您的 `hatshop` 数据库中创建 `catalog_flag_stop_words` 函数：

## 创建数据库函数

### 1. 创建 `catalog_flag_stop_words` 函数

```sql
-- 创建 catalog_flag_stop_words 函数
CREATE FUNCTION catalog_flag_stop_words(TEXT[])
RETURNS SETOF SMALLINT LANGUAGE plpgsql AS $$
DECLARE
    inWords ALIAS FOR $1;
    outFlag SMALLINT;
    query TEXT;
BEGIN
    FOR i IN array_lower(inWords, 1)..array_upper(inWords, 1) LOOP
        SELECT INTO query
            to_tsquery(inWords[i]);
        IF query = '' THEN
            outFlag := 1;
        ELSE
            outFlag := 0;
        END IF;
        RETURN NEXT outFlag;
    END LOOP;
END;
$$;
```

### 5. 创建 `catalog_count_search_result` 函数

使用查询工具执行以下代码，该代码将在您的 `hatshop` 数据库中创建 `catalog_count_search_result` 函数：

```sql
-- 函数返回与搜索字符串匹配的产品数量
CREATE FUNCTION catalog_count_search_result(TEXT[], VARCHAR(3))
RETURNS INTEGER LANGUAGE plpgsql AS $$
DECLARE
    -- inWords 是一个数组，包含用户搜索字符串中的单词
    inWords ALIAS FOR $1;
    -- inAllWords 对于“全部词”搜索为 'on'
    -- 对于“任意词”搜索为 'off'
    inAllWords ALIAS FOR $2;
    outSearchResultCount INTEGER;
    query TEXT;
    search_operator VARCHAR(1);
BEGIN
    -- 将查询初始化为空字符串
    query := '';
    -- 确定在准备搜索字符串时要使用的运算符
    IF inAllWords = 'on' THEN
        search_operator := '&';
    ELSE
        search_operator := '|';
    END IF;
    -- 组合搜索字符串
    FOR i IN array_lower(inWords, 1)..array_upper(inWords, 1) LOOP
        IF i = array_upper(inWords, 1) THEN
            query := query || inWords[i];
        ELSE
            query := query || inWords[i] || search_operator;
        END IF;
    END LOOP;
    -- 返回匹配的数量
    SELECT INTO outSearchResultCount
        count(*)
    FROM product,
        to_tsquery(query) AS query_string
    WHERE search_vector @@ query_string;
    RETURN outSearchResultCount;
END;
$$;
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH05.qxd 10/31/06 10:04 PM 第 177 页

**177**

第 5 章 ■ 搜索产品目录

### 6. 创建 `catalog_search` 函数

使用查询工具执行以下代码，该代码将在您的 `hatshop` 数据库中创建 `catalog_search` 函数：

```sql
-- 创建 catalog_search 函数
CREATE FUNCTION catalog_search(TEXT[], VARCHAR(3), INTEGER, INTEGER, INTEGER)
RETURNS SETOF product_list LANGUAGE plpgsql AS $$
DECLARE
    inWords ALIAS FOR $1;
    inAllWords ALIAS FOR $2;
    inShortProductDescriptionLength ALIAS FOR $3;
    inProductsPerPage ALIAS FOR $4;
    inStartPage ALIAS FOR $5;
    outProductListRow product_list;
    query TEXT;
    search_operator VARCHAR(1);
    query_string TSQUERY;

BEGIN
    -- 用空字符串初始化查询
    query := '';

    -- 匹配所有词还是任一词？
    IF inAllWords = 'on' THEN
        search_operator := '&';
    ELSE
        search_operator := '|';
    END IF;

    -- 组合搜索字符串
    FOR i IN array_lower(inWords, 1)..array_upper(inWords, 1) LOOP
        IF i = array_upper(inWords, 1) THEN
            query := query||inWords[i];
        ELSE
            query := query||inWords[i]||search_operator;
        END IF;
    END LOOP;

    query_string := to_tsquery(query);

    -- 返回搜索结果
    FOR outProductListRow IN
        SELECT product_id, name, description, price,
               discounted_price, thumbnail
        FROM product
        WHERE search_vector @@ query_string
        ORDER BY rank(search_vector, query_string) DESC
        LIMIT inProductsPerPage
        OFFSET inStartPage
    LOOP
        IF char_length(outProductListRow.description) >
           inShortProductDescriptionLength THEN
            outProductListRow.description :=
                substring(outProductListRow.description, 1,
                          inShortProductDescriptionLength) || '...';
        END IF;
        RETURN NEXT outProductListRow;
    END LOOP;
END;
$$;
```

## 工作原理解析：目录搜索功能

在本练习中，您创建了支持产品搜索业务层逻辑的数据库功能。在添加了本章开头说明的必要结构后，您新增了三个函数：

- `catalog_flag_stop_words`：如前所述，访问者输入的搜索字符串中的某些词可能不会用于搜索，因为它们被视为噪音词。默认情况下，`tsearch2` 引擎会移除这些噪音词，但我们需要找出被移除的词，以便将这些词告知访问者。我们通过 `catalog_flag_stop_words` 函数实现此目的，该函数将由业务层的 `FlagStopWords` 方法调用。

- `catalog_count_search_result`：此函数用于统计搜索结果的数量。表示层需要知道要显示多少个搜索结果页面。

- `catalog_search`：执行实际的产品搜索。

## 实现业务层

搜索功能的业务层包含两个方法：`FlagStopWords` 和 `Search`。我们先实现它们，然后讨论其工作原理。

### 练习：实现业务层

1. 全文搜索功能会自动移除短于指定长度的词。进行搜索时，您需要告知访问者哪些词已被移除。首先，通过 `FlagStopWords` 方法找出哪些词被移除。该方法接收一个词数组作为参数，并返回两个数组：一个用于停用词，另一个用于可接受词。将此方法添加到 `business/catalog.php` 中的 `Catalog` 类：

```php
// 在搜索查询中标记停用词
public static function FlagStopWords($words)
{
  // 构建 SQL 查询
  $sql = 'SELECT *
          FROM catalog_flag_stop_words(:words);';

  // 构建参数数组
  $params = array (':words' => '{' . implode(', ', $words) . '}');

  // 使用 PDO 特定功能准备语句
  $result = DatabaseHandler::Prepare($sql);

  // 执行查询
  $flags = DatabaseHandler::GetAll($result, $params);

  $search_words = array ('accepted_words' => array (),
                         'ignored_words' => array ());

  for ($i = 0; $i < count($flags); $i++)
    if ($flags[$i]['catalog_flag_stop_words'])
      $search_words['ignored_words'][] = $words[$i];
    else
      $search_words['accepted_words'][] = $words[$i];

  return $search_words;
}
```

2. 最后，在 `Catalog` 类中添加 `Search` 方法：

```php
// 搜索目录
public static function Search($searchString, $allWords, $pageNo, &$rHowManyPages)
{
  // 搜索结果将按以下格式的数组返回
  $search_result = array (
      'accepted_words' => array (),
      'ignored_words' => array (),
      'products' => array ()
  );

  // 如果搜索字符串为空，则返回空结果
  if (empty ($searchString))
      return $search_result;

  // 搜索字符串分隔符
  $delimiters = ',.; ';

  // 使用 strtok 获取搜索字符串的第一个单词
  $word = strtok($searchString, $delimiters);
  $words = array ();

  // 构建单词数组
  while ($word)
  {
      $words[] = $word;
      // 获取搜索字符串的下一个单词
      $word = strtok($delimiters);
  }

  // 将搜索单词分为两类：接受和忽略
  $search_words = Catalog::FlagStopWords($words);
  $search_result['accepted_words'] = $search_words['accepted_words'];
  $search_result['ignored_words'] = $search_words['ignored_words'];

  // 如果所有单词都是停用词，则返回空结果
  if (count($search_result['accepted_words']) == 0)
      return $search_result;

  // 统计搜索结果数量
  $sql = 'SELECT catalog_count_search_result(:words, :all_words);';
  $params = array (
      ':words' => '{' . implode(', ', $search_result['accepted_words']) . '}',
      ':all_words' => $allWords
  );

  // 计算显示产品所需的页数
  $rHowManyPages = Catalog::HowManyPages($sql, $params);

  // 计算起始条目
  $start_item = ($pageNo - 1) * PRODUCTS_PER_PAGE;

  // 检索匹配的产品列表
  $sql = 'SELECT *
          FROM catalog_search(:words,
                              :all_words,
                              :short_product_description_length,
                              :products_per_page,
                              :start_page);';
  $params = array (
      ':words' => '{' . implode(', ', $search_result['accepted_words']) . '}',
      ':all_words' => $allWords,
      ':short_product_description_length' => SHORT_PRODUCT_DESCRIPTION_LENGTH,
      ':products_per_page' => PRODUCTS_PER_PAGE,
      ':start_page' => $start_item
  );

  // 准备并执行查询，然后返回结果
  $result = DatabaseHandler::Prepare($sql);
  $search_result['products'] = DatabaseHandler::GetAll($result, $params);
  return $search_result;
}
```

## 工作原理：业务层的搜索方法

`FlagStopWords` 方法的主要目的是分析哪些单词将用于搜索，哪些不会。

PostgreSQL 的全文搜索功能默认会自动过滤少于四个字母的单词，你在业务层中无需干预此行为。不过，你需要找出哪些单词会被 PostgreSQL 忽略，以便告知访客。

业务层的 `Search` 方法由表示层调用，并带有以下参数（注意除第一个参数外，其余参数与数据层 `Search` 方法的参数相同）：

- `$searchString` 包含访客输入的搜索字符串。
- `$allWords` 为 “on” 时表示全词搜索。
- `$pageNo` 表示所请求的产品页数。
- `$rHowManyPages` 表示总页数。

该方法以关联数组的形式将结果返回给表示层。

## 实现表示层

目录搜索功能包含两个需要实现的独立界面元素：

- 一个名为 `search_box` 的组件化模板，其作用是为访客提供输入搜索字符串的途径（请参考图 5-1）。
- 一个名为 `search_results` 的组件化模板，用于显示与搜索条件匹配的产品（请参考图 5-2）。

你将通过两个独立的练习来创建这两个组件化模板。

### 创建搜索框

按照练习中的步骤构建 `search_box` 组件化模板，并将其集成到 HatShop 中。