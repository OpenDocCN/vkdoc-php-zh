# Bake 背后的魔法

CakePHP 团队运用了大量巧妙的编程工具，才打造出像 `bake` 这样便捷的工具。如前面的示例所示，它确实能为我们编写代码。

当我们烘焙模型时，它会检测数据库字段类型，并据此创建实体类和模型类。实体对象代表数据库中的一行，而模型对象则代表数据库表，包含其关联关系、验证规则和方法。由于模型继承了 CakePHP 的 `Table` 类，它们也继承了许多有用的方法。这就是为什么我们可以直接使用像 `get`、`save` 和 `delete` 这样的方法，而无需自己创建它们。这些方法是由框架本身提供的。

当我们烘焙控制器时，会生成 `index`、`edit`、`view` 和 `delete` 这些最常用的操作方法。这些方法会使用到相关的模型。

烘焙还会为所有控制器方法生成视图。例如，对于 `index` 方法，就会生成对应的 index 视图。在视图中，我们能够访问那些在控制器中设置的变量。

正如你所见，通过继承和 `bake`，我们直接从“烤箱”里得到了很多东西。不过，有一点你需要牢记在心：由 `bake` 生成的代码只是为了让你尽快得到一个可运行的基础骨架。它不会处理权限或用户认证问题，因此任何人都可以创建和删除东西。CakePHP 对这些功能也提供了良好的支持，但它们并不会由 `bake` 自动生成。

## 创建控制器测试

在概述了生成的控制器之后，我们来为它创建一些测试。首先，我们必须为分类（categories）、标签（tags）和文章（posts）模型创建测试固件（fixtures）。我们从分类开始。第 7 章已经提供了固件的概述，所以下面的代码示例应该很容易理解。

```
1   'Categories'];

11      public $records = [
12          [
13              'id' => 1,
14              'category' => 'Category 1'
15          ],
16          [
17              'id' => 2,
18              'category' => 'Category 2'
19          ],
20      ];
21  }
```

我们将 `Categories` 模型导入到固件中，并创建了两条分类记录。这段代码应该放在 `/tests/Fixture/CategoriesFixture.php` 文件中。该文件是由 `bake` 生成的。你可以随意修改它，或者直接用上面的内容创建一个新文件。

```
1   'Tags'];

10      public $records = [
11          [
12              'id' => 1,
13              'tag' => 'Tag 1'
14          ],
15          [
16              'id' => 2,
17              'tag' => 'Tag 2'
18          ],
19          [
20              'id' => 3,
21              'tag' => 'Tag 3'
22          ],
23      ];
24  }
```

对于标签，我们也应该做同样的操作。在这个例子中，我们创建了三个标签。它应该放在 `/tests/Fixture/TagsFixture.php` 文件中。

```
1   'Posts'];

10      public $records = [
11          [
12              'id' => 1,
13              'category_id' => 1,
14              'user_id' => 1,
15              'title' => 'First Post Tilte',
16              'body' => 'This is the body of the first post...',
17              'created' => '2016-05-01 13:00:00',
18              'modified' => '2016-05-01 13:00:00'
19          ],
20      ];
21  }
```

对于文章，我们也应该做同样的操作。目前，一条 `记录` 就足够了。这段代码应该放在 `/tests/Fixture/PostsFixture.php` 文件中。

文章和标签之间存在 `belongsToMany` （多对多）关联，意味着一篇文章可以有多个标签，一个标签也可以被分配给多篇文章。为了存储这些数据，我们在默认数据源中创建了一个名为 `posts_tags` 的数据库表。既然现在我们想在测试中使用它，那么我们也需要为连接表创建固件。

```
1   'PostsTags'];

10      public $records = [
11          [
12              'post_id' => 1,
13              'tag_id' => 1
14          ],
15          [
16              'post_id' => 1,
17              'tag_id' => 3
18          ],
19      ];
20  }
```

你可以看到，`id 为 1` 的那篇文章拥有两个标签，分别是 `id 为 1` 和 `3` 的标签。

现在我们已经拥有了所有必需的固件。之前我们已经通过烘焙生成了 `/tests/TestCase/Controller/PostsControllerTest.php` 文件，但为了找点乐子，我们把它删掉，然后从头开始创建一个新的。

我们的测试用例应该有它自己的命名空间，并且会使用 `PostsController` 和 `IntegrationTestCase`，所以我们需要加载它们。

```
1  <?php
2  namespace App\Test\TestCase\Controller;

4  use App\Controller\PostsController;
5  use Cake\TestSuite\IntegrationTestCase;

```

让我们创建基础类。

```
7  class PostsControllerTest extends IntegrationTestCase
8  {

```

我们将固件加载到测试用例中。

```
10  public $fixtures = [
11        'app.posts',
12        'app.categories',
13        'app.users',
14        'app.tags',
15        'app.posts_tags'
16  ];

```

我们没有加载 `comments` 固件，因为在本测试中不会用到它。加载它既无必要，还会拖慢测试运行速度。

最后，为了遵循测试驱动开发（TDD）的原则，我们创建一个空的测试函数。

```
18     public function testIndex()
19     {
20         $this->markTestIncomplete('Not implemented yet.');
21     }
22  }
```

`markTestIncomplete` 方法继承自 `IntegrationTestCase`，并且是在 `PHPUnit` 中定义的。它的目的很简单，也不会让人意外：就是标记该测试未完成。实际的测试函数需要稍多解释，我们稍后会添加它。

## 关于集成测试

集成测试允许你在更高层次上测试你的应用程序。它模拟一个发送到你应用的 HTTP 请求。测试你的控制器时，也会同时测试与特定请求相关的所有组件、模型和辅助器。

以下方法可用于模拟 HTTP 请求：

- `get()` 发送 `GET` 请求
- `post()` 发送 `POST` 请求
- `put()` 发送 `PUT` 请求
- `delete()` 发送 `DELETE` 请求
- `patch()` 发送 `PATCH` 请求

## 断言方法

通过使用 `IntegrationTestCase`，我们可以访问许多不同的断言方法。以下是一些最常用的方法：

- `$this->assertResponseOk();` 检查调用控制器后是否得到 `2xx` 响应码。
- `$this->assertResponseSuccess();` 检查调用控制器后是否得到 `2xx` 或 `3xx` 响应码。
- `$this->assertResponseError();` 检查调用控制器后是否得到 `4xx` 响应码。
- `$this->assertResponseFailure();` 检查调用控制器后是否得到 `5xx` 响应码。
- `$this->assertResponseCode(404);` 检查响应码是否为 404。
- `$this->assertRedirectContains('/posts/edit/');` 检查重定向的 `location` 头部的一部分内容。
- `$this->assertResponseEquals('Call out Gouranga and be Happy!');` 检查响应内容是否等于给定值。
- `$this->assertResponseContains('Gouranga!');` 检查响应内容是否包含给定值。
- `$this->assertResponseNotContains('You are logged in!');` 检查响应内容是否不包含给定值。
- `$this->assertSession(1, 'Auth.User.id');` 检查 `session` 变量。
- `$this->assertEquals('rrd', $this->viewVariable('username'));` 检查 `view` 变量。
- `$this->assertContentType('application/json');` 检查内容类型。



