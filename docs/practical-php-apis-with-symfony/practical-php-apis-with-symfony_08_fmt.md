# 1. 设计输入、输出和操作

在本章中，我们将构建面向操作 API 的基本核心功能。具体来说，我们将编写必要的代码来：

*   创建用于表示输入和输出的模型

*   创建用于表示操作的服务

*   创建一个可访问的集合来存储和检索这些操作

*   根据请求负载执行操作

*   验证负载

*   创建接收并执行操作请求的端点

## 第 1 节 请求输入和输出

在 API 请求的上下文中，以下是对输入和输出的快速分解：

*   **输入**：指您提供给 API 的数据或指令。根据 API 的设计，这些数据通常出现在请求负载或请求头中。例如，在“发送付款”操作中，输入可能包括收款人信息、金额以及可能的参考代码。

*   **输出**：指 API 处理完您的请求后，您从 API 收到的数据或响应。输出格式可能因 API 而异，但通常以 JSON 或 XML 格式提供。例如，“发送付款”操作的输出可能是一个包含成功状态和交易 ID 的确认消息。

在接下来的章节中，我们将为请求输入和输出创建一个类，以便稍后对其进行反序列化和验证。

*   您会注意到在本节中，一些模型的属性被包裹了 Symfony 验证约束。在“第 5 节 验证操作”中，我们将学习如何根据这些属性所持有的约束条件来验证这些模型。

### 输入模型

下面是一个表示 API 输入请求模型的类：

```php
namespace App\Api\Input;

use Symfony\Component\Validator\Constraints\NotBlank;

readonly class ApiInput
{
    public function __construct(
        #[NotBlank(message: '操作名称不能为空')]
        public string $operation,
        public array $data = []
    ) {}
}
```

输入模型包含以下属性：

*   **操作**：保存我们要执行的操作的名称。

*   **数据**：保存执行操作所需的数据。如果操作不需要数据，则此属性可以为空。

前面的 `ApiInput` 模型使用了 `NotBlank` Symfony 验证约束，这意味着操作名称不能为空。

### 输出模型

```php
namespace App\Api\Output;

readonly class ApiOutput
{
    public function __construct(
        public mixed $data,
        public int $code
    ) {}
}
```

输出模型包含以下属性：

*   **数据**：操作执行结果。它可以是一个对象数组、一个简单对象，如果操作不返回任何结果，甚至可以为空。

*   **代码**：响应中返回的 HTTP 状态码。在选择输出 HTTP 状态码时，我们应该考虑以下提示：

    *   如果我们返回最终的操作结果，我们应该选择 `200 OK` HTTP 状态码。

    *   如果我们返回一个操作进程跟踪键（端点接受执行，但执行被延迟），我们应该选择 `202 ACCEPTED` HTTP 状态码。

    *   如果我们不返回任何操作结果，我们应该选择 `204 NO CONTENT` HTTP 状态码。`204` 只能在不返回任何内容时使用，即使是一个空的 JSON 对象也不应该返回。

### 操作输入模型

每个操作执行时都可能需要一些数据。例如，一个支付操作需要指定支付金额和收款方。由于每个操作执行的任务不同，它们的输入模型也各不相同。

以下示例展示了一个假设的支付操作模型：

```php
use Symfony\Component\Validator\Constraints as Assert;
readonly class SendPaymentInput
{
    public function __construct(
        #[Assert\NotBlank(message: '收款方不能为空')]
        public string $receiver,
        #[Assert\NotBlank(message: '金额不能为空')]
        #[Assert\GreaterThan(0, message: '金额必须大于 0')]
        public float|int $amount,
    ){}
}
```

与之前的 `ApiInput` 模型类似，`SendPaymentInput` 模型也使用了 Symfony 的 [NotBlank](https://symfony.com/doc/current/reference/constraints/NotBlank.html) 约束，以指定 `receiver` 和 `amount` 属性值都是必填的。这意味着如果缺少其中任何一个值，`SendPayment` 操作就无法执行。此外，它还使用了 [GreaterThan](https://symfony.com/doc/current/reference/constraints/GreaterThan.html) 约束来指定金额必须大于 0。

最后一个输入模型指定了 `receiver` 和 `amount` 属性都是必填的。然而，有些输入模型可能允许某些参数是可选的。例如，`SendPaymentInput` 模型可以包含一个可选的支付标签。我们可以利用 PHP 的类型提示功能将 `label` 标记为可选：

```php
use Symfony\Component\Validator\Constraints as Assert;
readonly class SendPaymentInput
{
    public function __construct(
        #[Assert\NotBlank(message: '收款方不能为空')]
        public string $receiver,
        #[Assert\NotBlank(message: '金额不能为空')]
        #[Assert\GreaterThan(0, message: '金额必须大于 0')]
        public float|int $amount,
        public ?string $label = null
    ){}
}
```

如你所见，该模型将 `label` 声明为一个可选的字符串（使用了 `?` 修饰符），并将 `null` 作为其默认值。

### 操作输出模型

与我们为输入数据创建模型的方式相同，我们也可以为输出数据创建模型。继续以 `SendPaymentInput` 操作为例，我们来创建一个包含两个属性（`id` 和 `status`）的输出模型。

```php
readonly class SendPaymentOutput
{
    public function __construct(
        public string $id,
        public string $status
    ){}
}
```

这个输出模型将在操作执行后用于表示结果。

```php
$id = '......';
$status = 'ACCEPTED';
return new ApiOutput(
    new SendPaymentOutput($id, $status),
);
```

这种方式比使用原始数组要好，因为它为返回的数据提供了清晰的结构。这使得开发者更容易理解预期的响应格式。

对于从数据库返回结果数据的操作（例如，返回一个 `Client` 实体的元素列表的操作），我们应该创建具体的输出数据对象，而不是直接序列化实体本身。通过创建专门的输出对象（也称为 DTO – 数据传输对象），你可以在业务逻辑和数据表示之间保持清晰的分离。这有助于防止持久化逻辑和基础设施关注点泄露到表示层。

在下一节中，我们将分析一个完整的示例。