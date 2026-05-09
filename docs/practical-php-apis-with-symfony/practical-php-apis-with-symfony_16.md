# 5. 测试 API

在构建并保护了我们的面向操作的 API 之后，巩固其功能并确保其按预期运行至关重要。本章作为最后一章，将向您展示如何使用 Symfony 应用程序测试来检查 API 是否按预期工作。

## 第 1 节 创建测试类

Symfony Test Pack 组件允许我们通过 Symfony 生成器轻松创建测试类，因此我们用它来自动生成测试类。

```
bin/console make:test
```

执行上述命令后，系统会要求您输入一些信息来创建测试类：

- **要生成的测试类型**：选择 `WebTestCase` 类型。
- **类的名称**：输入您想要的名称。

完成后，它将在项目根目录的 `/tests/` 目录下生成测试类，通常为 `tests/Controller/ApiControllerTest.php`。如果我们打开测试类，会看到类似以下内容：

```
namespace App\Tests;
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
class ApiControllerTest extends WebTestCase
{
    public function testSomething(): void
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/');
        $this->assertResponseIsSuccessful();
        $this->assertSelectorTextContains('h1', 'Hello World');
    }
}
```

该类包含一个示例测试（`testSomething`）。让我们逐行分析其内容：

1. 创建一个客户端实例，以便我们可以发送请求
2. 向 URL 执行 GET 请求
3. 检查响应状态码是否在 200–299 范围内
4. 检查 `h1` 选择器是否包含 "Hello World" 文本

这只是一个模板。如果您直接运行此测试，它会失败，因为 "/" 路由和预期的 HTML 内容（`<h1>Hello World</h1>`）在应用程序中尚不存在。

让我们在下一节编写我们的测试。

### 结论

在本节中，我们探讨了如何在 Symfony 生成器的帮助下，使用 Symfony Test Pack 组件创建测试类。通过执行 `bin/console make:test` 命令，我们可以轻松生成测试类的模板，其中包含一个基本的示例测试。在下一节中，我们将用一组测试填充测试类，以便覆盖尽可能多的用例。

## 第 2 节 API 测试

以下各节展示了一系列测试，每个测试都针对特定的使用场景。

- 为了能够发送自定义的 HTTP 标头，特别是身份验证令牌，我们必须使用 `HTTP_` 字符串作为标头键的前缀。

在创建自己的测试时，应使用用户夹具中的身份验证令牌来对 API 进行身份验证。

### 测试成功执行操作

此测试检查操作是否成功执行。

```
public function testOperationSuccess(): void
{
    $client = static::createClient();
    $client->request('POST', '/api/v1/operation', [], [], [
        'HTTP_X-AUTH-TOKEN' => 'fYDg7nMhlvxAtL7KfBnS',
        'HTTP_Content-Type' => 'application/json'
    ], json_encode([
        'operation' => 'SendPayment',
        'data' => [
            'amount' => 13.69,
            'receiver' => 'hghgfyyfg'
        ]
    ]));
    $this->assertResponseStatusCodeSame(200);
}
```

该测试发送一个有效的令牌，操作负载包含一个现有操作，并且所需参数有效，因此测试应返回 `200 HTTP OK` 状态码。

### 测试成功后台操作

此测试检查操作是否成功发送到后台。对于此测试，您需要创建一个操作，其元数据属性将操作标记为后台操作。在第 3 章中，我们学习了如何通过将 `Background` 类实例传递给 `OperationMetadata` 属性的 `background` 参数来实现这一点。

```
public function testBackgroundOperation(): void
{
    $client = static::createClient();
    $client->request('POST', '/api/v1/operation', [], [], [
        'HTTP_X-AUTH-TOKEN' => 'fYDg7nMhlvxAtL7KfBnS',
        'HTTP_Content-Type' => 'application/json'
    ], json_encode([
        'operation' => 'SendPaymentBackground',
        'data' => [
            'amount' => 13.69,
            'receiver' => 'hghgfyyfg'
        ]
    ]));
    $this->assertResponseStatusCodeSame(202);
}
```

由于该操作被标记为后台操作，测试应返回 `202 ACCEPTED` 状态码。



### 测试无效的操作数据

在此案例中，我们将测试一项因发送数据无效而无法执行的操作。

```php
public function testOperationInvalidData(): void
{
$client = static::createClient();
$client->request('POST', '/api/v1/operation', [], [], [
'HTTP_X-AUTH-TOKEN' => 'fYDg7nMhlvxAtL7KfBnS',
'HTTP_Content-Type' => 'application/json'
], json_encode([
'operation' => 'SendPayment',
'data' => [
'amount' => 0,
'receiver' => 'hghgfyyfg'
]
]));
$this->assertResponseStatusCodeSame(400);
}
```

测试发送了一个无效的请求负载，因为 `amount` 参数必须大于 0。它应返回一个 *400 Bad request* 响应。此测试假设操作输入模型包含一个 `amount` 参数，并且该参数被 Symfony 的 `GreaterThan` 验证约束所包裹，该约束指定 `amount` 的值必须大于 0。让我们回顾一下第一章中创建的输入模型。

```php
readonly class SendPaymentInput
{
public function __construct(
#[Assert\NotBlank(message: 'Receiver cannot be empty')]
public string $receiver,
#[Assert\NotBlank(message: 'Amount cannot be empty')]
#[Assert\GreaterThan(0, message: 'Amount must be greater than 0')]
public float|int $amount,
public ?string $label = null
){}
}
```

如上所示，`amount` 参数既不能为空，也不能为 0。

### 测试不存在的操作

此项测试检查一项因操作不存在而无法执行的操作。

```php
public function testUnexistingOperation(): void
{
$client = static::createClient();
$client->request('POST', '/api/v1/operation', [], [], [
'HTTP_X-AUTH-TOKEN' => 'fYDg7nMhlvxAtL7KfBnS',
'HTTP_Content-Type' => 'application/json'
], json_encode([
'operation' => 'SendPayme',
'data' => [
'amount' => 0,
'receiver' => 'hghgfyyfg'
]
]));
$this->assertResponseStatusCodeSame(404);
}
```

测试发送了一个无效的操作名称，因此应返回 *404 Not Found* 响应码。

### 测试缺少操作名称

此项测试检查因未发送操作名称而导致操作请求验证失败的情况。

```php
public function testMissingOperationName(): void
{
$client = static::createClient();
$client->request('POST', '/api/v1/operation', [], [], [
'HTTP_X-AUTH-TOKEN' => 'zPLhK8iQqg9fg7u5jnp',
'HTTP_Content-Type' => 'application/json'
], json_encode([
'operation' => '',
'data' => [
'amount' => 13.69,
'receiver' => 'hghgfyyfg'
]
]));
$this->assertResponseStatusCodeSame(422);
}
```

测试发送了一个有效的数据负载；然而，操作名称缺失，这应导致返回 *422 UnprocessableEntityHttpException* 状态码。出现此响应的原因是操作名称的缺失会在 `ApiInput` 模型验证期间触发异常。具体来说，当请求到达控制器时，Symfony 的 `MapRequestPayload` 属性执行验证并随后抛出 `UnprocessableEntityHttpException` 异常。

### 测试无效的用户令牌

此项测试检查因发送的令牌无效而导致操作无法执行的情况。

```php
public function testInvalidAuth(): void
{
$client = static::createClient();
$client->request('POST', '/api/v1/operation', [], [], [
'HTTP_X-AUTH-TOKEN' => 'fYDg7AtL7KfBnS',
'HTTP_Content-Type' => 'application/json'
], json_encode([
'operation' => 'SendPayment',
'data' => [
'amount' => 13.69,
'receiver' => 'hghgfyyfg'
]
]));
$this->assertResponseStatusCodeSame(401);
}
```

测试发送了正确的负载，但 `X-AUTH-TOKEN` 头部包含了一个无效的令牌，因此响应码应为 *401 Unauthorized*。

### 测试未授权用户

此项测试检查 API 因用户没有正确角色而拒绝执行操作的情况。

```php
public function testUnauthorizedToExecuteOperation(): void
{
$client = static::createClient();
$client->request('POST', '/api/v1/operation', [], [], [
'HTTP_X-AUTH-TOKEN' => 'vWvn1GOu2Lx5foFgQrRp',
'HTTP_Content-Type' => 'application/json'
], json_encode([
'operation' => 'SendPayment',
'data' => [
'amount' => 13.69,
'receiver' => 'hghgfyyfg'
]
]));
$this->assertResponseStatusCodeSame(403);
}
```

测试发送了正确的负载和一个存在的用户令牌，但此用户没有正确的角色，因此返回的状态码应为 *403 Forbidden*。为了执行此测试，我们应该创建一个持有未被允许执行 `SendPayment` 操作角色的用户，并使用其令牌进行测试。

### 测试无效的上下文

此项测试尝试请求一个不允许在当前上下文中执行的操作。

```php
public function testInvalidContext(): void
{
$client = static::createClient();
$client->request('POST', '/api/v1/inventory-management/operation', [], [], [
'HTTP_X-AUTH-TOKEN' => 'fYDg7nMhlvxAtL7KfBnS',
'HTTP_Content-Type' => 'application/json'
], json_encode([
'operation' => 'SendPayment',
'data' => [
'amount' => 13.69,
'receiver' => 'hghgfyyfg'
]
]));
$this->assertResponseStatusCodeSame(403);
}
```

测试发送了正确的负载和一个有效的用户令牌，但由于 `SendPayment` 操作不能在 `inventory-management` 上下文中执行，因此应返回一个 *403 Forbidden* 响应码。

### 进行轻微重构

你应该注意到，所有测试都共享相同的代码来发送 API 请求。为了提高可读性并避免重复，我们可以创建一个私有方法来执行 API 请求。

```php
private function postOperation(array $payload, string $token, string $uri = '/api/v1/operation'): void
{
static::createClient()->request('POST', $uri, [], [], [
'HTTP_X-AUTH-TOKEN' => $token,
'HTTP_Content-Type' => 'application/json',
], json_encode($payload));
}
```

上述方法接收要发送的负载、认证令牌和要调用的 URI。`uri` 参数默认持有 `/api/v1/operation` 路径，因为它在大多数测试中都会用到。

然后我们可以在测试中使用这个方法。例如，在无效上下文测试中使用它。

```php
public function testInvalidContext(): void
{
$this->postOperation([
'operation' => 'SendPayment',
'data' => [
'amount' => 13.69,
'receiver' => 'hghgfyyfg'
]], 'vWvn1GOu2Lx5foFgQrRp');
$this->assertResponseStatusCodeSame(403);
}
```

如你所见，通过此更改，我们不再重复发送请求的逻辑。在每个测试中，我们只需要提供负载和认证令牌。

### 执行测试

要执行测试，我们需要从项目根目录运行以下命令：

```bash
bin/phpunit
```

这将执行你所有的测试。

### 结论

本章提供了关于如何使用 Symfony 为 API 实现有效测试的详细指南。通过各种示例，我们学会了验证操作的预期行为，从成功执行到错误处理和验证。这些测试不仅确保了 API 的功能性，还有助于在整个开发过程中维护代码质量。通过整合这些测试实践，开发者可以确保其 API 的健壮性和可靠性，从而提升最终用户的整体体验。

