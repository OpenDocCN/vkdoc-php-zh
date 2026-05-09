# 总结

在本节中，我们创建了开始构建操作所需的基本模型。在下一节中，我们将学习如何创建操作以及如何创建必要的服务来容纳它们。

## 第 2 节：构建操作

本节概述了创建定义良好的 API 操作的关键原则：

- **接口强制**：每个操作都必须实现一个特定的接口。该接口定义了一个标准方法，所有操作都必须使用它来封装其执行逻辑。这有助于提升一致性并简化操作管理。
- **元数据用于清晰**：每个操作都将用一个元数据属性进行标记。该属性作为一个中心位置，用于存储有关操作的基本信息，例如其名称和表示其输入模型的类。这种元数据增强了代码的可读性并简化了操作的发现。

总之，这些原则确保了 API 操作结构良好、可维护，并具有清晰的执行逻辑和易于访问的元数据。

### 操作接口

`OperationInterface` 充当一个契约，所有操作都必须实现它。`perform` 方法将是操作用来封装其逻辑的方法。

```
namespace App\Api\Operation;
use App\Api\Output\ApiOutput;
use Symfony\Component\DependencyInjection\Attribute\Autoconfigure;
#[Autoconfigure(tags: ['api.operation'])]
interface OperationInterface
{
public function perform(mixed $data): ApiOutput;
}
```

[AutoConfigure](https://symfony.com/doc/current/service_container/tags.html%2523autoconfiguring-tags) 属性告诉 Symfony 内核，那些实现了此接口的服务将被标记为 `api.operation`。这意味着每个操作在实现了 `OperationInterface` 后都会被如此标记。

`perform` 方法必须返回一个 `ApiOutput` 类的实例。如果你还记得上一节的内容，这个类保存了操作请求的结果。

### 操作元数据

操作元数据属性包含两个属性。第一个指定操作名称，第二个指定操作输入模型的类。

```
namespace App\Api\Attribute;
#[\Attribute(\Attribute::TARGET_CLASS)]
readonly class OperationMetadata
{
public function __construct(
public string $name,
public ?string $input = null
){ }
}
```

一个请求操作的有效负载将在执行操作之前被反序列化为由 `$input` 参数指定的输入类。如果输入类的属性使用了 Symfony 验证约束进行注解，那么反序列化后的模型还将被验证，以确保接收到的数据是有效的。

我们将在本章的剩余部分看到整个过程。

### 一个操作类

到目前为止，我们知道一个操作必须实现 `OperationInterface`，并且需要使用 `OperationMetadata` 属性进行标记。在本节中，我们将展示一个操作类的示例。

```
namespace App\Api\Operation;
use App\Api\Attribute\OperationMetadata;
use App\Api\Input\Operation\SendPaymentInput;
use App\Api\Output\ApiOutput;
#[OperationMetadata(
name: 'SendPayment',
input: SendPaymentInput::class
)]
class SendPaymentOperation implements OperationInterface
{
/**
* @param SendPaymentInput $data
*/
public function perform(mixed $data): ApiOutput
{
// 该方法将包含负责发送支付的代码
$id = '......';
$status = 'ACCEPTED';
return new ApiOutput(new SendPaymentOutput($id, $status), 202);
}
}
```

操作元数据属性指定了操作名称（`SendPayment`）和输入数据类名（`SendPaymentInput`）。`perform` 方法将执行操作，并返回包含输出模型形式的结果以及 HTTP 响应码的 `ApiOutput`。

为了避免在元数据的 `name` 参数中硬编码操作名称，我们可以创建一个 PHP 枚举来集中管理所有操作名称：

```
enum OperationNames
{
case SendPayment;
}
```

现在，我们可以这样修改 `OperationMetadata` 的 `name` 参数：

```
#[OperationMetadata(
name: OperationNames::SendPayment->name,
input: SendPaymentInput::class
)]
```



### `Array` 作为操作结果输出

在上一节中，我们讨论了从数据库返回结果数据的操作。我们明确指出，最好创建一个具体的输出模型，而不是直接序列化数据库实体，这样能防止持久化逻辑和基础设施问题泄露到表示层。假设我们正在创建一个操作来返回数据库中存储的客户数据。我们可以将这个操作命名为：`GetClients`。

在创建操作之前，我们先为客户实体创建一个输出模型。假设我们只想返回客户名称、企业邮箱、联系人姓名和联系人电话。

```
readonly class ClientOutput {
public function __construct(
public string $name,
public string $email,
public string $contactName,
public string $contactPhone
){}
}
```

现在，我们已经准备好创建 `GetClientsOperation` 类：

```
#[OperationMetadata(
name: 'GetClients',
input: GetClientsInput::class
)]
class GetClientsOperation implements OperationInterface
{
public function __construct(
private readonly EntityManagerInterface $em
){}
/**
* @param GetClientsInput $data
*/
public function perform(mixed $data): ApiOutput
{
$clients = $this->em->getRepository(Client::class)->findAll();
$clientsOutput = [];
foreach($clients as $client) {
$clientsOutput[] = new ClientOutput(
$client->getName(),
$client->getEmail(),
$client->getContactName(),
$client->getContactPhone()
);
}
return new ApiOutput($clientsOutput, 200);
}
}
```

如上述代码所示，即使对 `Client` 实体进行了修改，也不会影响到表示层，这可以防止某些类型的错误，例如序列化过程中的循环引用问题。

我想强调的是，负责遍历客户列表并为每个客户创建 `ClientOutput` 的代码应该放在主操作之外，理想情况下应置于一个独立的服务中。通过这样做，我们可以有效地解耦功能，增强可维护性和可扩展性。由于本书并非专注于 SOLID 原则或如何构建面向领域的应用，我们不会再对此方面进行深入讨论。

