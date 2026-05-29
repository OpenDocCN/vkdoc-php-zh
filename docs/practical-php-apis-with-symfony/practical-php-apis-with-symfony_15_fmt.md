# 4. 上下文特定操作

在前面的章节中，我们已经学习了以下内容：

- 如何创建能够根据请求有效负载内容处理操作的端点
- 如何保护我们的端点和操作
- 如何在后台执行操作

在本章中，我们将深入探讨将端点封装在特定上下文中的过程，确保它只能执行为该上下文定义的操作。这种方法不仅增强了我们 API 的安全性和清晰度，而且通过促进关注点的清晰分离，简化了多个控制器的管理。这样做可以精简我们的代码库，使其随着应用程序的发展更易于维护和扩展。

有必要澄清一点，我们不应将保护操作（确保只有特定角色才能执行操作，如第 2 章所述）与端点上下文化的概念混为一谈。基于角色的访问控制主要关注用户权限，而上下文化则根据其指定的上下文限制端点可用的操作。这种区分促进了更有组织性和可维护性的代码结构。这两种策略可以有效地结合使用以增强安全性。例如，“支付”上下文可能只允许与支付相关的操作，其中某些操作限制为特定角色才能执行，从而确保敏感操作仅由授权用户执行。

## 第一节 上下文化端点

在本节中，我们将构建必要的元素，以便能够为端点包裹一个上下文。

### 上下文枚举

首先，我们将创建一个枚举，用于保存上下文以及这些上下文相关的操作。

```php
enum Context
{
    case inventoryManagement;
    case payment;
    case orderManagement;

    public function getAllowedOperations()
    {
        match($this) {
            self::inventoryManagement => [
                'AddInventoryItem',
                'UpdateInventoryItem',
                'RemoveInventoryItem',
                'ListInventoryItems'
            ],
            self::payment => [
                'SendPayment',
                'RefundPayment',
                'ChargePayment'
            ],
            self::orderManagement => [
                'CreateOrder',
                'UpdateOrder',
                'CancelOrder'
            ]
        };
    }
}
```

上述枚举包含以下情况：

- `inventoryManagement`：表示与库存管理相关的操作的上下文。
- `payment`：表示与支付相关的操作的上下文。
- `orderManagement`：表示与订单管理操作的上下文。

`getAllowedOperations()` 方法返回一个包含该上下文所允许操作的数组。

### `OperationContext` 属性

在本节中，我们将创建用于为端点包裹上下文的属性。

```php
use App\Api\Context;

#[\Attribute(\Attribute::TARGET_METHOD)]
class OperationContext
{
    public function __construct(
        public readonly Context $context
    ){ }
}
```

如你所见，`OperationContext` 属性的构造函数接收一个 `Context` 枚举作为参数。这使得我们可以简单地指定上下文，正如我们将在下一节中看到的那样。

### 为端点包裹上下文

在本节中，我们将使用 `OperationContext` 属性来指定面向操作的端点上下文。

```php
#[Route('/inventory-management/operation', name: 'api_inventory_management_operation', methods: ['POST'])]
#[OperationContext(Context::inventoryManagement)]
public function inventoryManagementOperationsAction(#[MapRequestPayload] ApiInput $apiInput): JsonResponse
{
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}

#[Route('/payment/operation', name: 'api_payment_operation', methods: ['POST'])]
#[OperationContext(Context::payment)]
public function paymentOperationsAction(#[MapRequestPayload] ApiInput $apiInput): JsonResponse
{
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}

#[Route('/order-management/operation', name: 'api_order_management_operation', methods: ['POST'])]
#[OperationContext(Context::orderManagement)]
public function orderManagementOperationsAction(#[MapRequestPayload] ApiInput $apiInput): JsonResponse
{
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}
```

如你所见，我们使用了 `OperationContext` 属性来指定每个端点只能执行其上下文允许的操作。

### 结论

在本节中，我们创建了一个 PHP 枚举，用于集中特定上下文所允许的操作，并实现了一个 `OperationContext` 属性来指定可以在控制器中执行的操作。最后，我们创建了三个不同的端点（每个上下文一个），并使用 `OperationContext` 属性将它们包裹起来，并将相应的上下文传递给其构造函数。

在下一节中，我们将创建一个专用的订阅者来监听 Symfony 的 `KernelController` 事件。该订阅者需要确定所请求的操作执行是否可以被该端点执行。

## 第二节 检测指定上下文中的端点

我们需要一种方法来检测请求是否已路由到指定了上下文的端点，以便检查要执行的操作是否被允许。为了实现这一点，让我们依赖 Symfony 的[控制器内核事件](https://symfony.com/doc/6.4/reference/events.html%23kernel-controller)。

### 监听内核控制器事件

我们将在第 1 章创建的 `KernelSubscriber` 类中添加一个方法来监听控制器事件。

#### 用于通知上下文操作无效的异常

在编写事件相关的订阅器之前，我们先编写当操作无法执行时将抛出的异常。

```php
use Symfony\Component\HttpKernel\Attribute\WithHttpStatus;
#[WithHttpStatus(403)]
class InvalidContextForOperationException extends \RuntimeException
{
    public function __construct(string $context, string $operation, int $code = 403, ?\Throwable $previous = null)
    {
        $msg = sprintf('操作 %s 不属于上下文 %s', $operation, $context);
        parent::__construct($msg, $code, $previous);
    }
}
```

上述异常继承了 PHP 的 `RuntimeException`。`WithHttpStatus` 属性负责在抛出异常时发送给客户端的 HTTP 响应，而 `$code` 变量则指异常本身的错误码。在此特定情况下，两者都设置为 403，但如有需要也可以设置不同的值。我们使用 403 Access Denied 异常是因为我们拒绝执行某个操作的访问，尽管原因与在第 3 章中使用 Symfony 投票器时不同。

#### 内核控制器订阅器

现在我们有了一个用于报告上下文错误的异常，终于可以编写内核控制器订阅器了。

```php
public function onKernelController(ControllerEvent $event): void
{
    $controllerAttributes = $event->getAttributes();
    $operationContextAttributes = $controllerAttributes[OperationContext::class] ?? null;
    if(count($operationContextAttributes) > 0) {
        $operationContextAttribute = $operationContextAttributes[0];
        $apiInput = $this->validationHandler->deserializeAndValidate($event->getRequest()->getContent(), ApiInput::class, null);
        if(!in_array($apiInput->operation, $operationContextAttribute->context->getAllowedOperations())) {
            throw new InvalidContextForOperationException($operationContextAttribute->context->name, $apiInput->operation);
        }
    }
}
```

以上监听器的工作流程如下：

- 检查与事件关联的控制器是否标注了 `OperationContext` 属性。

- 如果控制器已标注该属性，则通过 `Context` 枚举的 `getAllowedOperations` 方法检查操作是否可执行。如果不可执行，则抛出 `InvalidContextForOperationException` 异常以通知错误。

#### 在 `KernelSubscriber` 中检查 `InvalidContextForOperationException`

与迄今为止处理所有异常的方式一样，我们必须指示 `KernelSubscriber` 订阅器检测此异常并将消息格式化为 JSON。

```php
if($exception instanceof InvalidContextForOperationException){
    $event->setResponse(new JsonResponse([
        'errors' => [
            'operation_endpoint_context' => $exception->getMessage()
        ]
    ], 400));
}
```

如您所见，我们仅为 `InvalidContextForOperationException` 添加了另一个条件。

### 结论

我们再次借助 Symfony 的事件系统来捕获控制器请求。这样，我们就可以检查控制器方法是否标注了 `OperationContext` 属性，如果是，则确保上下文允许请求的操作执行。

在下一章中，我们将创建必要的应用程序测试，以确保我们的 API 按预期运行。