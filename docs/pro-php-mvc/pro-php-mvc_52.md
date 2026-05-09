# 第 23 章 ■ CodeIgniter：MVC

```php
public static function all($where = null, $fields = null, $order = null, $direction = "asc", $limit = null, $page = null)
{
    $user = new User();

    // 选择字段
    if ($fields) {
        $user->db->select(join(",", $fields));
    }

    // 缩小搜索范围
    if ($where) {
        $user->db->where($where);
    }

    // 排序结果
    if ($order) {
        $user->db->order_by($order, $direction);
    }

    // 限制结果数量
    if ($limit && $page) {
        $offset = ($page - 1) * $limit;
        $user->db->limit($limit, $offset);
    }

    // 获取所有用户
    $query = $user->db->get("user");
    $users = array();

    foreach ($query->result() as $row) {
        // 创建并填充用户对象
        $user = new User();
        $user->_populate($row);

        // 将用户添加到集合
        array_push($users, $user);
    }

    return $users;
}
}
```

这些方法与我们在`Model`类上定义的方法非常相似，并执行我们将在控制器中需要的各种数据库查询。当我们考虑到 CodeIgniter 并没有采用我们关于连接器和查询的理念，而是实现了自己的数据库查询系统时，情况就与我们习惯的做法有所不同了。不过，CodeIgniter 确实支持方法链，所以情况也不算太糟！

[www.it-ebooks.info](http://www.it-ebooks.info/)

