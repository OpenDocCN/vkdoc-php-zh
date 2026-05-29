# Baked 控制器概览

看看我们先前通过 `bake` 生成的 `/src/Controllers/PostsController.php` 文件。

```php
1   <?php
2   namespace App\Controller;
```

所有控制器共享同一个命名空间，即 `App\Controller`。

```php
4   use App\Controller\AppController;
```

使用 `use` 操作符，我们引入了 `App\Controller\AppController` 命名空间。

```php
6   /**
7    * Posts Controller
8    *
9    * @property \App\Model\Table\PostsTable $Posts
10   */
11  class PostsController extends AppController
12  {
```

所有控制器类都继承自 `AppController` 类，该类是 Cake 的 `Controller` 类的扩展。这确保了我们能够使用组件，并拥有用于重定向等方法。

```php
14      /**
15       * Index method
16       *
17       * @return \Cake\Network\Response|null
18       */
19      public function index()
20      {
```

由 `bake` 生成的 `index` 方法用于列出所有文章。

```php
21          $this->paginate = [
22              'contain' => ['Categories', 'Users']
23          ];
24          $posts = $this->paginate($this->Posts);
```

我们希望在视图中使用分页功能，以便在文章很多时允许用户进行分页。我们加载了关联的 `Categories` 和 `Users` 模型来查看文章的相关数据。

```php
26          $this->set(compact('posts'));
27          $this->set('_serialize', ['posts']);
28      }
```

然后我们设置视图所需的变量。我们只设置了一个变量 `post`，然后设置了 `_serialize` 变量，因为它在 JSON 响应中可能很有用。

```php
30      /**
31       * View method
32       *
33       * @param string|null $id Post id.
34       * @return \Cake\Network\Response|null
35       * @throws \Cake\Datasource\Exception\RecordNotFoundException
36       *    When record not found.
37       */
```

`view` 方法用于获取单独一篇文章及其所有关联数据，并为视图设置 `post` 变量。

```php
38      public function view($id = null)
39      {
40          $post = $this->Posts->get($id, [
41              'contain' => ['Categories', 'Users', 'Tags', 'Comments']
42          ]);
44          $this->set('post', $post);
45          $this->set('_serialize', ['post']);
46      }
```

由 `bake` 生成的下一个方法是 `add`。此方法用于添加新文章。

```php
48      /**
49       * Add method
50       *
51       * @return \Cake\Network\Response|void Redirects on successful
52       *    add, renders view otherwise.
53       */
54      public function add()
55      {
56          $post = $this->Posts->newEntity();
```

首先，我们为新文章创建一个新的实体对象。

```php
57          if ($this->request->is('post')) {
58              $post = $this->Posts->patchEntity(
59                  $post,
60                  $this->request->data
61              );
```

如果在请求中已经收到了一些文章数据，我们将其加载到 `$post` 实体对象中。

```php
62              if ($this->Posts->save($post)) {
63                  $this->Flash->success(__('The post has been saved.'));
64                  return $this->redirect(['action' => 'index']);
65              } else {
66                  $this->Flash->error(
67                      __('The post not saved. Please, try again.')
68                  );
69              }
70          }
```

接下来，我们尝试保存新文章，并设置成功或错误消息。如果保存操作成功，我们将用户重定向到 `posts/index` URL。

```php
71          $categories = $this->Posts->Categories
72              ->find('list', ['limit' => 200]);
73          $users = $this->Posts->Users->find('list', ['limit' => 200]);
74          $tags = $this->Posts->Tags->find('list', ['limit' => 200]);
75          $this->set(compact('post', 'categories', 'users', 'tags'));
76          $this->set('_serialize', ['post']);
77      }
```

然后我们为视图设置所有变量。

由 `bake` 生成的下一个方法是 `edit`，用于编辑之前保存的文章。

```php
79      /**
80       * Edit method
81       *
82       * @param string|null $id Post id.
83       * @return \Cake\Network\Response|void Redirects
84       *      on successful edit, renders view otherwise.
85       * @throws \Cake\Network\Exception\NotFoundException
86       *      When record not found.
87       */
88      public function edit($id = null)
89      {
90          $post = $this->Posts->get($id, [
91              'contain' => ['Tags']
92          ]);
```

首先，我们通过其 id 从模型中获取文章。


```php
93          if ($this->request->is(['patch', 'post', 'put'])) {
94              $post = $this->Posts->patchEntity(
95                  $post,
96                  $this->request->data
97              );
98              if ($this->Posts->save($post)) {
99                  $this->Flash->success(__('The post has been saved.'));
100                  return $this->redirect(['action' => 'index']);
101              } else {
102                  $this->Flash->error(
103                      __('The post not saved. Please, try again.')
104                  );
105              }
106          }
```

如果通过 HTTP `patch`、`post` 或 `put` 方法在请求中接收到文章数据，我们尝试保存文章。

```php
107          $categories = $this->Posts->Categories
108              ->find('list', ['limit' => 200]);
109          $users = $this->Posts->Users->find('list', ['limit' => 200]);
110          $tags = $this->Posts->Tags->find('list', ['limit' => 200]);
111          $this->set(compact('post', 'categories', 'users', 'tags'));
112          $this->set('_serialize', ['post']);
113      }
```

然后，和前面方法一样，我们设置将在相应视图中使用的变量。

由 `bake` 生成的最后一个方法是 `delete`，用于删除已保存的文章。

```php
115      /**
116       * Delete method
117       *
118       * @param string|null $id Post id.
119       * @return \Cake\Network\Response|null Redirects to index.
120       * @throws \Cake\Datasource\Exception\RecordNotFoundException
121       *      When record not found.
122       */
123      public function delete($id = null)
124      {
125          $this->request->allowMethod(['post', 'delete']);
126          $post = $this->Posts->get($id);
127          if ($this->Posts->delete($post)) {
128              $this->Flash->success(__('The post has been deleted.'));
129          } else {
130              $this->Flash->error(
131                  __('The post not deleted. Please, try again.')
132              );
133          }
134          return $this->redirect(['action' => 'index']);
135      }
136  }
```

这个方法很直接：我们删除由其 `id` 标识的文章，然后将用户重定向到 `posts/index`。
