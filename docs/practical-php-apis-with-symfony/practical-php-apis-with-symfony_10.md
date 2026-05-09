# 总结

我们已经确立了面向操作的 API 的核心要素：操作及其相关的元数据。在下一节中，我们将探讨如何有效地发现和收集这些已定义的操作，确保它们在我们的 API 中随时可访问。

## 第三节 收集与发现操作

我们已经学习了如何使用 `OperationInterface` 和 `OperationMetadata` 属性创建单个 API 操作。现在，让我们探讨如何构建一个专门用于管理这些操作的服务。该服务将负责：

*   **持有操作集合**：该服务将作为一个中央仓库，存储应用中所有已定义的操作。
*   **通过名称检索操作**：我们可以在服务中通过唯一名称高效检索特定操作。这简化了操作查找，并提升了代码的清晰度。

通过创建这个服务，我们将能以一种更有组织、更高效的方式来管理和访问应用中的 API 操作。

### 操作集合

首先，我们需要创建一个用于存储操作集合的类。

```
namespace App\Api\Collection;
class OperationCollection
{
private array $operations;
public function setOperations(array $operations): void
{
$this->operations = $operations;
}
public function getOperation(string $operation): ApiOperation
{
if(!isset($this->operations[$operation])){
throw new NotFoundOperationException(sprintf('Operation %s is not defined', $operation));
}
return $this->operations[$operation];
}
}
```

操作集合非常直接。它只包含一个用于加载操作的 setter 方法和一个用于获取具体操作的 getter 方法。如果操作不存在，则会抛出一个 `NotFoundOperationException` 异常。

我们将稍后创建这个异常。目前，我们只需知道它是在操作不存在时抛出的异常。

现在的问题是：我们如何加载操作数组？答案是使用 [Symfony 服务配置器](https://symfony.com/doc/current/service_container/configurators.html)。这些是特殊的服务，允许开发者将服务逻辑与提供所需配置的服务解耦。

让我们来看看操作集合配置器：

### 操作配置器

操作配置器将包含必要的代码，用于将操作填充到集合中。它使用一个名为 `ApiOperation` 的类来包装每个操作，该类接收操作处理器和操作元数据。

```
namespace App\Api;
use App\Api\Attribute\OperationMetadata;
use App\Api\Operation\OperationInterface;
readonly class ApiOperation {
public function __construct(
public OperationInterface $handler,
public OperationMetadata  $metadata
){}
}
```

现在，让我们探讨操作配置器是如何工作的。

```
namespace App\Api\Collection;
use App\Api\ApiOperation;
use App\Api\Attribute\OperationMetadata;
use Symfony\Component\DependencyInjection\Attribute\TaggedIterator;
class OperationCollectionConfigurator {
public function __construct(
#[TaggedIterator('api.operation')] private readonly iterable    $apiOperations
){}
public function configure(OperationCollection $operationCollection): void
{
$operations = [];
foreach ($this->apiOperations as $operation){
$metadata = $this->readAttribute(OperationMetadata::class, $operation);
$operations[$metadata->name] = new ApiOperation($operation, $metadata);
}
$operationCollection->setOperations($operations);
}
/**
* @template T
* @param class-string $attrClass
* @return T|null
*/
private function readAttribute(string $attrClass, object $object)
{
$reflectionClass = new \ReflectionClass($object);
$attrs = $reflectionClass->getAttributes($attrClass);
if(!empty($attrs)) {
$attr = reset($attrs);
return $attr->newInstance();
}
throw new \RuntimeException(sprintf('Operation class %s has no metadata', get_class($object)));
}
}
```

配置器服务包含稍多一些的代码。我们从构造函数开始看起。它注入了带有 `TaggedIterator` 属性的参数。这个属性会将所有标记为 `api.operation` 的服务（即所有操作）加载到 `$apiOperations` 可迭代对象中。

*   当我们创建 `OperationInterface` 时，我们使用了 `AutoConfigure` 属性来将所有实现了该接口的服务标记为 `api.operation`。

私有的 `readAttribute` 方法利用 PHP 的反射功能来检查作为第二个参数传入的对象是否标记了由第一个参数指定的属性。如果是，则返回该属性的一个新实例；否则，抛出一个 PHP 异常。

`configure` 方法遍历 `$apiOperations` 数组中的操作。对于每个操作，它使用 `readAttribute` 方法获取元数据，并将操作及其元数据包装到一个 `ApiOperation` 对象中。循环结束后，它通过 `setOperations` 方法将操作设置到 `OperationCollection` 中。

我们必须告诉 Symfony 使用这个配置器来配置 `OperationCollection` 服务。为此，我们需要在 `config/services.yaml` 文件的 `services` 部分进行定义。

```
App\Api\Collection\OperationCollection:
configurator: ['@App\Api\Collection\OperationCollectionConfigurator', 'configure']
```

通过上述配置，Symfony 就知道必须使用 `OperationCollectionConfigurator` 的 `configure` 方法来配置 `OperationCollection` 服务。



### 发现操作

发现操作非常简单，只需使用来自 `OperationCollection` 服务的 `getOperation` 方法即可。让我们创建一个 `ApiOperationHandler` 服务来封装此逻辑。

```php
namespace App\Api;
use App\Api\Collection\OperationCollection;
use App\Api\Input\ApiInput;
use App\Api\Output\ApiOutput;
class ApiOperationHandler
{
public function __construct(
private readonly OperationCollection $operationCollection,
){ }
public function performOperation(ApiInput $apiInput): ApiOutput
{
$operation = $this->operationCollection->getOperation($apiInput->operation);
return $operation->handler->perform($apiInput->data);
}
}
```

`performOperation` 方法接收一个输入模型作为参数。由于该模型包含操作名称，该方法将其传递给 `OperationCollection::getOperation` 方法，并获取一个 `ApiOperation` 包装器，其中包含操作处理器和操作元数据。然后，它执行操作（使用 `OperationInterface::perform` 方法）并将结果作为 `ApiOutput` 模型返回。

### 处理 `NotFoundOperationException` 错误

在编写操作集合服务时，我们在未找到操作时抛出了一个异常。让我们为这个异常编写类：

```php
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
class NotFoundOperationException extends NotFoundHttpException
{
}
```

这个异常类本身不做任何事情。它只是继承自 Symfony 的 `NotFoundHttpException`，该类会自动将 HTTP 响应码设置为“404 Not Found”。现在，我们需要指示 Symfony 在抛出此异常时，在缺失操作的消息中返回一个格式化的 JSON 响应。为此，我们将编写一个 [Symfony 事件订阅器](https://symfony.com/doc/current/event_dispatcher.html%2523listeners-or-subscribers)，它将持续监听 Symfony 的 [KernelException 事件](https://symfony.com/doc/current/reference/events.html%2523kernel-exception)。每次抛出异常时，Symfony 都会触发此事件。

```php
namespace App\EventSubscriber;
use App\Exception\NotFoundOperationException;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\KernelEvents;
class KernelSubscriber implements EventSubscriberInterface
{
public static function getSubscribedEvents(): array
{
return [
KernelEvents::EXCEPTION => 'onException'
];
}
public function onException(ExceptionEvent $event): void
{
$exception = $event->getThrowable();
if($exception instanceof NotFoundOperationException){
$event->setResponse(new JsonResponse([
'errors' => [
'operation' => $exception->getMessage()
]
]));
}
}
}
```

如上代码所示，`onException` 方法在内核异常事件被触发后执行。该方法检查抛出的异常是否是 `NotFoundOperationException` 的实例。如果是，它会创建一个包含异常消息的 [JsonResponse](https://symfony.com/doc/current/components/http_foundation.html%2523creating-a-json-response)，并将其传递给事件的 `setResponse` 方法。最后，此响应将被返回给最终客户端。

### 结论

我们已经探讨了如何创建一个作为中心枢纽的集合仓库，用于存储所有已定义的操作。此外，我们还引入了一个负责从集合中检索操作并执行它们的处理器。更进一步，我们处理了在集合中遇到不存在操作的情况。通过实现优雅地处理“未找到”操作的机制，我们可以确保 API 的健壮性和用户友好性。

在下一节中，我们将把重点转向惰性操作。通过利用 Symfony 的惰性加载技术，我们可以确保应用程序保持响应和高效，仅在需要时加载必要的组件。

## 第 4 节：惰性操作

在上一节中，我们学习了如何使用 Symfony 的功能来创建一个集合，用于存储可在 API 中执行的操作。在拥有大量可用操作的环境中，考虑预先加载所有操作对性能的影响至关重要。这就是惰性操作可以发挥作用的地方。

惰性操作允许我们将操作的加载推迟到真正需要它们的时候。这种方法可以显著提高 API 的性能，尤其是在处理大量操作或实例化成本较高的操作时。

例如，我们可以设想一个注入了邮件服务的操作。即使你不需要邮件服务，为了构造该操作并将其添加到集合中，它也会被始终实例化。

为了能够向集合中添加惰性操作，我们依赖于 `Autoconfigure` 属性的 `lazy` 参数。

### `LazyOperationInterface` 接口

`LazyOperationInterface` 的代码类似于 `OperationInterface`，但有以下不同之处：

- `Autoconfigure` 属性将参数 `lazy` 设置为 true。这意味着实现此接口的操作将被惰性加载到集合中。
- 它添加了一个名为 `getMetadata` 的方法来返回操作元数据。此方法是必需的，因为对于惰性加载的（代理）服务，无法可靠地访问元数据属性。

```php
use App\Api\Attribute\OperationMetadata;
use App\Api\Output\ApiOutput;
use Symfony\Component\DependencyInjection\Attribute\Autoconfigure;
#[Autoconfigure(lazy: true, tags: ['api.operation'])]
interface LazyOperationInterface
{
public function perform(mixed $data): ApiOutput;
public function getMetadata(): OperationMetadata;
}
```

我们必须注意，惰性操作服务不能是只读类或 final 类。

### 将操作标记为惰性

要将操作标记为惰性，我们只需实现上述接口即可。

```php
use App\Api\Attribute\OperationMetadata;
use App\Api\Operation\LazyOperationInterface;
use App\Api\Output\ApiOutput;
use App\Security\Miscelanea\OperationNames;
class SendReportingByEmail implements LazyOperationInterface
{
public function perform(mixed $data): ApiOutput
{
/**
* 通过电子邮件发送报告所需的代码
*/
return new ApiOutput([
'report_id' => '5899865', 'email_status' => 'queued'
], 200);
}
public function getMetadata(): OperationMetadata
{
return new OperationMetadata(
name: OperationNames::SendReportingByEmail->name,
input: SendReportingByEmailInput::class
);
}
}
```

我们可以看到，操作元数据在 `getMetadata` 方法中返回。

### 修改 `OperationCollectionConfigurator`

我们需要修改 `OperationCollectionConfigurator` 服务，以检查操作是否被标记为惰性。如果是，它将必须使用 `getMetadata` 方法来提取元数据，而不是读取 `OperationMetadata` 属性。

```php
public function configure(OperationCollection $operationCollection): void
{
$operations = [];
foreach ($this->apiOperations as $operation){
if($operation instanceof LazyOperationInterface) {
$metadata = $operation->getMetadata();
$operations[$metadata->name] = new ApiOperation(
$operation,
$metadata
);
}
else{
$metadata = $this->readAttribute(OperationMetadata::class,  $operation);
$operations[$metadata->name] = new ApiOperation($operation, $metadata);
}
}
$operationCollection->setOperations($operations);
}
```



### 结论

我们已经使用了 `Autoconfigure` 属性的 `lazy` 参数，使得所有实现了 `LazyOperation` 接口的操作都以惰性模式加载。这让我们能够提升性能，并且仅在需要执行时实例化重量级操作。

在下一节中，我们将把焦点转向验证。我们将探讨确保传入请求符合预期要求的策略，为可靠且无差错的执行操作铺平道路。

## 第 5 节 验证操作

在“第 1 节 请求输入与输出”中，我们讨论了定义可能需要特定参数才能执行的操作。确保这些参数有效且一致，对于避免操作执行过程中的错误至关重要。

本节将探讨如何利用 [Symfony 的验证](https://symfony.com/doc/6.4/validation.html)功能来实现这一目标。

### 验证服务

我们首先创建一个服务来封装验证逻辑：

```
namespace App\Validation;
use App\Exception\InvalidPayloadException;
use Symfony\Component\Serializer\SerializerInterface;
use Symfony\Component\Validator\Exception\ValidationFailedException;
use Symfony\Component\Validator\Validator\ValidatorInterface;
class ValidationHandler
{
public function __construct(
private readonly SerializerInterface $serializer,
private readonly ValidatorInterface $validator
){}
public function deserializeAndValidate(array|string $payload, string $className)
{
try{
$object = (is_array($payload))
? $this->serializer->denormalize($payload, $className)
: $this->serializer->deserialize($payload, $className, 'json')
;
}
catch(\Symfony\Component\Serializer\Exception\ExceptionInterface $e) {
throw new InvalidPayloadException();
}
$errors = $this->validator->validate($object);
if(count($errors) > 0) {
throw new ValidationFailedException(null, $errors);
}
return $object;
}
}
```

该服务使用了两个关键的 Symfony 组件：

-   **Symfony 序列化器**：处理器首先使用序列化服务，将传入的请求负载转换为元数据输入参数中指定的输入模型。负载格式可以是 JSON（编码字符串）或数组。
-   **Symfony 验证器**：反序列化后，使用 Symfony 验证器服务对输入模型进行验证。这确保了数据符合预定义的约束条件，保证了数据的一致性，并防止操作执行过程中可能出现错误。

如果验证过程顺利结束，将会返回已反序列化的输入模型，供操作处理器进一步处理。如果验证失败（数据无效），则会抛出一个 Symfony 的 `ValidationFailedException` 异常。

我们将反序列化过程包裹在 try-catch 块中，以处理潜在的错误，例如不正确的数据类型、缺少必填字段或无效的 JSON。如果出现这些问题，则会抛出一个 `InvalidPayloadException` 异常：

```
namespace App\Exception;
use Symfony\Component\HttpFoundation\Response;
class InvalidPayloadException extends \RuntimeException
{
public function __construct(string $msg = '格式错误或无法识别的有效负载' , int $code = Response::HTTP_BAD_REQUEST, ?\Throwable $previous = null)
{
parent::__construct($msg, $code, $previous);
}
}
```

`InvalidPayloadException` 仅添加了一条消息，提示已发送了无效的有效负载。

### 在执行操作前进行验证

我们对 `ApiOperationHandler` 进行一些修改，以便在执行操作之前验证数据。

```
use App\Api\Collection\OperationCollection;
use App\Api\Input\ApiInput;
use App\Api\Output\ApiOutput;
use App\Validation\ValidationHandler;
class ApiOperationHandler
{
public function __construct(
private readonly ValidationHandler $validationHandler,
private readonly OperationCollection $operationCollection,
){ }
public function performOperation(ApiInput $apiInput): ApiOutput
{
$operation = $this->operationCollection->getOperation($apiInput->operation);
$inputData = ($operation->metadata->input)
? $this->validationHandler->deserializeAndValidate($apiInput->data, $operation->metadata->input)
: $apiInput->data
;
return $operation->handler->perform($inputData);
}
}
```

现在，构造函数也注入了 `ValidationHandler` 服务。`performOperation` 方法会验证 API 输入数据（如果操作的元数据指定了输入模型类）。如果验证失败，将抛出一个 `ValidationFailedException` 异常，并且操作将不会被执行。

### 处理反序列化和验证错误

正如我们处理 `NotFoundOperationException` 一样，我们必须处理验证错误异常（`InvalidPayloadException` 和 `ValidationFailedException`），以便能够返回可读的 JSON 输出。让我们修改 `KernelSubscriber`。

```
namespace App\EventSubscriber;
use App\Exception\InvalidPayloadException;
use App\Exception\NotFoundOperationException;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\KernelEvents;
use Symfony\Component\Validator\Exception\ValidationFailedException;
class KernelSubscriber implements EventSubscriberInterface
{
public static function getSubscribedEvents(): array
{
return [
KernelEvents::EXCEPTION => 'onException',
];
}
public function onException(ExceptionEvent $event): void
{
$exception = $event->getThrowable();
if($exception instanceof ValidationFailedException){
$errors = [];
foreach($exception->getViolations() as $violation) {
$errors[$violation->getPropertyPath()] = $violation->getMessage();
}
$event->setResponse( new JsonResponse([
'errors' => $errors
], 400));
return;
}
if($exception instanceof InvalidPayloadException) {
$event->setResponse( new JsonResponse([
'payload' => $exception->getMessage()
], 400));
return;
}
if($exception  instanceof NotFoundOperationException){
$event->setResponse( new JsonResponse([
'errors' => [
'operation' => $exception->getMessage()
]
], 404));
return;
}
}
}
```

我们添加了两个 `if` 块来处理 `InvalidPayloadException` 和 `ValidationFailedException`。对于 `ValidationFailedException`，它会遍历所有违反约束的错误，并创建一个数组，其中键是导致错误的属性，值是错误消息。对于 `InvalidPayloadException`，它会返回异常消息。

### 结论

在建立了执行操作和确保传入负载有效性的机制之后，我们只需要创建接收操作请求的端点即可。

## 第 6 节 连接各模块：构建 API 端点

我们已经建立了用于处理操作执行的核心组件。现在，是时候让客户端能够与 API 进行交互了。本节重点介绍如何创建一个 Symfony 控制器，作为接收操作请求的端点。

通过在此控制器中定义路由，我们将建立一个指定的 URL，客户端可以使用它来提交操作请求。

让我们探索一下如何将我们创建的各种服务连接到外部世界，从而让我们的 API 能够处理传入的操作调用。

### 创建控制器

在“第 1 节 请求输入与输出”中，我们学习了如何构造我们的 `ApiInput`，以便将待执行的操作和执行该操作所需的数据封装到其中。尽管我们的控制器似乎必须严格按照 `ApiInput` 模型的结构接收数据，但事实并非必须如此。在本节的剩余部分，我们将学习如何通过编写少量额外代码，就能以多种不同方式接收输入数据。



### 将接收到的载荷直接映射到 `ApiInput`

第一种情况最为直接。接收到的载荷必须与 `ApiInput` 模型完全匹配。`MapRequestPayload` 属性会在内部将请求载荷反序列化为模型并进行验证。如果发现任何错误，Symfony 会创建一个包含错误信息的 `ValidationFailedException`，将其包装成 `UnprocessableEntityHttpException`，并以 422 Unprocessable Entity HTTP 状态码抛出。

```php
use App\Api\ApiOperationHandler;
use App\Api\Input\ApiInput;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\HttpKernel\Attribute\MapRequestPayload;

class ApiController extends AbstractController
{
    public function __construct(
        private readonly ApiOperationHandler $apiOperationHandler
    ) {}

    #[Route('/api/v1/operation', name: 'api_operation', methods: ['POST'])]
    public function operationAction(#[MapRequestPayload] ApiInput $apiInput): JsonResponse
    {
        $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
        return new JsonResponse($apiOutput->data, $apiOutput->code);
    }
}
```

### 将操作名称作为路由参数接收

此类情况下，操作名称来源于路由参数 `operationSlug`，而操作数据来自请求载荷。为了处理该请求，我们需要做一些修改：

#### 从 Slug 中获取操作名称

为了确定需要执行哪个操作，让我们在 `OperationNames` 枚举中创建一个静态方法，将操作 slug 转换为操作名称。

```php
use function Symfony\Component\String\u;
// .......

public static function getBySlug(string $slug): string
{
    return u($slug)->camel()->title();
}
```

`getBySlug` 函数使用 Symfony String 的 `u` 函数将 slug 转为驼峰式并将首字母大写。通过这种方式，我们成功将 slug 转换成了有效的操作名称。

#### 手动创建 `ApiInput` 模型

在下面的代码中，我们将按照以下步骤创建 `ApiInput` 模型：

- 确保接收到的载荷是有效的 JSON。这一步是强制性的，因为此情况下我们没有使用 `MapRequestPayload` 属性。
- 使用 `getBySlug` 函数获取操作名称，并将其作为 `ApiInput` 构造函数的第一个参数。
- 将接收到的载荷（操作数据）解码为数组，并将其作为 `ApiInput` 构造函数的第二个参数。

我们来在 `ValidationHandler` 类中创建一个方法，确保接收到的载荷是有效的 JSON：

```php
class ValidationHandler
{
    public function __construct(
        private readonly SerializerInterface $serializer,
        private readonly ValidatorInterface $validator
    ) {}

    // 其他方法...

    public function validateAndGetJsonInput(string $content): array
    {
        $data = json_decode($content, true);
        if (is_null($data)) {
            if (json_last_error() !== JSON_ERROR_NONE) {
                throw new InvalidPayloadException('Invalid Json Payload');
            }
            $data = [];
        }
        return $data;
    }
}

#[Route('/api/v1/operation/{operationSlug}', name: 'api_slug_operation', methods: ['POST'])]
public function operationActionSlug(string $operationSlug, Request $request, ValidationHandler $validationHandler): JsonResponse
{
    $data = $validationHandler->validateAndGetJsonInput($request->getContent());
    $apiInput = new ApiInput(
        OperationNames::getBySlug($operationSlug),
        $data
    );
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}
```

如果没有错误，`ApiInput` 的第二个参数（保存操作数据）会接收到 JSON 解码后的数据。请注意，此数据会根据操作输入模型，由 `performOperation` 方法进行验证。为了进行验证，它使用了上一章中的 `ValidationHandler` 服务。

### 不使用 Slug 参数，为每个操作使用独立的路由

此情况与前一种类似，但路由不通过参数传递操作 slug，而是直接定义操作名称。这样，我们可以在保持 API 面向操作的同时，方便文档的生成。

```php
#[Route('/api/v1/operation/send-payment', name: 'api_send_payment_operation', methods: ['POST'])]
public function operationActionSendPayment(Request $request, ValidationHandler $validationHandler): JsonResponse
{
    $data = $validationHandler->validateAndGetJsonInput($request->getContent());
    $apiInput = new ApiInput(
        OperationNames::SendPayment->name,
        $data
    );
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}
```

### 使用 HTTP GET 路由

HTTP GET 请求通常通过 URL 查询字符串接收数据。因此，为了获取数据并将其传递给 `ApiInput` 模型，我们可以依赖 Symfony Request 的查询参数。

```php
#[Route('/api/v1/operation/get-campaigns', name: 'api_send_get_campaigns', methods: ['GET'])]
public function operationActionGetCampaigns(Request $request): JsonResponse
{
    $apiInput = new ApiInput(
        OperationNames::GetCampaigns->name,
        $request->query->all()
    );
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}
```

### 在路由参数中传递数据

可能会出现某些参数通过路由参数传递，而不是使用查询字符串或请求载荷的情况。让我们看下面的例子。

```php
#[Route('/api/v1/sensor/{sensorId}/operation/update-data', name: 'api_update_sensor_data', methods: ['PATCH'])]
public function operationActionUpdateSensorData(int $sensorId, Request $request, ValidationHandler $validationHandler): JsonResponse
{
    $data = $validationHandler->validateAndGetJsonInput($request->getContent());
    $apiInput = new ApiInput(
        OperationNames::UpdateSensorData->name,
        array_merge(
            $data,
            ['sensorId' => $sensorId]
        )
    );
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}
```

在此情况下，我们需要手动将 `sensorId` 参数添加到接收到的载荷中，这通过 PHP 的 `array_merge` 函数实现。

作为改进，我们可以创建一个枚举来保存这些参数名称，而不是硬编码 `sensorId` 参数名。

```php
enum OperationParametersNames: string
{
    case SensorId = 'sensorId';
}
```

然后，我们将 `array_merge` 部分改成这样：

```php
array_merge(
    json_decode($request->getContent(), true),
    [OperationParametersNames::SensorId->value => $sensorId]
)
```

### 与传统的 CRUD 路由结合使用

与我们可以通过多种方式定义端点来接收待执行的操作和数据一样，我们也可以将面向操作的路由与传统的 CRUD 路由结合使用。

```php
#[Route('/api/v1/template')]
class ApiTemplatingController extends AbstractController
{
    public function __construct(
        private readonly ApiOperationHandler $apiOperationHandler
    ) {}

    #[Route('', name: 'api_get_templates', methods: ['GET'])]
    public function getTemplates(): JsonResponse
    {
        return new JsonResponse([]);
    }

    #[Route('/operation', name: 'api_template_operation', methods: ['POST'])]
    public function operationAction(#[MapRequestPayload] ApiInput $apiInput): JsonResponse
    {
        $apiOutput = $this->apiOperationHandler->performOperation($apiInput);
        return new JsonResponse($apiOutput->data, $apiOutput->code);
    }
}
```

如上控制器所示，它结合了一个 CRUD 类型的端点（`getTemplates`）和一个面向操作的端点（`operationAction`）。




### 结论

在本节中，我们学习了如何以不同方式构建端点，以便客户端能够连接到我们的 API 并执行操作。在下一章中，我们将研究如何为我们的 API 创建版本，从而允许某些操作仅适用于一个或多个 API 版本。

## 第 7 节 对 API 进行版本管理

在开发 API 时，添加新功能和新操作是很常见的，同样常见的是，其中一些新功能仅对特定版本可用。在本节中，我们将探讨如何指定一个操作仅对一个或多个版本可用。

### 一个新的元数据参数

操作元数据是指定操作在哪些版本中可用的理想位置。为了实现这一点，让我们向该属性添加一个新参数。

```
#[\Attribute(\Attribute::TARGET_CLASS)]
class OperationMetadata
{
    public function __construct(
        public readonly string $name,
        public readonly ?array $version = null,
        public readonly ?string $input = null
    ){ }
}
```

`version` 参数指定了哪些 API 版本可以执行该操作。如果未指定版本，则意味着不要求任何特定版本。现在，我们需要以某种方式接收 API 版本。一种方式是使用 Symfony 路由参数。

### 接收版本

以下代码向控制器路由添加了一个 `version` 参数。

```
#[Route('/api/{version}/operations', name: 'post_api_operation', methods: ['POST'])]
public function operationAction(string $version, #[MapRequestPayload] ApiInput $apiInput): JsonResponse
{
    $apiOutput = $this->apiOperationHandler->performOperation($apiInput, $version);
    return new JsonResponse($apiOutput->data, $apiOutput->code);
}
```

`version` 参数作为路由参数传入。然后，该参数被传递给 `ApiOperationHandler`，以便其能在内部检查版本是否与操作匹配。

### 检查版本

现在，是时候对 `ApiOperationHandler` 进行一些修改，使其能够检查版本了。

```
public function performOperation(ApiInput $apiInput, ?string $version = null): ApiOutput
{
    $operation = $this->operationCollection->getOperation($apiInput->operation);
    if(!is_null($version) && $operation->metadata->version && !in_array($version, $operation->metadata->version)) {
        OperationDoesNotSupportApiVersionException(sprintf(
            '操作支持的版本：%s - 使用的版本：%s',
            implode(“，”, $operation->metadata->version),
            $version
        ));
    }
    $inputData = ($operation->metadata->input)
        ? $this->validationHandler->deserializeAndValidate($apiInput->data, $operation->metadata->input)
        : null
    ;
    $operation->handler->perform($inputData);
}
```

`performOperation` 方法现在接收 `version` 作为一个可选参数。然后，该方法检查以下条件：

- 传入了版本。
- 操作元数据中保存了版本信息。
- 传入的版本不存在于操作允许的版本数组中。

如果满足上述条件，则会抛出一个 `OperationDoesNotSupportApiVersionException` 异常，告知客户端该操作无法在该 API 版本上执行。让我们看一下异常代码。

```
namespace App\Exception;
class OperationDoesNotSupportApiVersionException extends \RuntimeException
{
}
```

这个异常本身不执行任何操作。我们只需在 `KernelSubscriber` 中捕获它，以便在 JSON 响应中告知用户版本错误。

### 在 KernelSubscriber 中检查 OperationDoesNotSupportApiVersionException 异常

正如我们在上一节中所做的那样，我们必须在内核订阅器中检查版本错误，并正确地格式化错误信息。

```
if($exception instanceof OperationDoesNotSupportApiVersionException){
    $event->setResponse( new JsonResponse([
        'errors' => [
            'operation_version' => $exception->getMessage()
        ]
    ], 400));
    return;
}
```

如你所见，我们只是为这种类型的异常添加了另一个条件判断。

### 结论

在本节中，我们扩展了 `OperationMetadata` 属性，添加了一个新参数来指定操作所支持的 API 版本。在最后一节中，我们将看到如何利用 Symfony 的 Monolog Bundle 及其事件系统来自动记录每个 API 的请求和响应。

## 第 8 节 监控操作

在开发 API（无论采用何种方法）时，另一个需要考虑的关键方面是如何监控它们的性能。例如：

-   一个操作需要多长时间来处理？
-   哪些操作最容易导致瓶颈？
-   每个操作或端点的错误率（4xx/5xx）是多少？
-   有多少比例的请求超过了可接受的延迟阈值（例如，第 95 百分位数）？

有许多工具允许 DevOps 人员监控 API，例如 Prometheus + Grafana、ELK（Elastic/Logstash/Kibana）、OpenTelemetry 等。

在本章中，我们不会介绍如何设置外部监控工具。相反，我们将重点介绍如何使用 Symfony 的内置功能生成包含详细执行数据的日志文件。之后，开发人员和 DevOps 工程师可以将这些日志导入任何监控平台，以构建仪表板并分析结果。

### 创建 Monolog 处理器

在上一章的*第 3 节 先决条件*中，我们学习了如何安装 Monolog bundle：它是 Symfony 应用程序中日志记录的事实标准。其配置位于 `config/packages/monolog.yaml` 文件中。以下是一个示例，它定义了一个写入专用日志文件的自定义处理器：

```
monolog:
    handlers:
        operations_performance:
            type: stream
            path: "%kernel.logs_dir%/op_performance.log"
            level: debug
            channels: [op_performance]
    channels:
        - deprecation # 弃用警告会记录到专用的“deprecation”通道中（如果存在）
        - op_performance
```

这里，`operations_performance` 处理器监听 `op_performance` 通道，并将所有 debug 级别（及更高级别）的记录写入 `var/log/op_performance.log` 文件（`%kernel.logs_dir%` 变量的默认位置）。由于最近的 Symfony 版本包含了自动服务注册功能，内核会为你配置的每个 Monolog 自定义通道动态创建一个服务。服务 ID 由处理器的通道名称转换而来，转换为驼峰格式，并附加 `Logger` 字符串。在此示例中，服务变为 `$opPerformanceLogger`。




### 创建日志包装器以记录操作执行信息

在本节中，我们将探讨如何注入上一节配置的日志记录器，以及如何使用它向配置的文件写入内容。

```php
use Psr\Log\LoggerInterface;
use Symfony\Component\Uid\Uuid;
class OperationPerformanceLogger
{
public function __construct(
private readonly LoggerInterface $opPerformanceLogger
){}
public function logStartOperation(string $operation, ?array $inputData): string
{
$id = Uuid::v4();
$this->opPerformanceLogger->info(json_encode([
'operation' => $operation,
'id' => $id,
'input_data' => $inputData ?? [],
'ts_start' => microtime(true)
]));
return (string)$id;
}
public function logEndOperation(string $operation, string $id, int $responseCode, ?array $outputData): void
{
$this->opPerformanceLogger->info(json_encode([
'operation' => $operation,
'id' => $id,
'output_data' => $outputData ?? [],
'ts_end' => microtime(true),
'response_code' => $responseCode
]));
}
}
```

我们从构造函数开始。如你所见，我们注入了 `$opPerformanceLogger` 服务，它实现了 Symfony 内核所需的 `Psr\Log\LoggerInterface`。该服务暴露了两个关键方法：

- **`logStartOperation`**：此方法应在收到操作请求时调用。它接收操作名称和输入值数组（如果没有则为空数组），并将以下信息（以 JSON 格式）写入日志文件：
    - **`operation`**：操作名称
    - **`id`**：用于标识请求的 ID。这里我们使用 Symfony Uid 组件生成一个 UUID v4 作为 ID。
    - **`input_data`**：接收到的输入数据数组，如果没有接收到输入数据则为空数组。
    - **`ts_start`**：收到操作的时间戳。

    此方法返回生成的 ID。在接下来的几节中，我们将学习如何在请求中携带此 ID，以便用于记录操作终止数据。

- **`logEndOperation`**：此方法应在操作请求执行结束、响应已发送给客户端后调用。它接收操作名称、在 `logStartOperation` 方法中生成的 ID、返回给客户端的 HTTP 响应码以及执行结果数据。该方法将以下信息写入日志文件：
    - **`operation`**：操作名称。
    - **`id`**：用于标识请求的 ID，与 `logStartOperation` 方法中生成的 ID 相同。
    - **`output_data`**：输出数据数组，如果没有向客户端返回响应数据则为空数组。这可能是出现 204 No Content HTTP 响应的情况。
    - **`ts_end`**：操作结束的时间戳。
    - **`response_code`**：返回给客户端的 HTTP 响应码。

有了这个日志服务，你可以利用 Symfony 的事件系统自动记录每个 API 操作的开始和结束。

### 在收到操作请求时写入日志

我们将使用 `kernelEvents.REQUEST` Symfony 事件来检测是否收到了操作请求，如果是，则写入日志。为此，我们将创建一个新的 Symfony 内核订阅者：

```php
use App\Api\Input\ApiInput;
use App\Api\Logger\OperationPerformanceLogger;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\RequestEvent;
use Symfony\Component\HttpKernel\KernelEvents;
use Symfony\Component\Serializer\Exception\ExceptionInterface;
use Symfony\Component\Serializer\SerializerInterface;
class OperationPerformanceSubscriber implements EventSubscriberInterface
{
public const string OP_REQ_ATTR = 'OP_REQ';
public function __construct(
private readonly SerializerInterface $serializer,
private readonly OperationPerformanceLogger $operationPerformanceLogger
){}
public static function getSubscribedEvents(): array
{
return [
KernelEvents::REQUEST   => ['onOperationReceived', 2]
];
}
public function onOperationReceived(RequestEvent $event): void
{
$uri = $event->getRequest()->getUri();
if(!preg_match('#\/api\/v[1-9]{1}\/operation#', $uri)) {
return;
}
try{
$apiInput = $this->serializer->deserialize($event->getRequest()->getContent(), ApiInput::class, 'json');
if(!empty($apiInput->operation)) {
$operationRequestId = $this->operationPerformanceLogger->logStartOperation($apiInput->operation, $apiInput->data);
$event->getRequest()->attributes->set(self::OP_REQ_ATTR, $operationRequestId . ':' . $apiInput->operation);
}
}
catch(ExceptionInterface $e) {}
}
}
```

让我们来分析一下上述新的订阅者。该订阅者持续监听 `KernelEvents::REQUEST` 事件，并在捕获到该事件后执行 `onOperationReceived` 方法。此方法执行以下逻辑：

- 检查请求 URI 是否匹配操作请求路由路径。
- 如果不匹配，则返回且不执行额外逻辑。
- 如果匹配，则将负载反序列化为 `ApiInput` 模型。
- 如果反序列化过程没有错误，该方法使用 `OperationPerformanceLogger` 服务中的 `logStartOperation` 方法写入操作请求信息。然后，它获取由 `logStartOperation` 方法返回的 `$operationRequestId`，并将其保存到请求属性包中，以便稍后使用。

正如你所注意到的，`onOperationReceived` 的 catch 块中没有代码。这是因为，如果在验证接收到的 API 输入负载后出现任何错误，客户端将收到 400 Bad Request 响应，并且请求不会继续到处理器，因此在这种情况下不值得写入日志。



### 在操作请求结束时写入日志

最后，我们只需在操作请求结束后写入日志。为此，我们将依赖 Symfony 的 `KernelEvents.TERMINATE` 事件。

```
use App\Api\Input\ApiInput;
use App\Api\Logger\OperationPerformanceLogger;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\RequestEvent;
use Symfony\Component\HttpKernel\Event\TerminateEvent;
use Symfony\Component\HttpKernel\KernelEvents;
use Symfony\Component\Serializer\Exception\ExceptionInterface;
use Symfony\Component\Serializer\SerializerInterface;
class OperationPerformanceSubscriber implements EventSubscriberInterface
{
public const string OP_REQ_ATTR = 'OP_REQ';
public function __construct(
private readonly SerializerInterface $serializer,
private readonly OperationPerformanceLogger $operationPerformanceLogger
){}
public static function getSubscribedEvents(): array
{
return [
KernelEvents::REQUEST   => ['onOperationReceived', 2],
KernelEvents::TERMINATE => 'onOperationTerminated'
];
}
public function onOperationReceived(RequestEvent $event): void
{
$uri = $event->getRequest()->getUri();
if(!preg_match('#^\/api\/v[1-9]{1}\/operation#', $uri)) {
return;
}
try{
$apiInput = $this->serializer->deserialize($event->getRequest()->getContent(), ApiInput::class, 'json');
if(!empty($apiInput->operation)) {
$operationRequestId = $this->operationPerformanceLogger->logStartOperation($apiInput->operation, $apiInput->data);
$event->getRequest()->attributes->set(self::OP_REQ_ATTR, $operationRequestId . ':' . $apiInput->operation);
}
}
catch(ExceptionInterface $e) {}
}
public function onOperationTerminated(TerminateEvent $event): void
{
$opReqAttr = $event->getRequest()->attributes->get(self::OP_REQ_ATTR);
if($opReqAttr) {
list($opId, $opName) = explode(':', $opReqAttr);
$responseArray = ($event->getResponse()->getContent())
? json_decode($event->getResponse()->getContent(), true)
: []
;
$responseCode = $event->getResponse()->getStatusCode();
$this->operationPerformanceLogger->logEndOperation($opName, $opId, $responseCode, $responseArray);
}
}
}
```

现在我们来分析一下 `onOperationTerminated` 事件。其执行流程如下：

-   它检查请求属性包中是否包含键名为 `OP_REQ` 的值。如果包含，则意味着在 `REQUEST` 事件中已经记录了请求操作，现在我们需要记录其终止。
-   它使用 `:` 标记字符串来分割 `$opReqAttr`，从而获取操作 ID 和操作名称的值。
-   它通过对接收到的响应内容应用 PHP 的 `json_decode` 函数来创建响应数组；如果未向客户端发送响应内容，则创建 `null` 值。
-   它获取 HTTP 响应码。
-   最后，它使用 `OperationPerformanceLogger logEndOperation` 方法将终止信息写入日志。

