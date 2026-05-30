# 第 23 章 ■ CodeIgniter：MVC

`Users`控制器的代码非常简单，包含了与我们自己框架中类似的“钩子”方法。`_getUser()`方法访问会话来获取用户 ID，并加载`User`模型来检索用户行。接下来我们编写`register()`操作和对应的视图，如清单 23-6 和 23-7 所示。

**清单 23-6** `application/controllers/users.php`（节选）

```php
class Users extends CI_Controller
{
    //...

    public function register()
    {
        $success = false;

        // 加载验证库
        $this->load->library("form_validation");

        // 如果表单已提交
        if ($this->input->post("save")) {
            // 初始化验证规则
            $this->form_validation->set_rules(array(
                array(
                    "field"   => "first",
                    "label"   => "名字",
                    "rules"   => "required|alpha|min_length[3]|max_length[32]"
                ),
                array(
                    "field"   => "last",
                    "label"   => "姓氏",
                    "rules"   => "required|alpha|min_length[3]|max_length[32]"
                ),
                array(
                    "field"   => "email",
                    "label"   => "邮箱",
                    "rules"   => "required|max_length[100]"
                ),
                array(
                    "field"   => "password",
                    "label"   => "密码",
                    "rules"   => "required|min_length[8]|max_length[32]"
                )
            ));

            // 如果表单数据通过验证...
            if ($this->form_validation->run()) {
                // 加载用户模型
                $this->load->model("user");
```

[www.it-ebooks.info](http://www.it-ebooks.info/)