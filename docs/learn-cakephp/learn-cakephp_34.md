# 使用身份验证进行测试

在控制器中使用身份验证的可能性很大。这里暂时不会引入身份验证，因此我不会详细解释相关代码。在我们的博客中，任何人都可以编辑和删除任何内容，即使未登录也可以。身份验证是让用户登录并区分已登录和未登录用户的方法。授权是处理权限的方式，例如管理员可以编辑所有文章，而用户只能编辑自己的文章。

CakePHP 为处理身份验证和授权提供了良好的支持。

在你的 `/src/Controller/CommentsController.php` 文件中，应有以下代码。简而言之，这段代码确保未认证的用户无法访问任何与评论相关的 URL，除了无需身份验证即可访问的 `index` 和 `add`。

```
1  public function initialize()
2  {
3      parent::initialize();
4      $this->loadComponent(
5          'Auth',
6          [
7              'loginRedirect' => [
8                  'controller' => 'Users',
9                  'action' => 'login'
10              ]
11          ]
12      );
13  }

15  public function beforeFilter(Event $event)
16  {
17      parent::beforeFilter($event);
18      $this->Auth->allow(['index', 'add']);
19  }
```

别忘了在该文件的开头添加 `use Cake\Event\Event;`，因为我们在 `beforeFilter()` 方法中使用了 `Event` 类。

你的浏览器访问 `http://localhost/∼rrd/cakeBlog/comments` 时，如果有评论则会显示，但访问 `http://localhost/∼rrd/cakeBlog/comments/view/1` 时，由于未登录，会被重定向到 `http://localhost/∼rrd/cakeBlog/users/login`。我们尚未创建登录功能。

现在在 `/tests/TestCase/Controller/CommentsControllerTest.php` 文件中创建一个测试。以下测试应能成功运行：

```
1  public function testIndex()
2  {
3      $this->get('/comments');
4      $this->assertResponseOk();
5  }
```

让我们探讨一个需要身份验证的方法测试。由于未进行身份验证，我们应被重定向到登录页面。我们通过调用 `assertRedirect()` 方法来检查这一点。

```
1  public function testViewUnauthenticated()
2  {
3      $this->get('/comments/view/1');
4      $this->assertRedirect(
5          [
6              'controller' => 'Users',
7              'action' => 'login'
8          ]
9      );
10  }
```

你已经见过如何模拟 `session` 变量，这里正好需要它，因为登录数据存储在 `session` 中。

```
1  public function testViewAuthenticated()
2  {
3      $this->session([
4          'Auth' => [
5              'User' => [
6                  'id' => 1,
7                  'username' => 'rrd',
8                  'role' => 10
9              ]
10          ]
11      ]);
12      $this->get('/comments/view/1');
13      $this->assertResponseOk();
14  }
```

如果已登录，我们应得到正常的 OK 响应，而不是重定向。

## 测试 JSON 响应

JSON（JavaScript Object Notation）是一种标准格式，使用人类可读的文本传输由属性-值对组成的数据对象。JSON 是一种与语言无关的数据格式。它最初是为 JavaScript 创建的，但许多编程语言都能生成和解析 `JSON-format` 数据。JSON 文件扩展名是 `.json`。

以下是一个 JSON 字符串示例：

```
{
"name": "rrd",
"isAlive": true,
"age": 40,
"phoneNumbers": [
{
"type": "home",
"number": "987 654 3210"
},
{
"type": "mobile",
"number": "123 456 7890"
}
],
"children": [],
"spouse": "syj"
}
```

你可以在根目录下找到一个真实的 JSON 示例文件，因为 `composer` 将其 `composer.json` 放在那里。

Web 服务的 Ajax 调用响应格式为 JSON 或 XML。为了测试这些响应，我们首先需要将 `RequestHandler` 组件添加到控制器的 `initialize()` 方法中，如下所示。

```
1  $this->loadComponent('RequestHandler');
```

我们需要在 `/config/routes.php` 文件的第 45 行附近添加以下代码，以使其处理 `json` 扩展名。

```
1  Router::extensions(['json']);
```

让我们在 `CommentsControllerTest.php` 文件中创建测试方法。

```
1  public function testAdd()
2  {
3      $this->configRequest(
4          [
5              'headers' => ['Accept' => 'application/json']
6          ]
7      );
8      $data = [
9          'comment' => 'Call out Gouranga and be happy',
10          'user_id' => 1,
11          'post_id' => 1,
12          'category_id' => 2
13      ];
14      $this->post('/comments/add.json', $data);
15      $this->assertResponseSuccess();

17      $expected = [
18          'comment' => [
19              'comment' => 'Call out Gouranga and be happy',
20              'user_id' => 1,
21              'post_id' => 1,
22              'category_id' => 2,
23              'id' => 2
24          ],
25      ];
26      $expected = json_encode($expected, JSON_PRETTY_PRINT);
27      $this->assertEquals($expected, $this->_response->body());
28  }
```

首先，我们模拟请求头，然后模拟 POST 数据，最后检查响应的 HTTP 头部代码及其内容。

在你的 `CommentsController.php` 文件的 `add()` 方法中，删除（或注释掉）以下代码行：

```
1  return $this->redirect(['action' => 'index']);
```

你的代码应该类似这样：

```
1  public function add()
2  {
3      $comment = $this->Comments->newEntity();
4      if ($this->request->is('post')) {
5          $comment = $this->Comments->patchEntity(
6              $comment,
7              $this->request->data
8          );
9          if ($this->Comments->save($comment)) {
10              $this->Flash->success(__('The comment has been saved.'));
11              //return $this->redirect(['action' => 'index']);
12          } else {
13              $this->Flash->error(
14                  __('The comment not saved. Please, try again.')
15              );
16          }
17      }
18      $users = $this->Comments->Users->find('list', ['limit' => 200]);
19      $posts = $this->Comments->Posts->find('list', ['limit' => 200]);
20      $this->set(compact('comment', 'users', 'posts'));
21      $this->set('_serialize', ['comment']);
22  }
```

现在我们的测试应该可以成功运行了。

## 总结

在本章中，你学习了如何通过模拟来测试身份验证背后的功能。然后，你看到了一个测试控制器响应的示例，该控制器除了返回 HTML 外，还响应 JSON 数据。

