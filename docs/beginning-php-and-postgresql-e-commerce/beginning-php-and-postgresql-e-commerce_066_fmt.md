# 第 7 章：目录管理

**221**

```
}
}
?>
```

**3.** 修改 `admin.php` 文件，以加载新创建的 `admin_departments` 组件化模板：

```php
// 如果管理员已登录，加载管理页面菜单
$pageMenuCell = 'admin_menu.tpl';

if (isset ($_GET['Page']))
    $admin_page = $_GET['Page'];

// 如果未显式设置页面，则默认加载部门页面
else
    $admin_page = 'Departments';

// 如果正在执行注销操作...
if (isset ($_GET['Page']) && ($_GET['Page'] == 'Logout'))
{
    unset($_SESSION['AdminLogged']);
    header('Location: admin.php');
    exit;
}

// 选择要加载的管理页面...
if ($admin_page == 'Departments')
    $pageContentsCell = 'admin_departments.tpl';
}
```

## 工作原理：`admin_departments` 组件化模板

您在此练习中编写了大量代码，但仍然无法进行测试。这是先创建用户界面的难点所在。不过，仔细审视这些代码，它其实并不复杂。下面我们来看看 `admin_departments.tpl` 模板是如何实现的。

以下是用于构建表格行的 `{section}` 结构方案：

```
{section name=cDepartments loop=$admin_departments->mDepartments}
    {if $admin_departments->mEditItem ==
         $admin_departments->mDepartments[cDepartments].department_id}
        ...
    {else}
        ...
    {/if}
{/section}
```

默认情况下，部门名称和描述是不可编辑的。但当您点击某个部门的编辑按钮时，`$admin_departments->mEditItem` 会被设置为该点击部门的 `department_id` 值，Smarty 呈现逻辑便会生成可编辑的文本框来代替标签。这将允许管理员编辑所选部门的详细信息（在编辑模式下，更新/取消按钮会替代编辑按钮出现，如之前图示所示）。

每当用户点击这些按钮中的任意一个时，从 `admin_departments` 模板加载的 Smarty 插件函数（位于 `load_admin_departments.php` 中）便会执行，并对访客的操作做出响应。该函数能够识别点击的是哪个按钮，并在解析已提交变量列表并读取点击按钮的名称后，知晓该执行何种操作。在部门管理页面（参见 `admin_departments.tpl` 模板文件）中，按钮的名称格式类似 `submit_edit_dep_1`。

所有按钮名称均以 `submit` 开头，并以部门 ID 结尾。名称中间的部分是按钮类型的代码，用于指定对该部门执行何种操作。名为 `submit_edit_dep_1` 的按钮指示插件函数进入 `department_id` 值为 1 的部门的编辑模式。

请注意，对于“添加部门”按钮，按钮名称中指定的部门 ID 变得无关紧要，因为其值是由数据库自动生成的（`department_id` 是一个 `SERIAL` 列）。

在本例中，按钮类型可以是：

- `add_dep`：用于添加部门按钮

- `edit_dep`：用于编辑部门按钮

- `update_dep`：用于更新按钮

- `delete_dep`：用于删除按钮

- `edit_categ`：用于编辑类别按钮

根据点击按钮的类型，将调用相应的业务层方法。接下来我们来逐一介绍这些方法。

## 实现业务层

您从 `AdminDepartments` 类中调用了四个中间层方法。现在需要添加它们对应的业务层方法：

- `GetDepartmentsWithDescriptions`：返回要在部门管理页面中显示的部门列表。

- `UpdateDepartment`：更改部门的详细信息。其参数包括部门的 `department_id` 值、新名称和新描述。

- `DeleteDepartment`：删除由 `department_id` 参数指定的部门。