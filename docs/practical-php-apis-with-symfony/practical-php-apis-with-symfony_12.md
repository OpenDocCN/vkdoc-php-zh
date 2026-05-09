# 2. 保护操作安全

在本节中，我们将使用 Symfony 的安全特性来保护端点（endpoints）和操作（operations）。为了保护端点，我们将使用 Symfony 的防火墙系统，通过它来建立用户认证机制。然后，我们将依靠 Symfony 的投票器（voters），根据特定规则来授予或拒绝操作的执行。

## 第一节 用户

在本节中，我们将创建用户实体，用于存储可以访问 API 的用户。然后，我们将启动在本书前言“第 3 节 前提条件”中配置的数据库，并向其中加载用户。

### 用户实体

我们将使用 Symfony 的 maker 命令来创建实体。为此，我们需要在项目根文件夹中执行以下命令：

```
bin/console make:entity
```

创建一个名为 `token` 的字符串字段，一个名为 `email` 的字符串字段，以及一个名为 `roles` 的 JSON 字段（JSON 字段需要支持它们的平台，例如 PostgreSQL 和 MySQL 5.7+）：

-   `token` 字段将存储令牌，该令牌将在本章后续部分用于对用户进行身份验证。
-   `email` 字段将用作标识符。
-   `roles` 字段将存储用户角色。我们将使用该字段，通过 Symfony 投票器来授权特定角色执行操作。

命令执行完毕后，您可以在 `src/Entity` 文件夹下看到一个新的用户类。它看起来应该像这样：

```
namespace App\Entity;
use App\Repository\UserRepository;
use Doctrine\ORM\Mapping as ORM;
#[ORM\Entity(repositoryClass: UserRepository::class)]
class User
{
#[ORM\Id]
#[ORM\GeneratedValue]
#[ORM\Column]
private ?int $id = null;
#[ORM\Column(length: 255)]
private ?string $token = null;
#[ORM\Column(length: 255)]
private ?string $email = null;
#[ORM\Column]
private array $roles = [];
public function getId(): ?int
{
return $this->id;
}
public function getToken(): ?string
{
return $this->token;
}
public function setToken(string $token): static
{
$this->token = $token;
return $this;
}
public function getEmail(): ?string
{
return $this->email;
}
public function setEmail(string $email): static
{
$this->email = $email;
return $this;
}
public function getRoles(): array
{
return $this->roles;
}
public function setRoles(array $roles): static
{
$this->roles = $roles;
return $this;
}
}
```

我们现在必须让我们的 `User` 实体实现 `Symfony UserInterface`。实现此接口是 Symfony 认证系统识别 `User` 实体为有效用户的必要条件。

```
namespace App\Entity;
use App\Repository\UserRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\UserInterface;
#[ORM\Entity(repositoryClass: UserRepository::class)]
class User implements UserInterface
{
// ... 其他方法在此
public function eraseCredentials()
{
}
public function getUserIdentifier(): string
{
return $this->getEmail();
}
}
```

由于我们的实体已经定义了 `getRoles` 方法（因为它是一个实体属性），我们需要定义接口要求的另外两个方法：

-   **eraseCredentials**：我们可以使用此方法来清除存储在此对象中的敏感信息。我们将其留空，因为我们没有存储诸如明文密码之类的临时敏感数据。
-   **getUserIdentifier**：我们使用此方法来返回用户标识符，即电子邮件。



### 向数据库填充用户数据

借助 Symfony Fixtures Bundle，我们可以创建充当数据库加载服务的类。这些服务必须位于 `src/DataFixtures` 文件夹下，并且必须继承 `Doctrine\Bundle\FixturesBundle\Fixture` 类。让我们创建一个 Fixture 类，用于向 `User` 实体填充用户数据：

```
namespace App\DataFixtures;
use App\Entity\User;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;
class UserFixtures extends Fixture
{
public function load(ObjectManager $manager): void
{
$user = new User();
$user->setToken('vWvn1GOu2Lx5foFgQrRp');
$user->setRoles(['ROLE_USER']);
$user->setEmail('spiderman@avengers.com');
$user2 = new User();
$user2->setToken('zPLhK8iQqg9fg7u5jnp');
$user2->setRoles(['ROLE_SENDER']);
$user2->setEmail('captain-america@avengers.com');
$manager->persist($user);
$manager->persist($user2);
$manager->flush();
}
}
```

`load` 方法接收 Doctrine 的 `ObjectManager` 服务作为参数。在该方法内部，我们创建了两个用户，一个具有 `ROLE_USER` 角色，另一个具有 `ROLE_SENDER` 角色。然后，我们使用 `ObjectManager` 的 `persist` 和 `flush` 方法将它们保存到数据库中。

要执行 fixture 并将用户数据加载到数据库，我们必须执行以下命令：

*   第一个命令只需要执行一次。它创建数据库模式，以便下一个命令能够成功加载数据行。如果你之前已创建数据库，则必须在再次创建之前使用 `doctrine:schema:drop` 命令删除它。在真实环境中，请考虑使用 Doctrine 迁移。

```
bin/console doctrine:schema:create
bin/console doctrine:fixtures:load
```

命令执行完毕后，我们的用户表就已经加载好了，可以开始下一节的内容了。

### 结论

在本节中，我们创建了 `User` 实体，它代表了存储在数据库中的、可以访问我们 API 的用户。在下一节中，我们将创建一个 Symfony 防火墙，它能够保护我们的 API 端点，并只允许经过身份验证的用户访问。

## 第 2 节：保护端点

为了确保只有经过身份验证的用户才能访问我们的 API 端点，我们将使用自定义的 [Symfony 身份验证器](https://symfony.com/doc/current/security/custom_authenticator.html)来实现基于令牌的身份验证。它将从请求头中提取令牌，进行验证，如果令牌有效，则用户身份验证成功；否则，将向客户端返回身份验证失败的信息。

### 操作身份验证器

该身份验证器服务继承了 [AbstractAuthenti​cator](https://github.com/symfony/symfony/blob/current/src/Symfony/Component/Security/Http/Authenticator/AbstractAuthenticator.php) 类，该类实现了 [AuthenticatorInt​erface](https://github.com/symfony/symfony/blob/current/src/Symfony/Component/Security/Http/Authenticator/AuthenticatorInterface.php) 接口。所有身份验证器都必须实现此接口。`AbstractAuthenticator` 定义了 `createToken` 方法，该方法应能满足大多数用例。下面你可以看到身份验证器的代码：

```
namespace App\Security;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\Exception\CustomUserMessageAuthenticationException;
use Symfony\Component\Security\Http\Authenticator\AbstractAuthenticator;
use Symfony\Component\Security\Http\Authenticator\Passport\Badge\UserBadge;
use Symfony\Component\Security\Http\Authenticator\Passport\Passport;
use Symfony\Component\Security\Http\Authenticator\Passport\SelfValidatingPassport;
class ApiTokenAuthenticator extends AbstractAuthenticator
{
public function supports(Request $request): ?bool
{
return $request->headers->has('X-AUTH-TOKEN') || preg_match('#\/api\/v1#', $request->getUri());
}
public function authenticate(Request $request): Passport
{
$apiToken = $request->headers->get('X-AUTH-TOKEN');
if (empty($apiToken)) {
throw new CustomUserMessageAuthenticationException('未提供 API 令牌');
}
return new SelfValidatingPassport(new UserBadge($apiToken));
}
public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
{
return null;
}
public function onAuthenticationFailure(Request $request, AuthenticationException $exception): JsonResponse
{
$data = [
'message' => strtr($exception->getMessageKey(), $exception->getMessageData())
];
return new JsonResponse($data, Response::HTTP_UNAUTHORIZED);
}
}
```

让我们逐步分析身份验证器的方法：

*   **supports**：它决定是否处理该身份验证器。如果请求头中包含 `X-AUTH-TOKEN`，或者请求 URI 匹配 `/api/v1` 模式，则执行此身份验证器。通过实现这种方法，我们可以有效处理不包含令牌的请求，同时又不会损害安全性。

*   **authenticate**：首先，它检查令牌是否为空。如果为空，则抛出异常。否则，它将返回一个 [SelfValidatingPa​ssport](https://symfony.com/doc/current/security/custom_authenticator.html%2523self-validating-passport)，并将 API 令牌作为徽章传递。返回后，安全组件将检查该令牌是否已注册。

在配置 User Provider 时，我们将了解安全层如何验证令牌。

*   **onAuthenticationSuccess**：仅在身份验证成功时执行。例如，可用于更新用户的有效登录计数器。在本例中，我们将其留空，以便流程继续执行。

*   **onAuthenticationFailure**：仅在身份验证失败时执行。在这种情况下，它会返回一个包含身份验证错误消息和 `401 Unauthorized` 状态码的 JSON 响应。

### 防火墙与用户提供器

防火墙和用户提供器都必须在 Symfony 的 `config/packages/security.yaml` 文件中定义。下面我们来展示一下：

#### 用户提供器

我们使用 [entity-user-provider](https://symfony.com/doc/current/security/user_providers.html%2523security-entity-user-provider) 作为用户提供器。

```
security:
# .......
providers:
users_provider:
entity:
class: App\Entity\User
property: token
```

此提供器使用 Doctrine 包在 `User` 实体中查找令牌与请求头中发送的令牌匹配的用户。在 `authenticate` 方法返回 `SelfValidatingPassport` 后，安全层将使用此提供器检查令牌是否有效。

#### 防火墙

防火墙将保护所有 URI 匹配 `^/api/v1` 模式的请求。它将使用 ApiTokenAuthenticator 对用户进行身份验证，并使用上一节配置的提供器加载用于比较的用户。

```
firewalls:
### 其他防火墙
api:
pattern: '^/api/v1'
provider: users_provider
stateless: true
custom_authenticators:
- App\Security\ApiTokenAuthenticator
```

`stateless` 参数指示防火墙应该是无状态的还是有状态的。当设置为 `true` 时，表示 Symfony 不会将会话中的用户身份验证信息存储起来。这在基于令牌进行 API 身份验证时非常有用，因为每次请求都会在请求头中发送令牌。

### 结论

通过使用 Symfony 身份验证器，我们实现了只有持有有效凭据的授权用户才能访问 API。在下一节中，我们将把安全措施从端点扩展到单个操作。我们将探索 Symfony Voter 的使用，这是 Symfony 中一个灵活的授权系统。



## 第三节 保护操作

现在，我们的端点已受到保护，只有经过身份验证的用户才能执行操作，因此下一个安全目标是启用对需要保护的操作的防护。例如，某些操作可能只允许具有特定角色的用户执行。为了实现这一目标，我们将使用 [Symfony 投票器](https://symfony.com/doc/current/security/voters.html)。

### 抽象操作投票器

首先，让我们编写一个抽象的操作投票器，其余的操作投票器都将继承它。

```php
use App\Api\ApiOperation;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;
abstract class OperationVoter extends Voter {
protected function supports(string $attribute, mixed $subject): bool
{
return ($attribute === 'PERFORM' && $subject instanceof ApiOperation);
}
}
```

`supports` 方法为所有继承它的投票器添加了通用逻辑。它检查属性参数是否持有值 `"PERFORM"`，以及主题参数是否为 `ApiOperation` 类的实例。考虑到当前面向操作的上下文，这两项检查是合理的：

-   我们始终检查属性是否持有 `"PERFORM"` 值，因为我们始终想要执行一个操作。
-   我们始终检查主题是否为 `ApiOperation` 实例，这样我们就能在子类中检查操作名称。

### 操作投票器

如上一节所述，操作投票器必须继承自抽象操作投票器，并实现授权逻辑。让我们为 `SendPayment` 操作创建一个投票器，确保经过身份验证的用户拥有 `"ROLE_SENDER"` 角色。

```php
use App\Security\Miscelanea\OperationNames;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
class SendPaymentVoter extends OperationVoter
{
protected function supports(string $attribute, mixed $subject): bool
{
if(parent::supports($attribute, $subject) && $subject->metadata->name === OperationNames::SendPayment->name){
return true;
}
return false;
}
protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
{
if(!in_array('ROLE_SENDER', $token->getUser()->getRoles())) {
return false;
}
return true;
}
}
```

`supports` 方法使用父类逻辑检查属性和主题实例，同时还检查操作名称是否为 `"SendPayment"`。如果 `supports` 方法返回 `"true"`，Symfony 将执行 `voteOnAttribute` 方法，并检查用户是否持有 `ROLE_SENDER` 角色。如果用户角色与所需角色不匹配，将返回 403 禁止访问响应，并停止操作执行。

可能存在一组操作共享相同授权规则的情况。在这种情况下，同一个投票器可以授权这组操作。为此，投票器的 `supports` 方法应检查请求的操作是否属于该集合。举个例子，假设我们有一个名为 `"ApprovePayment"` 的操作，并且 `"SendPayment"` 和 `"ApprovePayment"` 操作只能由 `ROLE_SENDER` 执行。我们可以在第 1 章的 `OperationNames` 枚举中创建一个名为 `getPaymentOperations` 的新静态方法。该方法应返回一个包含允许 `ROLE_SENDER` 执行的操作的数组，然后投票器应检查所请求的操作是否包含在该数组中。

```php
enum OperationNames
{
case SendPayment;
case ApprovePayment;
public static function getPaymentOperations(): array
{
return [
self::ApprovePayment->name,
self::SendPayment->name
];
}
}
```

如我们所见，`getPaymentOperations` 方法返回了允许执行的操作。现在在投票器中，我们应该在 `supports` 方法中使用它：

```php
use App\Security\Miscelanea\OperationNames;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
class SenderVoter extends OperationVoter
{
protected function supports(string $attribute, mixed $subject): bool
{
if(parent::supports($attribute, $subject) && in_array($subject->metadata->name, OperationNames::getPaymentOperations())){
return true;
}
return false;
}
protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
{
if(!in_array('ROLE_SENDER', $token->getUser()->getRoles())) {
return false;
}
return true;
}
}
```

只要用户的角色是 `"ROLE_SENDER"`，上述投票器就会授予对 `getPaymentOperations` 返回的数组中所有操作的访问权限。

在检查投票器之前，我们还需要进行一项额外的配置。由于我们不知道是否有投票器会被执行，如果没有任何投票器被执行，则被视为所有投票器都投了弃权票。当这种情况发生时，Symfony 会查阅安全配置参数 `"allow_if_all_abstain"`，该参数默认为 `"false"`。我们必须将此参数的值更改为 `"true"`，因为我们希望 Symfony 在没有任何投票器被执行时允许我们执行操作。如果您想查询有关访问决策策略的更多信息，可以阅读 [Symfony 安全文档](https://symfony.com/doc/current/security/voters.html%2523changing-the-access-decision-strategy)。

我们可以通过在 `config/packages/security.yaml` 文件中设置新值来更改此配置：

```yaml
security:
access_decision_manager:
strategy: unanimous
allow_if_all_abstain: true
```

### 处理访问被拒绝错误

Symfony 的安全 HTTP 组件通过异常监听器来监控安全错误。为了定制“操作被拒绝”的响应，我们需要创建一个自定义订阅者。该订阅者必须在默认的安全监听器之前执行。我们可以通过为自定义的 `AccessDeniedSubscriber` 分配更高的优先级来实现这一点。

```php
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\KernelEvents;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;
class AccessDeniedSubscriber implements EventSubscriberInterface
{
public static function getSubscribedEvents(): array
{
return [
KernelEvents::EXCEPTION => ['onException', 2]
];
}
public function onException(ExceptionEvent $event): void
{
$exception = $event->getThrowable();
if (!$exception instanceof AccessDeniedException) {
return;
}
$event->setResponse(new JsonResponse(
[ 'authorization' => $exception->getMessage()],

));
}
}
```

在上述订阅者中，有两个重要的方面需要我们注意：

-   订阅者的优先级设置为 2。这使其能够首先拦截“访问被拒绝”异常并定制响应。
-   订阅者会忽略并非 `AccessDeniedException` 实例的异常。这是因为这些其他错误将由第 1 章中详述的 `KernelSubscriber` 处理。



### 检查权限

在本节中，我们将修改 `ApiOperationHandler`，使其能够检查用户是否拥有执行操作的足够权限。

```
use App\Api\Collection\OperationCollection;
use App\Api\Input\ApiInput;
use App\Api\Output\ApiOutput;
use App\Validation\ValidationHandler;
use Symfony\Bundle\SecurityBundle\Security;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;
class ApiOperationHandler
{
public function __construct(
private readonly ValidationHandler $validationHandler,
private readonly OperationCollection $operationCollection,
private readonly Security $security
){ }
public function performOperation(ApiInput $apiInput): ApiOutput
{
$operation = $this->operationCollection->getOperation($apiInput->operation);
$isGranted = $this->security->isGranted('PERFORM', $operation);
if(!$isGranted){
throw new AccessDeniedException('Not allowed to perform this operation');
}
$inputData = ($operation->metadata->input)
? $this->validationHandler->deserializeAndValidate($apiInput->data, $operation->metadata->input)
: null
;
return $operation->handler->perform($inputData);
}
}
```

让我们重点关注 `getOperation` 行下方的变化。它使用了 `security.helper` Symfony 服务的 `isGranted` 方法来检查用户是否拥有权限。在内部，此方法会执行所有其 `supports` 方法返回 `true` 的投票器。由于投票器采用“一致同意”策略，如果任何投票器拒绝访问，则请求的操作将被撤销；否则，该操作将被授予权限。

-   为了使用 `security.helper` 服务，我们通过注入 `Symfony\Bundle\SecurityBundle\Security` 类（它是该服务的别名）来使用它。

### 结论

通过使用投票器，我们已经实现了对操作访问的细粒度控制。这确保了只有授权的用户才能执行特定操作，从而保护了 API 中的敏感数据和功能。在下一节中，我们将处理操作需要访问当前登录用户信息的情况。

## 第 4 节：从操作中访问已登录用户

可能存在需要访问已登录用户数据的操作。例如，`SendPayment` 操作在发送付款前可能需要获取当前用户的余额。因此，我们需要一种方法来实现：

-   确定操作是否需要访问用户信息
-   将用户传递给操作，以便操作可以使用它

### OperationRequiresUserInterface 接口

让我们先创建一个接口，该接口将由那些需要访问用户信息的操作来实现。

```
use Symfony\Component\Security\Core\User\UserInterface;
interface OperationRequiresUserInterface
{
public function setUser(UserInterface $user): void;
}
```

该接口声明了一个名为 `setUser` 的方法，`ApiOperationHandler` 将使用此方法将用户传递给操作。

### 将用户传递给操作

我们必须修改 `ApiOperationHandler` 的 `performOperation` 方法，使其能够检查操作是否需要用户信息并将其传递。

```
public function performOperation(ApiInput $apiInput): ApiOutput
{
$operation = $this->operationCollection->getOperation($apiInput->operation);
$isGranted = $this->security->isGranted('PERFORM', $operation);
if(!$isGranted){
throw new AccessDeniedException('Not allowed to perform this operation');
}
$inputData = ($operation->metadata->input)
? $this->validationHandler->deserializeAndValidate($apiInput->data, $operation->metadata->input)
: null
;
if($this-security-getToken()?-getUser() && $operation->handler instanceof OperationRequiresUserInterface) {
$operation->handler->setUser($this->security->getToken()->getUser());
}
$operation->handler->perform($inputData);
}
```

在验证操作数据之后，`performOperation` 函数会检查操作对象是否是 `OperationRequiresUserInterface` 的实例，也就是说，该操作实现了这个接口。如果是，处理器就会使用 `setUser` 方法将用户传递给该操作。

### 实现 OperationRequiresUserInterface 接口

让我们让 `SendPayment` 操作实现 `OperationRequiresUserInterface` 接口。

```
use App\Api\Attribute\OperationMetadata;
use App\Api\Input\Operation\SendPaymentInput;
use App\Api\Output\ApiOutput;
use App\Security\Miscelanea\OperationNames;
use Symfony\Component\Security\Core\User\UserInterface;
#[OperationMetadata(
name: OperationNames::SendPayment->name,
input: SendPaymentInput::class,
)]
class SendPaymentOperation implements OperationInterface, OperationRequiresUserInterface
{
private ?UserInterface $user = null;
/**
* @param SendPaymentInput $data
*/
public function perform(mixed $data): ApiOutput
{
/**
* 发送付款的业务逻辑将在此处编写
*/
return new ApiOutput([
'id' => '77fg76hf'
], 202);
}
public function setUser(?UserInterface $user): void
{
$this->user = $user;
}
}
```

如您所见，`setUser` 方法使用作为参数传递的用户来初始化操作的 `user` 属性，从那一刻起，该用户便可在操作中使用。

### 结论

在本节中，我们设计了一种方法，允许操作在需要时访问当前登录的用户。在下一节中，我们将学习如何根据用户角色验证输入请求。

## 第 5 节：根据用户角色验证操作

在上一章中，我们根据操作输入类的属性约束来验证操作。在本章中，我们将更进一步，也根据用户角色来验证操作。

### 在输入模型中使用验证组

Symfony [验证组](https://symfony.com/doc/current/validation/groups.html) 是一个很好的功能，可以仅针对对象的某些约束进行验证。让我们从以下假设开始：我们想要创建一个用于创建项目的操作。这些项目始终需要名称和描述。项目的开始日期和结束日期仅对 `ROLE_PROJECT_MANAGER` 用户是必需的，因此：

-   **ROLE_PROJECT_MANAGER**：他们必须发送所有参数。
-   **ROLE_DEVELOPER**：他们只需发送名称和描述。

下面您可以看到输入类：

```
use Symfony\Component\Validator\Constraints\IsNull;
use Symfony\Component\Validator\Constraints\Date;
use Symfony\Component\Validator\Constraints\NotBlank;
readonly class CreateProjectInput
{
public function __construct(
#[NotBlank(message: '名称不能为空', groups: ['ROLE_PROJECT_MANAGER', 'ROLE_DEVELOPER'])]
public string $name,
#[NotBlank(message: '描述不能为空', groups: ['ROLE_PROJECT_MANAGER', 'ROLE_DEVELOPER'])]
public string $description,
#[NotBlank(message: '开始日期不能为空', groups: ['ROLE_PROJECT_MANAGER'])]
#[IsNull(message: '开始日期必须为空', groups: ['ROLE_DEVELOPER'])]
#[Date(message: '开始日期必须是一个有效的日期', groups: ['ROLE_PROJECT_MANAGER'])]
public string $startDate,
#[NotBlank(message: '结束日期不能为空', groups: ['ROLE_PROJECT_MANAGER'])]
#[IsNull(message: '结束日期必须为空', groups: ['ROLE_DEVELOPER'])]
#[Date(message: '结束日期必须是一个有效的日期', groups: ['ROLE_PROJECT_MANAGER' ])]
public string $endDate,
){}
}
```

正如您所看到的，该模型规定名称和描述对于管理员和开发人员都是必需的，但开始和结束日期只能由项目经理指定。实际上，[`IsNull`](https://symfony.com/doc/current/reference/constraints/IsNull.html) 约束指定了开发人员不能填写这些属性。

-   我们使用 `IsNull` 约束来避免由于尝试在可为空的日期字段中存储空字符串而导致数据库错误。



### 指定操作使用验证组

现在，我们需要指定操作使用验证组，以便处理器能够据此做出响应。为此，我们将在 `OperationMetadata` 属性中添加一个名为 `validateByRole` 的新参数，使 `ApiOperationHandler` 能够判断是否使用验证组。

```
#[Attribute(\Attribute::TARGET_CLASS)]
class OperationMetadata
{
public function __construct(
public readonly string $name,
public readonly ?string $version = null,
public readonly ?string $input = null,
public readonly ?bool $validateByRole = null
){ }
}
```

### 调整验证处理器以使用验证组

如果你还记得第 1 章“第 5 节：验证操作”中的内容，我们创建了一个名为 `ValidationHandler` 的类，用于根据操作输入模型来反序列化和验证接收到的操作数据。我们需要修改这个类，使其能够在需要时应用验证组。

让我们再次查看 `ValidationHandler` 类：

```
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
public function deserializeAndValidate(array|string $payload, string $className, ?array $groups)
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
$errors = $this->validator->validate($object, null, $groups);
if(count($errors) > 0) {
throw new ValidationFailedException(null, $errors);
}
return $object;
}
}
```

如你所见，我们只做了两处修改：

- 在 `deserializeAndValidate` 方法中添加了第三个参数，以便能够传入要应用的验证组。该参数可能为 null，因为可能不需要应用验证组。
- 将验证组传递给了 Symfony 验证服务。对于不需要应用验证组的情况，Symfony 也接受将 groups 参数设为 null。

### 调整 ApiOperationHandler 以传递验证组

为了完成本部分内容，我们必须指示 `ApiOperationHandler`，使其能够判断是否使用验证组。

```
public function performOperation(ApiInput $apiInput): ApiOutput
{
$operation = $this->operationCollection->getOperation($apiInput->operation);
$isGranted = $this->security->isGranted('PERFORM', $operation);
if(!$isGranted){
throw new AccessDeniedException('Not allowed to perform this operation');
}
$validationGroups = ($operation->metadata->validateByRole) ? $this->security->getUser()->getRoles() : null;
$inputData = ($operation->metadata->input)
? $this->validationHandler->deserializeAndValidate($apiInput->data, $operation->metadata->input, $validationGroups)
: $apiInput->data
;
return $operation->handler->perform($inputData);
}
```

如果我们查看 `ApiOperationHandler performOperation` 方法，它会检查 `validateByRole` 的值。如果为 true，它会为 `$validationGroups` 变量分配一个数组，该数组包含组名（与用户角色对应）。否则，它会赋值为 null。然后，`$validationGroups` 变量作为第三个参数传递给 `deserializeAndValidate` 方法。

### 结论

我们已经探讨了如何使用 Symfony 验证组，根据经过身份验证的用户角色来验证操作输入属性的子集。这为我们提供了灵活性，可以针对每个用户角色的特定权限和职责实施不同的验证规则。

在下一节中，我们将学习如何使用速率限制器来控制每个用户发起请求的频率，以防止滥用并确保系统稳定性。

## 第 6 节：控制请求

为了防止拒绝服务攻击，我们将依赖 Symfony 的速率限制器组件来控制到达 API 的请求数量。

### 令牌桶策略

将令牌桶想象成一个容纳有限数量令牌的容器。在对 API 进行速率限制请求的上下文中，令牌桶策略的工作原理如下：

- 初始时，令牌桶被填充了一定数量的令牌。
- 每次向 API 发出请求时，系统会检查桶中是否有足够的令牌。
- 如果有足够的令牌可用，请求继续执行，系统会从桶中移除一个令牌。
- 如果没有足够的令牌可用，则请求会被延迟，直到有更多令牌可用。

随着时间的推移，令牌桶会以一定的速率重新填充，向桶中添加新的令牌。

### 配置令牌桶策略

速率限制器的配置位于文件 `config/packages/framework.yaml` 中的 `rate_limiter` 部分。

```
framework:
rate_limiter:
api:
policy: 'token_bucket'
limit: 5000
rate: { interval: '20 minutes', amount: 200 }
```

上述配置允许最多 5000 个令牌，并且在发出第一个 HTTP 请求后，每 20 分钟添加 200 个令牌。`policy` 键表示我们正在使用 `token_bucket` 策略。由于我们使用了服务的自动装配，Symfony 会创建一个名为 `apiLimiter` 的绑定变量，该变量将持有我们的速率限制器服务。让我们在下一节中看看如何使用它。



### 使用限流器

我们将在第 1 章创建的订阅者中使用 `apiLimiter` 服务。具体来说，我们将监听 Symfony 的 `KernelRequest` 事件，以检查是否可以继续处理请求，即是否有可用的令牌。这样，我们就能在请求到达控制器之前检查限制。

```
use App\Exception\InvalidPayloadException;
use App\Exception\NotFoundOperationException;
use App\Validation\ValidationHandler;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\Event\RequestEvent;
use Symfony\Component\HttpKernel\KernelEvents;
use Symfony\Component\RateLimiter\RateLimiterFactory;
use Symfony\Component\Validator\Exception\ValidationFailedException;
use Symfony\Component\RateLimiter\Exception\RateLimitExceededException;

class KernelSubscriber implements EventSubscriberInterface
{
    public function __construct(
        private readonly ValidationHandler $validationHandler,
        private readonly RateLimiterFactory $apiLimiter
    ){}

    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::EXCEPTION => 'onException',
            KernelEvents::REQUEST => 'onKernelRequest'
        ];
    }

    public function onKernelRequest(RequestEvent $event): void
    {
        $limiter = $this->apiLimiter->create($event->getRequest()->getClientIp());
        $limiter->consume()->ensureAccepted();
    }

    public function onException(ExceptionEvent $event): void
    {
        $exception = $event->getThrowable();
        if($exception instanceof ValidationFailedException){
            $errors = [];
            foreach ($exception->getViolations() as $violation) {
                $errors[$violation->getPropertyPath()] = $violation->getMessage();
            }
            $event->setResponse(new JsonResponse([
                'errors' => $errors
            ], 400));
            return;
        }
        if($exception instanceof RateLimitExceededException) {
            $event->setResponse(new JsonResponse([
                'error' => 'Api Rate limit exceeded'
            ], 429));
            return;
        }
        if($exception instanceof InvalidPayloadException) {
            $event->setResponse(new JsonResponse([
                'error' => $exception->getMessage()
            ], 400));
            return;
        }
        if($exception instanceof NotFoundOperationException){
            $event->setResponse(new JsonResponse([
                'errors' => [
                    'operation' => $exception->getMessage()
                ]
            ], 400));
            return;
        }
    }
}
```

`onKernelRequest` 方法通过使用 `apiLimiter` 服务获取限流器，并检查请求是否被接受。如果不接受，则会抛出 `RateLimitExceededException`。我们还在 `onException` 方法中检查此异常，并向客户端返回 *429 Too Many Requests* HTTP 状态码。

