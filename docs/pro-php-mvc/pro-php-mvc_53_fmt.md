# 第 23 章 ■ CodeIgniter：MVC

### 控制器

有了`User`模型之后，我们可以开始编写`Users`控制器了，如清单 23-5 所示。

**清单 23-5** `application/controllers/users.php`（节选）

```php
class Users extends CI_Controller
{
    public $user;

    public static function redirect($url)
    {
        header("Location: {$url}");
        exit();
    }

    protected function _isSecure()
    {
        // 获取用户会话
        $this->_getUser();

        if (!$this->user) {
            self::redirect("/login");
        }
    }

    protected function _getUser()
    {
        // 加载会话库
        $this->load->library("session");

        // 获取用户 ID
        $id = $this->session->userdata("user");

        if ($id) {
            // 加载用户模型
            $this->load->model("user");

            // 获取用户
            $this->user = new User(array(
                "id" => $id
            ));
        }
    }
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)