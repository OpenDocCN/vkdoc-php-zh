# 第 14 章 ■ 注册与登录

```php
public function initialize()
{
    $type = $this->getType();
    if (empty($type))
    {
        $configuration = Registry::get("configuration");
        if ($configuration)
        {
            $configuration = $configuration->initialize();
            $parsed = $configuration->parse("configuration/session");
            if (!empty($parsed->session->default) &&
                !empty($parsed->session->default->type))
            {
                $type = $parsed->session->default->type;
                unset($parsed->session->default->type);
                $this->__construct(array(
                    "type" => $type,
                    "options" => (array) $parsed->session->default
                ));
            }
        }
    }
    if (empty($type))
    {
        throw new Exception\Argument("无效类型");
    }
    switch ($type)
    {
        case "server":
        {
            return new Session\Driver\Server($this->getOptions());
            break;
        }
        default:
        {
            throw new Exception\Argument("无效类型");
            break;
        }
    }
}
```

工厂类以我们最近添加到 `Cache` 和 `Configuration` 中的自动配置功能开始。

`"sever"` 驱动程序是我们唯一会创建的，它将以 `Session\Driver\Server` 的实例形式返回。`Session\Driver` 类实际上与 `Cache\Driver` 和 `Configuration\Driver` 类完全相同。

如代码清单 14-10 所示，服务器驱动程序也相当简单。

**代码清单 14-10.** `Session\Driver\Server` 类

```php
namespace Framework\Session\Driver
{
    use Framework\Session as Session;

    class Server extends Session\Driver
    {
        /**
         * @readwrite
         */
        protected $_prefix = "app_";

        public function __construct($options = array())
        {
            parent::__construct($options);
            session_start();
        }

        public function get($key, $default = null)
        {
            $prefix = $this->getPrefix();
            if (isset($_SESSION[$prefix.$key]))
            {
                return $_SESSION[$prefix.$key];
            }
            return $default;
        }

        public function set($key, $value)
        {
            $prefix = $this->getPrefix();
            $_SESSION[$prefix.$key] = $value;
            return $this;
        }

        public function erase($key)
        {
            $prefix = $this->getPrefix();
            unset($_SESSION[$prefix.$key]);
            return $this;
        }

        public function __destruct()
        {
            session_commit();
        }
    }
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)