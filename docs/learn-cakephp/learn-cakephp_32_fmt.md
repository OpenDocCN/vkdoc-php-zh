# 设置请求数据

大多数情况下，我们需要控制器中的数据来处理来自`get`、`post`、`cookie`或`session`的内容。

```php
public function testAdd()
{
    $data = [
        'category_id' => 2,
        'user_id' => 1,
        'title' => 'Test Post Title',
        'body' => 'Test post body with same sample text',
        'created' => '2016-05-01 14:00:00',
        'modified' => '2016-05-01 14:00:00',
        'tags' => [
            ['id' => 1],
            ['id' => 2],
        ]
    ];
    $this->post('/posts/add/', $data);

    $this->assertResponseSuccess();
```

首先，我们创建一个数据数组以 POST 方式发送到控制器。然后发送数据并检查是否收到成功响应状态码。

```php
$posts = TableRegistry::get('Posts');
$query = $posts->find()->where(['title' => $data['title']]);
$this->assertEquals(1, $query->count());
```

在上一章中，我们创建了帖子的固定数据集。因此，我们知道只有这个新帖子的标题与数据数组中的标题相同，所以计数应返回 1。

```php
$result = $query->toArray();
$poststags = TableRegistry::get('PostsTags');
$query = $poststags->find()->where(['post_id' => $result[0]->id]);
$result = $query->toArray();
$this->assertEquals(1, $result[0]->tag_id);
$this->assertEquals(2, $result[1]->tag_id);
}
```

在最后一个断言中，我们检查这个新帖子是否包含两个标签：1 和 2。（见图 9-1。）

![A337704_2_En_9_Fig1_HTML.jpg](img/A337704_2_En_9_Fig1_HTML.jpg)

图 9-1. `testAdd` 的结果

测试 `GET` 数据非常简单。只需将查询字符串添加到 `get` 调用的第一个参数中即可。（见图 9-2。）

![A337704_2_En_9_Fig2_HTML.jpg](img/A337704_2_En_9_Fig2_HTML.jpg)

图 9-2. `testEdit` 的结果

```php
public function testEdit()
{
    $this->get('/posts/edit/1');
    $this->assertResponseOk();
}
```

## 小节

本章概述了由 `bake` 生成的控制器。我们探讨了 `bake` 的工作原理及其局限性。随后创建了控制器测试，并引入了集成测试和控制器特定的断言方法。