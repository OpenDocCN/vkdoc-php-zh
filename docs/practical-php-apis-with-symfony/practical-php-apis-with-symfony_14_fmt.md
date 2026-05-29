# 3. API 操作的背景执行

到目前为止，我们学习了如何通过单个端点管理操作、如何使用 Symfony 防火墙保护端点，以及如何使用 Symfony voter 授予或拒绝访问操作的权限。在本章中，我们将学习如何在后台执行操作，这样客户端就无需等待操作执行完成。这对于可能需要一段时间才能执行的操作非常有用。我们的 `SendPayment` 操作就是一个很好的例子，因为它可能需要连接到支付提供商来发送资金。由于我们不知道这种连接可能需要多长时间，因此最好在后台执行它，并在操作完成后向客户端发送通知（可以是电子邮件、短信或推送通知等）。

## 第 1 节 将操作标记为后台执行

我们在第 1 章中创建了 `OperationMetadata` 属性，用于存储与操作相关的以下数据：

- 操作名称

- 类名，包含执行操作所需的参数

- 是否需要在验证过程中将角色作为组传递

现在让我们向属性类添加一个新参数，该参数将指示操作是否必须在后台执行。

```php
readonly class Background
{
    /**
     * 后台操作执行前的延迟时间（秒）
     */
    public function __construct(
        public ?int $delay = null
    ){ }
}

#[\Attribute(\Attribute::TARGET_CLASS)]
class OperationMetadata
{
    public function __construct(
        public readonly string $name,
        public readonly ?string $input = null,
        public readonly ?bool $useValidationGroups = null,
        public readonly ?Background $background = null
    ){ }
}
```

由于后台执行可能会有延迟，我们首先创建一个包含可为 `null` 的 `delay` 参数的 `Background` 类。如果提供了该参数，操作将在后台执行并增加额外的延迟。`OperationMetadata` 现在包含一个新的 `Background` 类型参数。如果提供了该参数，操作必须发送到后台执行，而不是实时执行。

现在，如果我们想将 `SendPayment` 操作标记为后台执行，只需向属性添加 `background` 参数即可。

```php
#[OperationMetadata(
    name: OperationNames::SendPayment->name,
    input: SendPaymentInput::class,
    useValidationGroups: false,
    background: new Background(300)
)]
class SendPaymentOperation implements OperationInterface
{
    // 其余方法
}
```

在上面的示例中，我们指明 `SendPayment` 操作将在后台执行，并且执行将延迟 300 秒（5 分钟）。

### 结论

我们通过添加一个新参数扩展了现有的 `OperationMetadata` 属性，该参数允许我们指示特定操作是否应在后台执行。通过设置此标志，我们表明操作的执行可以是异步的，并且与用户的即时请求解耦。在下一节中，我们将探讨如何创建操作消息及其对应的处理程序，以及使用 Symfony 的 `AsMessageHandler` 属性将它们连接起来的魔力。

## 第 2 节 操作消息和处理程序

我们将使用 [Symfony messenger 组件](https://symfony.com/doc/current/messenger.html)在后台执行操作。首先，我们必须创建一个消息类和一个消息处理程序。

### 创建操作消息

```php
use Symfony\Component\Security\Core\User\UserInterface;

class OperationMessage
{
    public function __construct(
        public readonly string $operation,
        public readonly mixed $operationInput,
        public readonly ?string $userIdentifier
    ){ }
}
```

消息类的构造函数中包含执行操作所需的参数：操作名称、操作数据和用户标识符（如果操作需要）。现在让我们编写消息处理程序。

-   正如我们将在下一节中看到的，我们将使用 User 实体中的 `getUserIdentifier` 方法来设置 `OperationMessage` 用户参数的值。此方法返回用户的电子邮件。

### 创建操作处理器

现在，我们需要一个处理器来管理 `OperationMessage` 消息。

```php
use App\Api\Collection\OperationCollection;
use App\Api\Operation\OperationRequiresUserInterface;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;
use Doctrine\ORM\EntityManagerInterface;

#[AsMessageHandler]
class OperationMessageHandler
{
    public function __construct(
        private readonly OperationCollection $operationCollection,
        private readonly EntityManagerInterface $em
    ){ }

    public function __invoke(OperationMessage $message): void
    {
        $apiOperation = $this->operationCollection->getOperation($message->operation);

        if($message->user && $apiOperation->handler instanceof OperationRequiresUserInterface) {
            $user = $this->em->getRepository(User::class)->findOneByEmail($message->userIdentifier);
            if(!$user){
                throw new \InvalidArgumentException("Unknown user while executing operation: " . $message->operation);
            }
            $apiOperation->handler->setUser($user);
        }

        $apiOperation->handler->perform($message->operationInput);
    }
}
```

处理器的 `__invoke` 函数与 `ApiOperationHandler` 的行为方式类似。它从集合中获取要执行的操作，检查操作是否需要设置用户，如果需要则进行设置。最后，它执行该操作。如果需要用户，处理器会先从数据库中检索用户，然后将用户传递给操作。由于用户实体中的 `getUserIdentifier()` 方法返回用户的电子邮件，我们在后台处理器中使用 `findOneByEmail()` 从数据库中检索用户。

### 结论

我们已经创建了操作消息及其对应的处理器，这样，在将操作消息分发到后台后，它就能被成功处理。在下一节中，我们将弥合现有 `ApiOperationHandler` 之间的差距，以根据先前在 `OperationMetadata` 属性中引入的标志来检查操作是否需要后台处理。

## 第三节：将操作发送到后台

到目前为止，我们已经建立了将操作标记为后台操作以及处理这些已入队操作（消息及其处理器）的机制。现在，我们需要让 `ApiOperationHandler` 能够检查某个操作是否必须发送到后台。让我们修改它以添加此功能。

```php
use App\Api\Collection\OperationCollection;
use App\Api\Input\ApiInput;
use App\Api\Operation\OperationRequiresUserInterface;
use App\Api\Output\ApiOutput;
use App\Messenger\OperationMessage;
use App\Validation\ValidationHandler;
use Symfony\Bundle\SecurityBundle\Security;
use Symfony\Component\Messenger\MessageBusInterface;
use Symfony\Component\Messenger\Stamp\DelayStamp;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;

class ApiOperationHandler
{
    public function __construct(
        private readonly ValidationHandler $validationHandler,
        private readonly OperationCollection $operationCollection,
        private readonly Security $security,
        private readonly MessageBusInterface $bus
    ){ }

    public function performOperation(ApiInput $apiInput): ApiOutput
    {
        $operation = $this->operationCollection->getOperation($apiInput->operation);
        $isGranted = $this->security->isGranted('PERFORM', $operation);
        if(!$isGranted){
            throw new AccessDeniedException('Not allowed to perform this operation');
        }

        $validationGroups = ($operation->metadata->useValidationGroups) ? [$this->security->getUser()->getRoles()] : null;
        $inputData = ($operation->metadata->input)
            ? $this->validationHandler->deserializeAndValidate($apiInput->data, $operation->metadata->input, $validationGroups)
            : null
        ;

        $userIdentifier = null;
        $user = null;
        if($operation->handler instanceof OperationRequiresUserInterface) {
            $user = $this->security->getToken()->getUser();
            $userIdentifier = $user->getUserIdentifier();
        }

        if($operation->metadata->background) {
            // 延迟时间乘以 1000，因为 DelayStamp 期望延迟时间以毫秒为单位
            $stamps = ($operation->metadata->background->delay) ? [new DelayStamp($operation->metadata->background->delay * 1000)] : [];
            $this->bus->dispatch(new OperationMessage($operation->metadata->name, $inputData, $userIdentifier), $stamps);
            return new ApiOutput(['status' => 'queued'], 202);
        }

        if($user){
            $operation->handler->setUser($user);
        }

        return $operation->handler->perform($inputData);
    }
}
```

让我们检查一下我们引入的更改：

- 构造函数注入了 `MessageBusInterface` 服务，这使我们能够将操作发送到后台（我们将在下一章学习如何将消息路由到相应的传输器）。

- 在确保操作可以被执行、其数据有效以及是否需要用户之后，处理器通过检查操作元数据是否包含 `background` 参数（即不为 `null`）来确定操作是否必须发送到后台。如果是，处理器将按如下方式继续：

  - 它通过查看 `background` 的 `delay` 值来检查是否必须添加延迟。如果指定了延迟，它将创建一个 `DelayStamp`，并传入指定的延迟时间（我们将延迟时间乘以 1000，因为 `DelayStamp` 构造函数接收的延迟时间是以毫秒为单位的）。

  - 它创建操作消息并使用总线服务分发它。

  - 它返回一个带有 *202 Accepted* 状态码的相应输出，这表示请求的操作已被接受，稍后将执行。

由于 `ApiOperationHandler` 在将操作加入队列之前验证了操作参数并检查了授权规则，因此我们不必在消息处理器中再次进行这些检查。

### 结论

我们已经探讨了如何修改处理器以识别被标记为后台处理的操作，并如何利用 Symfony Messenger 总线服务将其入队。然而，确保高效的后台处理需要正确的配置。在下一节中，我们将学习如何配置系统，将后台操作执行消息路由到指定的传输器。

## 第四节：路由操作

到目前为止，如果我们发送一个请求来执行标记为后台的操作，它不会被加入队列。这是因为我们需要配置最重要的部分，即，将操作消息路由到一个[传输器](https://symfony.com/doc/6.4/messenger.html%23routing-messages-to-a-transport)。

传输器配置位于 `config/packages/messenger.yaml` 文件中。下面，我们将展示如何将我们在“Bootfront 关键事项”的第三节“先决条件”中安装的 Redis 服务器配置为传输器，以及如何指示 Messenger 将操作消息路由到该传输器。

```yaml
framework:
    messenger:
        transports:
            operations:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
        routing:
            App\Messenger\OperationMessage: 'operations'
```

如我们所见， “transports” 部分显示了可用的传输器。在这个案例中，只有一个创建的 Redis 传输器，它使用的 `dsn` 来自 `.env` 文件。`routing` 部分指定了哪些消息被路由到哪些传输器。在这个案例中， `OperationMessage` 将被路由到 `operations` 传输器。

### 消费消息

当传输就绪后，发送至后台的操作不会立即执行。相反，它们会被加入队列，等待一个名为 `worker` 的专用进程来处理。这些 worker 会在后台持续运行，等待队列中出现新的操作。Symfony messenger 提供了一个名为 `messenger:consume` 的实用命令来启动 worker 并使其持续消费消息。让我们看看如何从 `operations` 传输中消费消息：

```
bin/console messenger:consume operations
```

我们不应让消费者永久运行，因为它们可能会随时间推移消耗更多内存。为避免这种情况，我们可以使用以下命令选项来控制 worker 的活动时长：

- `time-limit`：用于指定 worker 将活动多长时间（以秒为单位）。

- `message-limit`：用于指定在退出前将消费多少条消息。

```
bin/console messenger:consume operations --time-limit=3600
```

上述命令启动一个 worker，它将持续从 `operations` 传输中消费消息一小时。一小时后它将被终止。

### 管理 Worker

由于我们的 worker 应在一定时间后被终止，我们应该使用一个工具来重新启动 worker，以便它们持续从队列中消费消息。[Supervisor](https://symfony.com/doc/6.4/messenger.html%2523supervisor-configuration) 是一个易于使用的工具，可以帮助实现这一点。你可以通过以下命令安装 supervisor：

```
sudo apt-get install supervisor
```

安装完成后，我们必须编写所需的配置，以便我们的操作 worker 能在每次重启后启动。Supervisor 的配置通常存放在 `/etc/supervisor/conf.d` 目录中。我们可以在该文件夹中创建一个名为 `operations.conf` 的文件，配置内容如下：

```
/etc/supervisor/conf.d/operations.conf
[program:messenger-consume]
command=php /path/to/your/app/bin/console messenger:consume operations --time-limit=3600
user=
numprocs=4
startsecs=0
autostart=true
autorestart=true
startretries=5
process_name=%(program_name)s_%(process_num)02d
```

让我们逐项回顾上述配置：

- `command`：这是 Supervisor 将处理的命令，在我们的案例中是操作执行 worker。

- `user`：执行命令的操作系统用户。

- `numprocs`：将执行的进程（worker）数量。

- `autostart`：指示命令是否自动初始化（我们必须将其设置为 true，因为我们希望 worker 自动启动）。

- `autorestart`：指示命令是否自动重启（我们必须将其设置为 true，因为我们希望 worker 在每 3600 秒后自动重启）。

- `startretries`：指示 Supervisor 在失败后尝试启动进程的次数。

- `process_name`：指示用于构建进程名称的格式。

### 检查传输是否正常工作

为了检查执行某个操作是否被放入队列以待后续处理，我们将修改 `SendPayment` 元数据属性，将其标记为后台操作。正如我们在“第 1 节：将操作标记为后台”中所学的，我们必须将操作元数据 `background` 参数设置为一个新的 `Background` 类实例。

现在，让我们请求操作端点来执行一个 `SendPayment` 操作。由于它被标记为后台操作，端点应使用 Redis 传输将其加入队列，并返回 202 Accepted 状态码。

```
请求：
curl --request POST \
--url http://127.0.0.1:8000/api/v1/operation \
--header 'Content-Type: application/json' \
--header 'X-AUTH-TOKEN: zPLhK8iQqg9fg7u5jnp' \
--data '{
"operation" : "SendPayment",
"data" : {
"receiver" : "8528f744fg788",
"amount" : 56.87
}
}'
响应：
202 Accepted
{
"status": "queued"
}
```

至此，操作已被加入队列，但在 worker 启动之前不会被执行。在启动 worker 之前，让我们确保操作已成功路由到 Redis 传输。为此，我们可以登录到 Redis 服务器并检查操作流的内容。我们有两种方式登录 Redis 服务器：

- 通过访问 docker 容器，然后使用 `redis-cli` 命令进入 Redis shell（该容器已预装 `redis-cli` 命令）。

- 直接通过链接端口访问（需安装 `redis-cli`）。

我们使用第一种方式。要访问容器，打开终端并执行以下命令：

```
docker exec -it  sh
```

进入容器提示符后，我们需要登录到 Redis 服务器。这只需运行下面的命令：

```
redis-cli
```

然后，当我们登录到 Redis 服务器后，可以运行以下命令来检查操作流的长度：

```
xlen operations
```

注意：`operations` 指的是在 messenger 传输配置中定义的流名称。

上述命令应返回长度 1，这告诉我们操作已成功路由。最后，我们只需要启动一个 worker 并确保其处理该操作。让我们执行 `messenger:consume` 命令并观察操作如何被执行。

```
bin/console messenger:consume operations -vv
```

执行 worker 后，我们应该看到以下输出：

```
[OK] Consuming messages from transport "operations".
// The worker will automatically exit once it has received a stop signal via the messenger:stop-workers command.
// Quit the worker with CONTROL-C.
[info] Received message App\Messenger\OperationMessage
[info] Message App\Messenger\OperationMessage handled by App\Messenger\OperationMessageHandler::__invoke
[info] App\Messenger\OperationMessage was handled successfully (acknowledging to transport).
```

正如我们所见，`info` 日志告诉我们操作已成功处理。

### 重试失败的操作

默认情况下，如果处理消息失败三次，Symfony 将丢弃该消息。你可以使用 `max_retries` 选项更改这些参数，如下所示：

```
framework:
messenger:
transports:
operations:
dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
retry_strategy
max_retries 2
```

如上配置所示，我们将指示 Symfony messenger 重试 2 次，而不是 3 次。为避免失败时永久丢失消息，我们必须配置一个失败传输。当 Symfony messenger 检测到配置了失败传输时，它会将失败的消息路由到该传输，以便在导致失败的错误得到修复后稍后重试它们。让我们看看如何配置：

```
framework:
messenger:
failure_transport: failed
transports:
operations:
dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
retry_strategy
max_retries 2
failed 'doctrine://default?table_name=failed_operations'
```

现在，当一个操作处理失败后，Symfony 不会丢弃它，而是将失败的消息路由到 `failed` 传输。`failed` 传输使用 doctrine 作为后端，因此操作消息将存储在我们数据库的名为 `failed_operations` 的表中。之后，我们可以像使用任何其他传输一样消费这些消息。

- Symfony 提供了一组命令来从失败传输中重试消息。你可以在此处阅读更多内容：[`https://symfony.com/doc/current/messenger.html`](https://symfony.com/doc/current/messenger.html)。

## 结论

在本节中，我们学习了如何设置正确的配置，将操作消息路由到 `Redis` 传输层。我们还探讨了如何通过检查对应的 `Redis` 键来验证操作是否成功路由，并最终运行了一个工作进程来执行该操作。此外，我们还展示了如何使用 `Supervisor` 来管理工作进程并在其崩溃后重启。

下一节，我们将学习如何根据优先级创建独立的队列来路由操作。

## 第 5 节 操作优先级排序

在某些情况下，需要发送到后台处理的操作量可能非常大。这时，根据优先级等因素将操作量按队列进行分段管理会显得很实用。这就是我们本节要实现的内容。为此，我们将依赖 [Symfony 优先级传输](https://symfony.com/doc/current/messenger.html) 这一特性。

### 定义优先级

为了定义可用的操作优先级，我们先创建一个枚举来存储它们：

```php
enum Priority
{
    case NORMAL;
    case HIGH;
}
```

为简单起见，该枚举包含两个表示优先级的选项：`NORMAL` 和 `HIGH`。

### 修改元数据属性

现在，我们需要在 `Background` 类属性中新增一个参数，以便用指定的优先级来标记操作。

```php
class Background
{
    public function __construct(
        public readonly ?int $delay = null,
        public readonly ?Priority $priority = Priority::NORMAL
    ){ }
}
```

如你所见，`Background` 类的构造函数将 `Priority` 枚举值作为第二个构造参数。默认情况下，所有后台操作都拥有 `NORMAL` 优先级。

### 为高优先级操作创建消息

`NORMAL` 优先级的操作将继续使用现有的 `OperationMessage` 消息类。回顾“第 4 节 路由操作”，分派一个 `OperationMessage` 后，它会被路由到 `operations` 传输层。为了区分 `NORMAL` 和 `HIGH` 操作，我们必须将它们路由到不同的传输层。因此，首先我们需要为 `HIGH` 优先级的消息创建一个新的消息类：

```php
use App\Messenger\OperationMessage;

class HighPriorityOperationMessage extends OperationMessage
{
}
```

由于 `NORMAL` 优先级和 `HIGH` 优先级操作拥有相同的参数，我们可以简单地继承现有的 `OperationMessage` 类。

### 为每种优先级创建传输层

由于我们希望将不同优先级的操作分离到不同的传输层，需要对传输层配置进行一些修改。

```yaml
framework:
    messenger:
        transports:
            operations:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
                    group: 'normal'
            operations_high:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
                    group: 'high'
        routing:
            App\Messenger\OperationMessage: 'operations'
            App\Messenger\HighPriorityOperationMessage: 'operations_high'
```

让我们看一下这个新配置。

*   这里有两条传输层：

    *   `operations`：用于路由 `NORMAL` 优先级的操作

    *   `operations_high`：用于路由 `HIGH` 优先级的操作

*   每个选项都设置了 `group` 参数。在使用 Symfony 的 `redis-transport` 时，你可以在 `options` 部分使用 `group` 参数来设置优先级。Symfony 在内部利用 `Redis` 的流组和消费者特性来实现此功能。

*   每条消息将被路由到其对应的传输层。

### 修改消息处理器

消息处理器需要能够处理这两种类型的消息。Symfony Messenger 允许我们通过使用方法级别的 `AsMessageHandler` 属性来实现这一点，而不是在类级别使用。

```php
class OperationMessageHandler
{
    public function __construct(
        private readonly OperationCollection $operationCollection,
        private readonly EventDispatcherInterface $eventSubscriber,
        private readonly EntityManagerInterface $em
    ){ }

    #[AsMessageHandler]
    public function handleNormalPriorityOperations(OperationMessage $message): void
    {
        $this->handleMessage($message);
    }

    #[AsMessageHandler]
    public function handleHighPriorityOperations(HighPriorityOperationMessage $message): void
    {
        $this->handleMessage($message);
    }

    private function handleMessage(OperationMessage|HighPriorityOperationMessage $message): void
    {
        $apiOperation = $this->operationCollection->getOperation($message->operation);
        $user = null;
        if($message->user && $apiOperation->handler instanceof OperationRequiresUserInterface) {
            $user = $this->em->getRepository(User::class)->findOneByEmail($message->user);
            $apiOperation->handler->setUser($user);
        }
        $apiOperation->handler->perform($message->operationInput);
    }
}
```

如上代码所示，我们创建了一个名为 `handleNormalPriorityOperations` 的方法来处理 `NORMAL` 操作，以及另一个名为 `handleHighPriorityOperations` 的方法来处理 `HIGH` 操作。由于这两个方法执行相同的代码，我们将处理逻辑移到了一个名为 `handleMessage` 的私有方法中。

### 根据优先级分派消息

这里，我们需要回顾一下前面的章节。在那里，我们修改了 `ApiOperationHandler` 类来检查是否需要将操作执行放入队列。现在，我们需要再次修改它，以检查操作优先级并使用相应的消息类。

```php
if($operation->metadata->background) {
    $stamps = ($operation->metadata->background->delay)
        ? [new DelayStamp($operation->metadata->background->delay * 1000)]
        : []
    ;
    $message = ($operation->metadata->background->priority === Priority::NORMAL)
        ?  new OperationMessage($operation->metadata->name, $inputData, $userIdentifier)
        :  new HighPriorityOperationMessage($operation->metadata->name, $inputData, $userIdentifier)
    ;
    $this->bus->dispatch($message, $stamps);
    return new ApiOutput(['status' => 'queued'], 202);
}
```

如上所示，在检查了元数据的延迟参数之后，我们为`NORMAL`优先级操作创建`OperationMessage`消息类的实例，或者为`HIGH`优先级操作创建`HighPriorityOperationMessage`消息类的实例。

### 结论

在本节中，我们学习了如何利用 Symfony Messenger 的功能，根据优先级参数来平衡操作消息在队列间的分布。这使我们能够优先处理某些操作，使其不必等待其他操作处理完毕。

下一章，我们将使用 Symfony 事件系统，在后台操作完成后分派一个事件。

## 第 6 节 触发执行后通知

某些操作可能需要在执行完成后通知用户。例如，在发送一笔付款后，用户可能会收到一封包含付款流程信息的电子邮件或短信。

我们将只对后台操作实现执行后通知，因为用户不知道操作何时结束，通知他们将非常有用。

### 标记需要发送通知的操作

由于并非所有操作都需要发送通知，我们需要一种方法来指定某个操作在处理后需要发送通知。让我们在`OperationMetadata`属性中添加一个新参数来指定这一点。

```php
#[\Attribute(\Attribute::TARGET_CLASS)]
class OperationMetadata
{
    public function __construct(
        public readonly string $name,
        public readonly ?string $version = null,
        public readonly ?string $input = null,
        public readonly ?bool $validateByRole= null,
        public readonly ?Background $background = null,
        public readonly ?bool $requiresNotification = null
    ){ }
}
```

负责决定是否发送通知的代码将需要检查`$requiresNotification`参数。我们将在下一节处理这个问题。

### 创建事件和订阅者

首先，我们需要一个在操作执行完成后触发的事件，以及一个将持续监听该事件的订阅者。

```php
use Symfony\Contracts\EventDispatcher\Event;
class OperationPerformedEvent extends Event
{
public function __construct(
public readonly OperationMetadata $operationMetadata,
public readonly mixed $inputData,
public readonly mixed $outputData,
public readonly ?UserInterface $user
){}
}
```

上述事件携带着以下数据：

- `operationMetadata`: 操作元数据。

- `inputData`: 来自 ApiInput 模型的操作输入数据。

- `outputData`: 来自 ApiOutput 模型的操作结果数据。

- `user`: 操作的用户。它可能为空，因为并非所有操作都需要用户。只有那些实现了`OperationRequiresUserInterface`的操作才需要。

现在，让我们编写订阅者的代码，它将持续监听`OperationPerformedEvent`事件。

```php
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
class OperationSubscriber implements EventSubscriberInterface
{
public static function getSubscribedEvents(): array
{
return [
OperationPerformedEvent::class => 'onOperationPerformed'
];
}
public function onOperationPerformed(OperationPerformedEvent $event): void
{
}
}
```

正如我们所说，订阅者将持续监听`OperationPerformedEvent`事件，并在事件被触发后执行`onOperationPerformed`方法。稍后我们将定义该方法的具体内容。

### 创建通知服务

通知服务应负责向用户发送通知。

```php
class Notifier {
public function notify(string $operation, mixed $inputData, mixed $outputData, ?UserInterface $user) : void
{
}
}
```

`notify`方法是空的。在此方法中，我们将编写发送通知所需的代码。我们可以使用像 Amazon 这样的服务发送电子邮件，或者使用像 Twilio 这样的服务发送短信。我们应该将通知服务注入到订阅者中，并在`onOperationPerformed`方法中使用它。`notify`方法接收操作名称、输入和输出数据以及用户作为参数。

### 基于操作处理通知

需要发送通知的操作可以有不同的处理方式。这就是为什么我们需要将每个操作的通知逻辑封装在不同的服务中。

#### 创建 NotificationHandlerInterface

该接口将定义所有通知处理器必须实现的契约。

```php
use Symfony\Component\Security\Core\User\UserInterface;
interface NotificationHandlerInterface {
public function handleNotification(mixed $inputData, mixed $outputData, ?UserInterface $user);
}
```

处理器必须在`handleNotification`方法中实现发送通知的逻辑。

#### 创建通知处理器

下面，我们展示 SendPayment 操作的通知处理器。它实现了`NotificationHandlerInterface`，并定义了`handleNotification`方法。在此方法中，我们将实现发送逻辑（短信、电子邮件等）。根据接口要求，`handleNotification`方法接收输入和输出数据以及用户。用户可以为空，因为并非所有操作都需要它。

需要注意的是，大多数通知都需要用户，因为通知通常发送给用户。

```php
use App\Api\Notification\NotificationHandlerInterface;
use Symfony\Component\Security\Core\User\UserInterface;
class SendPaymentNotificationHandler implements NotificationHandlerInterface {
public function handleNotification(mixed $inputData, mixed $outputData, ?UserInterface $user) : void
{
}
}
```

但是，我们如何将处理器与操作关联起来呢？让我们借助 [Symfony 服务订阅者](https://symfony.com/doc/6.4/service_container/service_subscribers_locators.html)来实现这个目标。

#### 创建服务订阅者

服务订阅者允许我们访问一组预定义的服务，同时仅在需要时通过服务定位器（一个独立的延迟加载容器）实例化它们。让我们编写服务订阅者的代码。

```php
use App\Api\Notification\Types\SendPaymentNotificationHandler;
use App\Security\Miscelanea\OperationNames;
use Psr\Container\ContainerInterface;
use Symfony\Contracts\Service\ServiceSubscriberInterface;

class NotificationHandlerSubscriber implements ServiceSubscriberInterface
{
    public function __construct(
        private readonly ContainerInterface $locator,
    ) { }

    public static function getSubscribedServices(): array
    {
        return [
            OperationNames::SendPayment->name => SendPaymentNotificationHandler::class
        ];
    }

    public function getHandler(string $operationName): NotificationHandlerInterface
    {
        return $this->locator->get($operationName);
    }
}
```

这个服务订阅者非常简洁。构造函数注入了定位器，该定位器仅持有在 `getSubscribedServices` 方法中定义的服务。`getSubscribedServices` 方法以键值对数组的形式加载可用的服务，其中键是服务键（我们使用操作名称作为键），值是服务类。`getHandler` 方法使用定位器服务来返回与 `$operationName` 参数相关的服务。

#### 使用服务订阅者

最后，我们必须使用订阅者来通过正确的处理器发送通知。让我们修改 `Notifier` 服务。

```php
use Symfony\Component\Security\Core\User\UserInterface;

class Notifier {
    public function __construct(
        private readonly NotificationHandlerSubscriber $notificationHandlerSubscriber
    ){}

    public function notify(string $operation, mixed $inputData, mixed $outputData, ?UserInterface $user) : void
    {
        $handler = $this->notificationHandlerSubscriber->getHandler($operation);
        $handler->handleNotification($inputData, $outputData, $user);
    }
}
```

现在，`Notifier` 服务根据操作名称使用相应的处理器。它注入了新创建的 `NotificationHandlerSubscriber`，并使用它来获取处理器。然后，它使用该处理器发送通知。

### 使用通知服务

准备好 `Notifier` 服务后，我们可以在 `OperationSubscriber` 中使用它。

```php
use App\EventSubscriber\Event\OperationPerformedEvent;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class OperationSubscriber implements EventSubscriberInterface
{
    public function __construct(
        private readonly Notifier $notifier
    ){}

    public static function getSubscribedEvents(): array
    {
        return [
            OperationPerformedEvent::class => 'onOperationPerformed'
        ];
    }

    public function onOperationPerformed(OperationPerformedEvent $event): void
    {
        if($event->operationMetadata->requiresNotification){
            $this->notifier->notify($event->operationMetadata->name, $event->inputData, $event->outputData, $event->user);
        }
    }
}
```

`onOperationPerformed` 方法仅当操作需要发送通知时才会执行此操作。它通过检查元数据的 `requiresNotification` 参数来判断。

### 触发事件

到目前为止，我们已经创建了订阅者，并构建了一个用于放置发送通知代码的服务。现在我们需要在后端操作完成后触发事件。我们将把这部分代码放在 `OperationMessageHandler` 中。

```php
use App\Api\Collection\OperationCollection;
use App\Api\Operation\OperationRequiresUserInterface;
use App\EventSubscriber\Event\OperationPerformedEvent;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
class OperationMessageHandler
{
    public function __construct(
        private readonly OperationCollection $operationCollection,
        private readonly EventDispatcherInterface $eventDispatcher,
        private readonly EntityManagerInterface $em
    ){ }

    public function __invoke(OperationMessage $message): void
    {
        $apiOperation = $this->operationCollection->getOperation($message->operation);
        $user = null;
        if($message->user && $apiOperation->handler instanceof  OperationRequiresUserInterface) {
            $user = $this->em->getRepository(User::class)->findOneByEmail($message->user);
            $apiOperation->handler->setUser($user);
        }
        $apiOutput = $apiOperation->handler->perform($message->operationInput);
        $this->eventDispatcher->dispatch(
            new OperationPerformedEvent($apiOperation->metadata, $message->operationInput, $apiOutput->data, $user),
        );
    }
}
```

我们注入了 Symfony 事件分发器服务，并在获取操作结果后，使用 `dispatch` 方法来触发事件。事件触发后，`OperationSubscriber` 将检测到触发的事件并做出响应。此外，我们将用户初始化为 `null` 值，以避免在将其传递给事件构造函数时出现“未定义变量”错误。

### 结论

本章节结束之际，我们已掌握了如何在后台操作完成后分发事件，从而能够通知用户。在下一章中，我们将实现必要的机制，确保端点只能执行与其上下文相关的操作。