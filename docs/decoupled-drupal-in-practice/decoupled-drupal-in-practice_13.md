# 9. 在 Drupal 8 中认证请求

在解耦式 Drupal 架构中，最受关注的领域之一便是安全性。本质上，解耦式 Drupal 会带来显著的安全隐患，这些隐患既影响 Drupal 中存储的数据，也影响通过消费者访问 Drupal 内容的用户安全。实际上，第 6 章中提到解耦式 Drupal 的缺点之一，就是开发团队需要承担更重的责任来构建稳健的身份认证机制。

针对解耦式 Drupal，Drupal 8 提供了三种相关的认证方法。基本认证和 OAuth2 Bearer Token 认证是解耦式 Drupal 架构中最常用的，而基于 Cookie 的认证则对渐进式解耦 Drupal 实现至关重要，因为在这种实现中，消费者会使用 Drupal 渲染页面上的用户会话 cookie。此外还存在其他方法，最引人注目的是 JSON Web Tokens (JWT)，它由 Drupal 贡献模块实现。

## 基本认证

基本认证模块是目前最易用的，但安全性也相对较低。该模块集成于 Drupal 8 核心，能够处理传入请求，提取提供的用户名和密码，并在 Drupal 中进行认证，以确认相应用户具有检索或操作所请求内容的正确权限。[²⁸]

尽管如此，在解耦式 Drupal 中使用基本认证时仍需极度谨慎，因为用户名和密码仅受到有限的保护。在基本认证中，凭证以 base64 编码（即未加密或哈希处理）在网络上传输，这种编码很容易被转换为可被利用的明文。此外，每个请求通常都包含凭证，这意味着这些敏感信息会被反复传输，从而形成了更大的攻击窗口。[²⁹] 鉴于这些隐患，基本认证应仅在 Drupal 后端启用 HTTPS 时使用。

### 警告

由于基本认证存在固有问题，最佳实践是在需要检索敏感数据的生产环境中使用 Simple OAuth 或 JSON Web Tokens（本章后续会介绍）。然而，针对非敏感数据的认证请求，基本认证能提供更便捷的开发体验，因此在开发或技术演示中经常使用。

### HTTP 基本认证

基本认证模块实现了 HTTP Basic 协议，该协议规定了如何进行基本访问认证。HTTP Basic 协议允许用户代理发出请求，其中包含标准格式的用户名和密码。HTTP Basic 协议通常备受青睐，因为它无需使用 Cookie（见下一节）或会话标识符即可实施访问控制，并且通过 HTTP 标头实现，免去了握手过程。[³⁰]

HTTP Basic 协议规定了如何构建 `Authorization` 字段以向 Drupal 传输认证凭证。根据 RFC 7617（2015），我们可以按以下步骤构建 `Authorization` 字段：[³¹]

1.  将用户名和密码用冒号连接起来，这意味着用户名中不能包含冒号。
2.  将连接后的字符串编码为八位字节序列。此编码步骤的字符集默认可以未指定，或由服务器发出的 `charset` 参数指定。
3.  然后使用 base64 编码的变体对该编码字符串进行编码。
4.  最后，在 base64 编码的字符串前加上相应的认证方法（例如 `Basic`）和一个空格。

### Authorization 标头

在 Drupal 中，如果请求执行了某项匿名用户无权执行的操作，则必须包含一个 `Authorization` 标头，其中包含拥有足够权限角色的用户的凭证，无论是更新还是删除实体。在发送到 Drupal 的任何敏感请求中，`Authorization` 标头必须由消费者设置，并且消费者还需要自行处理上述准备步骤。

例如，考虑用户名和密码组合 `admin` 和 `admin`。请看下面的 JavaScript 函数，它通过字符串拼接和 JavaScript 原生 `btoa()` 函数返回格式正确的 `Authorization` 字段：

```
function encodeBasicAuth(user, pass) {
  var creds = user + ':' + pass;
  var base64 = btoa(creds);
  return 'Basic ' + base64;
}
```

然后，在 `XMLHttpRequest` (XHR) 中，你可以调用该函数。正如我们将在后续章节中看到的，许多 JavaScript 框架（例如 `Waterwheel.js`，见第 16 章）通过提供自己的 XHR API 来加速此过程。请注意，以下示例中引用的 Drupal 后端与第 7 章中配置的相同。

```
var req = new XMLHttpRequest();
req.open('GET', 'https://core-rest.dd:8083/node/1');
req.setRequestHeader('Authorization', encodeBasicAuth('admin', 'admin'));
req.send('_format=hal_json');
```

该请求也可以写成下面这样，它反映了一个需要对内容实体进行基本认证才能检索的 Drupal 后端。如你所见，我们经过 base64 编码的 `Authorization` 字段为 `YWRtaW46YWRtaW4=`。

```
GET /node/1?_format=hal_json HTTP/1.1
Content-Type: application/json
X-CSRF-Token: SDEEgyW_n2vI3GygOI2Y-W7VRrfIiN8gk3PdO1O3vHo
Authorization: YWRtaW46YWRtaW4=
Host: core-rest.dd:8083
```



### 基于 Cookie 的身份验证

在 Drupal 中，*基于 Cookie 的身份验证*是在发出请求时验证用户凭据的一种附加方法。基本身份验证与基于 Cookie 的身份验证之间的主要区别在于后者用于 Drupal 的正常操作。基本身份验证侧重于第三方应用程序，而在基于 Cookie 的身份验证中，Drupal 在浏览器上使用 Cookie 来保持用户的会话。

因此，基于 Cookie 的身份验证在渐进式解耦的情况下特别有用，因为任何需要身份验证的 Drupal 页面都会引用浏览器中存储的会话 Cookie。由于渐进式解耦涉及将 JavaScript 框架插入到 Drupal 的前端，该框架也可以**完全免费**地访问经过身份验证的 Cookie。这是因为框架和周围的 Drupal 前端都可以访问存储 Cookie 的 `document.cookie` 对象。

对于完全解耦的实现，基于 Cookie 的身份验证与基本身份验证大致类似，但有一个关键区别，即活动会话 Cookie 的值很容易被窃取，随后被用来利用 Drupal 后端。因此，尽管为了完整性，我将其包含在内以说明其功能，但您应当极其谨慎地使用这种方法。

### 在完全解耦的消费者中检索 Cookie

可以在完全解耦的 Drupal 架构中用基于 Cookie 的身份验证替代基本身份验证，特别是当您的身份验证需求相对简单且数据敏感性较低时。在基本身份验证中，用户名和密码在客户端上以明文形式可用，而在完全解耦的基于 Cookie 的身份验证实现中，会话 Cookie 存储在客户端上，并在每次请求时传输。

### 警告

由于基于 Cookie 的身份验证存在固有漏洞，因此在需要检索敏感数据的生产环境中，最佳实践是使用 Simple OAuth 或 JSON Web Tokens（本章后面会介绍）。然而，对于针对非敏感数据的已验证请求，基于 Cookie 的身份验证可以提供更便捷的开发体验，因此在开发或技术演示中经常使用。

要检索用于基于 Cookie 的身份验证的会话 Cookie，我们需要一个用户会话。要获取用户会话，您可以向提供您内容的 Drupal 后端的登录表单（`/user/login?_format=json`）发出一个 `POST` 请求，并在请求中提交以下请求体：

```
{
"name": "admin",
"pass": "admin"
}
```

如图 9-1 和 9-2 所示，对该请求的响应将包含一个 `200 OK` 响应代码，并包含一个会话 Cookie，以及一个包含已启动会话的用户信息及可用于使用不安全 HTTP 方法发出请求（详见第 10 章关于 `X-CSRF-Token` 的内容）或注销用户的令牌的响应对象。

```
{
"current_user": {
"uid": "1",
"roles": [
"authenticated",
"administrator"
],
"name": "admin"
},
"csrf_token": "SDEEgyW_n2vI3GygOI2Y-W7VRrfIiN8gk3PdO1O3vHo",
"logout_token": "jW66ozYwAe_D_rj2sY61WfuzPW9ft91ei6vngUhoYms"
}
```

### 注意

在这些示例中，我们使用的是 HTTP 客户端 Postman。有关 Postman 及其用法的更多信息，请参阅第 10 章。

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig2_HTML.jpg](img/468476_1_En_9_Fig2_HTML.jpg)

**图 9-2** 会话 Cookie 可用于通过已登录用户的会话对来自消费者应用程序的请求进行身份验证

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig1_HTML.jpg](img/468476_1_En_9_Fig1_HTML.jpg)

**图 9-1** 我们可以向 Drupal 的登录表单发出一个 `POST` 请求，以检索进行基于 Cookie 的身份验证所需的 Cookie。该响应还包含有关用户角色和用于其他目的的令牌的有用信息。

### 使用 Cookie 进行身份验证

在完全解耦的 Drupal 实现中，需要执行上一节中的步骤才能访问会话 Cookie。然而，在渐进式解耦的实现中，会话 Cookie 在 `document.cookie` 中可用，因为 JavaScript 框架和 Drupal 渲染的前端共享同一个 DOM。

您可以在任何后续需要身份验证才能继续的请求的请求头中包含 Cookie 名称（例如：`SESS4aabd467346dd626e54a80f80ddac4cb`）和值（例如：`_bY169JQ3op6EgvPF2pWF1rW0NfFvKdiklNZR7rMJl0`），两者用 `=` 分隔。例如，考虑以下用于创建节点的 `POST` 请求的请求头。有关针对核心 REST 的 `POST` 请求的更多示例，请参阅第 10 章。

```
POST /entity/node?_format=json HTTP/1.1
Content-Type: application/json
X-CSRF-Token: SDEEgyW_n2vI3GygOI2Y-W7VRrfIiN8gk3PdO1O3vHo
Cookie: SESS4aabd467346dd626e54a80f80ddac4cb=_bY169JQ3op6EgvPF2pWF1rW0NfFvKdiklNZR7rMJl0
Host: core-rest.dd:8083
```

要注销用户，只需向 `/user/logout?_format=json&token=jW66ozYwAe_D_rj2sY61WfuzPW9ft91ei6vngUhoYms` 发出一个 `POST` 请求（使用之前检索到的 `logout_token` 作为第二个查询字符串参数），您将收到一个状态码 `204 No Content` 的响应，表示用户会话已终止。

## OAuth 2.0 Bearer Token 身份验证

OAuth 是当今最广泛使用的身份验证方法之一，它是一种用于访问授权的开放标准，允许无需密码即可授予对信息的访问权限。OAuth 有多个版本；第一个版本（OAuth）由 OAuth 贡献模块支持，第二个版本（OAuth 2.0）由 Simple OAuth 贡献模块支持。

在 OAuth 中，消费者被代表资源所有者授予访问服务器上资源的权限，而资源所有者授权访问时无需共享消费者凭据。OAuth 通过授权服务器向消费者颁发 HTTP 访问令牌。此后，消费者在其发出的请求中使用该访问令牌来检索或操作受保护的资源。

由于 OAuth 的第一个版本安全性较低，且与 OAuth 2.0 不向后兼容，因此本节我们只关注第二个版本。一些开发者可能选择使用 OAuth 1 以避免 HTTPS 服务器要求，但这已不再可取。

### 注意

Simple OAuth 模块（OAuth 2.0）可在 [Drupal.org](http://drupal.org) 上获取，网址为 [`www.drupal.org/project/simple_oauth`](https://www.drupal.org/project/simple_oauth)。OAuth 模块（OAuth 1）可在 [Drupal.org](http://drupal.org) 上获取，网址为 [`www.drupal.org/project/oauth`](https://www.drupal.org/project/oauth)。

### 警告

OAuth 1 现在被认为是比 OAuth 2.0 安全性更低，在可能使用 OAuth 2.0 的生产环境中不应使用。



### OAuth 2.0 授权许可类型

在 OAuth 2.0 中，*授权许可类型（grant）*是一种获取访问令牌（access token）的方式，消费者（consumer）可利用该令牌访问受保护的资源。从消费者的角度来看，通过 API 与后端通信有多种方式，但根据消费者与服务器之间的关系，某种特定的授权许可类型可能比其他类型更优。例如，*授权码许可类型*常用于构建许多消费 GitHub 数据的自定义第三方应用程序。^(³⁶)

PHP League 定义了四种 OAuth 2.0 授权许可类型，它们与 OAuth 2.0 授权框架 RFC（RFC 6749）中提到的五种授权许可类型精确对应。在此过程中，PHP League 还针对特定授权许可类型的选用，提出了若干理由。在下文中，*第一方*（first-party）指代受充分信任、可处理终端用户凭据的消费者，而*第三方*（third-party）消费者则不受信任。^(³⁷)

- *授权码许可类型*：如果访问资源需要用户（访问令牌所有者）的许可，并且消费者是 Web 应用程序或第三方原生应用程序，则可以使用授权码许可类型。
- *隐式许可类型*：如果访问资源需要用户的许可，并且消费者是基于浏览器的第三方应用程序，则可以使用隐式许可类型。
- *客户端凭证许可类型*：如果访问资源不需要用户的许可，则可以使用客户端凭证许可类型。
- *密码许可类型*：如果访问资源需要用户的许可，并且消费者是第一方应用程序，则可以使用密码许可类型。

`Simple OAuth` 模块强调使用密码许可类型，因为它是一种便捷的一体化方案，适用于受信任的第一方应用程序——这些应用可能需要访问 Drupal 中任何可能的操作，包括创建和删除用户。尽管如此，密码许可类型可能会分配给消费者应用程序过高的权限。在这种情况下，可以考虑使用其他授权许可类型。

### 警告

关于密码许可类型对于单页应用程序（如需要在可读代码中存储客户端密钥的应用程序）甚至原生应用程序是否足够安全，目前存在相当多的争议。额外要求用户凭据（参见后续章节）可能足以满足您的安全需求，但并非所有实现都将其视为最佳实践。有关更多背景信息，请参阅此 GitHub issue：[`https://github.com/thephpleague/oauth2-server/issues/889`](https://github.com/thephpleague/oauth2-server/issues/889)。

### 安装 Simple OAuth 并生成密钥

要安装 `Simple OAuth`，您可以执行以下 Composer 命令。

```
$ composer config repositories.drupal composer https://packages.drupal.org/8 && composer require drupal/simple_oauth:³
```

这将同时安装 `Simple OAuth` 模块和 `OAuth2 Server` 包（由 PHP League 提供），后者是一个严格的依赖项。您也可以使用 Drush 或 Drupal 用户界面安装 `Simple OAuth`，但安装 `OAuth2 Server` 是必需的步骤，无论您是通过 Composer 管理还是直接下载。

安装 `Simple OAuth` 后，您需要生成一对密钥来加密 `Simple OAuth` 生成的令牌。出于安全考虑，请将这些密钥存储在 Drupal 文档根目录之外，同时确保保存密钥的相应路径。

```
$ openssl genrsa -out private.key 2048
$ openssl rsa -in private.key -pubout > public.key
```

### 注意

如果您在尝试使用生成的密钥时遇到错误，可能是因为这些文件的权限设置不正确。请使用 `chmod` 命令将密钥目录的八进制代码设置为 `0600`。

### OAuth 2.0 作用域与 Drupal 角色

在 OAuth 2.0 中，*作用域（scopes）*是权限，有助于定义 OAuth 2.0 令牌应当能够访问的操作。在 Drupal 中，它们与*用户角色*是同义的。通常，最佳实践是将每个消费者与单个用户角色关联，例如为我们的“Swift 应用”创建一个显式角色，与现有的管理员和已验证用户角色并存。

在图 9-3 中，我们创建了一个新的 Swift 应用角色，并在图 9-4 中为其赋予了与管理角色相同的权限。

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig4_HTML.jpg](img/468476_1_En_9_Fig4_HTML.jpg)

图 9-4

在此示例中，我们将消费者的所有权限设置得与管理员的权限完全相同，因为该消费者是一个受信任的第一方实体。

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig3_HTML.jpg](img/468476_1_En_9_Fig3_HTML.jpg)

图 9-3

创建一个具有自身权限的新角色，以反映消费者可以访问的作用域。最佳实践是使消费者与其角色一一对应。

最后一步是让 Drupal 识别所有将使用 OAuth2 令牌来检索或操作内容的消费者。为此，我们将创建新的消费者，并赋予其我们刚刚创建的作用域（Drupal 角色）。导航至配置 ➤ Simple OAuth ➤ 消费者页面（`/admin/config/services/consumer`），然后单击“添加消费者”（`/admin/config/services/consumer/add`）。

表单将要求输入用于标识消费者的标签（例如，Swift 应用，一个用 Swift 编写的 iOS 应用程序）和新的密钥（消费者将使用的新密码，在 Drupal 中经过哈希处理；例如，`l0r3m1psum`）。还有一些可选字段，将在后续章节中介绍。保存表单后，Drupal 将生成一个消费者标识符，即一个 UUID，用于识别该消费者（例如，`24ac1dc6-9cd3-11e8-98d0-529269fb1459`）。这就是您将在 Simple OAuth 模块的消费者页面上看到的内容（图 9-5）。

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig5_HTML.jpg](img/468476_1_En_9_Fig5_HTML.jpg)

图 9-5

创建消费者后，它将出现在我们的消费者列表中，并附带其 UUID 和分配的作用域。

导航至 Simple OAuth 配置页面（`/admin/config/people/simple_oauth`），并插入您之前保存的公钥和私钥路径。此外，您还可以为令牌设置更长的过期时间（例如，`870000` 秒而非 `300` 秒）用于测试目的，但不应在生产环境中这样做。^(³⁸)



### 创建和验证访问令牌

现在我们可以发出第一个请求，获取 Drupal 授予我们用于对后端执行操作的访问令牌。我们需要发出请求的资源是由 Simple OAuth 提供的访问令牌资源，位于 `/oauth/token`。请求体需要包含 Drupal 识别客户端所需的所有元素。

由于 OAuth 2.0 规范要求 OAuth 令牌资源仅接受 `POST` 请求，且格式为 `form-data` 或 `x-www-form-urlencoded`，因此我们不能使用传统的 JSON 格式请求体来获取访问令牌。（事实上，JSON 格式的响应会被拒绝。）幸运的是，Postman 提供了一个方便的界面，可以用 `form-data` 或 `x-www-form-urlencoded` 格式插入请求体。

每个 `POST` 请求都需要包含表 9-1 中所示的 OAuth 2.0 所要求的编码信息。

**表 9-1** 获取 OAuth 2.0 令牌所需请求体参数

| 键 | 示例值 | 描述 |
| --- | --- | --- |
| `grant_type` | `password` | 授权类型，通常为 `password` |
| `client_id` | `24ac1dc6-9cd3-11e8-98d0-529269fb1459` | Drupal 在创建客户端时生成的消费者 UUID |
| `client_secret` | `l0r3m1psum` | 添加客户端时选择的客户端密钥 |
| `username` | `admin` | 希望与客户端关联（并授予相应权限）的账户用户名 |
| `password` | `admin` | 用户账户的密码 |

发出此 `POST` 请求将得到一个包含 JSON 对象的响应，其中有两个重要的键：`access_token` 和 `refresh_token`。以下是 OAuth 令牌资源的示例响应，为了简洁起见，令牌内容已截断。

```
{
  "token_type": "Bearer",
  "expires_in": 870000,
  "access_token": "eyJ0eXAi0iJKV1Qi[...]",
  "refresh_token": "uAXzh+B/7kCxsXkl[...]"
}
```

为了验证我们的访问令牌确实能与 Drupal 的 OAuth 2.0 服务器配合使用，我们可以向 `/oauth/debug` 资源发出一个 `GET` 请求，并附加 `?_format=json`。如果不带 `Authorization` 头部发出 `GET` 请求，将返回一个 JSON 编码的响应，其中包含典型匿名用户的 Drupal 角色和权限。

我们可以使用调试端点 `/oauth/debug` 来验证访问令牌是否正确。添加一个包含 `Bearer`（注意前缀后的空格）的 `Authorization` 头部，后接从 `/oauth/token` 响应中复制的访问令牌，然后发出 `GET` 请求。

```
GET /oauth/debug?_format=json HTTP/1.1
Authorization: Bearer eyJ0eXAi0iJKV1Qi[...]
```

响应将包含一个带有提交的访问令牌的 JSON 对象。如果身份验证成功，还将包含在获取访问令牌时提供凭据的用户的标识符。此外，Drupal 的 OAuth 2.0 服务器会在响应中提供该用户的角色和权限列表。^(³⁹)

### 发出 OAuth 2.0 认证的请求

将密钥保存到 Simple OAuth 配置页面后，你可以指示 Drupal 允许某些方法和资源通过 OAuth2 身份验证暴露。可以使用配置导入方法（见第 7 章）或 REST UI 贡献模块（见第 8 章）来启用 `oauth2` 身份验证方法。

我们可以按如下方式启用核心 REST 模块和 REST UI。

```
$ composer require drupal/restui
$ drush en -y rest restui
```

然后，导航到 Drupal 中的 Configuration » REST (`/admin/config/services/rest`) 以访问 REST UI，那里列出了核心 REST 可用的资源列表。目前，我们可以只关注 Content 类别下的设置。在 Content 资源的设置页面 (`/admin/config/services/rest/resource/entity%3Anode/edit`) 上，假设你已正确配置 Simple OAuth 模块，那么 `oauth2` 选项将在 Authenticated providers 列表中可用。启用 `oauth2` 选项将允许来自消费者的 OAuth 2.0 认证请求访问正确的资源。

要检索令牌，向 `/oauth/token` 发出一个 `POST` 请求，请求体包含以下参数，格式为 `form-data`。

```
grant_type: password
client_id: 24ac1dc6-9cd3-11e8-98d0-529269fb1459
client_secret: l0r3m1psum
username: admin
password: admin
```

最后，在消费者请求中添加一个包含 `Bearer` 前缀的 `Authorization` 头部，如下所示。一旦 Drupal 收到包含 `Authorization` 头部中 OAuth 2.0 令牌的消费者请求，Drupal 将在验证令牌后处理该请求。

```
Authorization: Bearer eyJ0eXAi0iJKV1Qi[...]
```

> **注：** 如需更多关于 Drupal 的 OAuth 2.0 实现的资源，Mateu Aguiló Bosch 在 YouTube 上制作了一个很好的视频系列，详细介绍了 Simple OAuth 模块的一些功能，特别是其对其他授权类型的处理，网址为 [`youtu.be/rTcC0maPLSA`](https://youtu.be/rTcC0maPLSA)。

#### 处理过期令牌

在开发期间，设置较长的 OAuth 2.0 令牌过期时间是完全合适的。然而，一旦你的消费者和服务器投入生产环境，不建议将 OAuth 2.0 令牌的过期时间设置得过长。毕竟，如果你的令牌在 120 秒内过期，所有需要该令牌的操作都必须在此时段内完成；否则，你的令牌将过期。

如果你的令牌已过期，可以按照以下步骤生成新令牌，以便继续向 Drupal 发出请求。当 Simple OAuth 生成一个访问令牌（用于所有 OAuth 2.0 认证请求，见上一节）时，它还会生成一个名为 *刷新令牌* 的身份验证令牌。刷新令牌的有效期更长，并与另一个可以替代过期访问令牌的访问令牌相关联。要获取新的访问令牌，我们需要使用刷新令牌授权，而不是密码授权。

要使用刷新令牌授权，你需要启用 Simple OAuth Extras 模块，该模块随 Simple OAuth 模块一起下载。

```
$ drush en -y simple_oauth_extras
```

要使用初始响应中提供的刷新令牌（包含过期访问令牌）获取新的访问令牌，向 `/oauth/token` 发出一个 `POST` 请求，请求体包含以下参数，格式为 `form-data`。

```
grant_type: refresh_token
refresh_token: uAXzh+B/7kCxsXkl[...]
client_id: 24ac1dc6-9cd3-11e8-98d0-529269fb1459
client_secret: l0r3m1psum
```

你还可以要求新的访问令牌仅限于特定范围（Drupal 角色）。请注意，`scope` 是可选的，但可以用于将新生成的令牌关联到与之前不同的范围，我们可以使用所有访问应该在哪个 Drupal 角色下进行的机器名称来表示。

```
scope: swift_app
```

作为响应，Drupal 的 OAuth 2.0 实现将返回一个 JSON 对象，其中包含键 `token_type`（值为 `Bearer`）、`expires_in`（新访问令牌的 TTL）、`access_token`（一个使用 Drupal 识别的私钥签名的新访问令牌）和 `refresh_token`（一个用于刷新访问令牌的新刷新令牌）。^(⁴⁰)

如果你的刷新令牌也已过期，那么别无选择，只能从头生成一个新令牌，这会产生一个问题，因为它需要通过网络传输用户凭据。你可以根据原始访问令牌响应中提供的持续时间，确保你的消费者在刷新令牌过期之前发出获取新访问令牌的请求，从而实现刷新访问令牌流程的自动化。



### 注意

您可以直接在 Drupal 站点上调试现有令牌，方法是导航至“配置” ➤ “Simple OAuth” ➤ “令牌”（`/admin/config/people/simple_oauth/oauth2_token`），您将在此处找到 Drupal 已生成的现有令牌列表。^(⁴¹)

PHP League 对 OAuth 2.0 的实现使用 JSON Web 令牌（JWT）来表达所有身份验证令牌。有关 JWT 的更多信息，请继续阅读下一节。

## JSON Web 令牌

JSON Web 令牌（JWT）由 RFC 7519 定义，是一个快速成熟的开放标准，用于通过 JSON 对象进行身份验证，该标准在 `JSON Web Token Authentication` 模块（或简称 `JWT` 模块）中实现。该 Drupal 实现由 Jonathan Green（`jonathan.green`）和 Gabe Sullice（`gabesullice`）编写，在 Drupal 中提供了一个身份验证提供者，允许通过数字签名验证信息。

在 JWT 中，令牌通过以下方式*签名*：要么使用 HMAC 算法通过密钥，要么通过 RSA 或 ECDSA 使用典型的公钥和私钥对。JWT 标准对于通过在每次请求时传输令牌来授权用户会话，以及通过验证签名令牌来促进各方之间的信息交换特别有用。^(⁴²)

### 注意

JSON Web 令牌（JWT）规范位于 [`https://jwt.io`](https://jwt.io)。

### JSON Web 令牌标准

单个 JSON Web 令牌（JWT，发音为 *jots*）由三个以点分隔的组件组成——头部（header）、载荷（payload）和签名（signature）——并遵循以下格式：

```
header.payload.signature
```

JWT 的第一部分是*头部*。JWT 头部通常分为两部分：令牌类型（JWT）和用于加密载荷的选定哈希算法（例如 HMAC SHA256 或 RSA），如下例所示，以 JSON 对象表达。

```
{
"alg": "HS256",
"typ": "JWT"
}
```

然后对该对象进行 base64url 编码，以构成 JWT 的第一部分。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

第二部分是*载荷*，它表达了关于消费者身份的*声明*（或信息陈述）、我们希望传输的信息，以及有关该令牌的其他信息。^(⁴³) 有三种不同类型的声明：

-   *已注册声明：* 已注册声明是载荷中的键/值对，这些声明是推荐的但不是强制性的，并且遵循某些预定义的、可互操作的声明，包括 `iss`（令牌的签发者）、`exp`（过期时间）、`sub`（令牌的主题）、`aud`（令牌的受众）、`nbf`（JWT 必须不被接受之前的时间）、`iat`（JWT 的签发时间）和 `jti`（唯一的令牌标识符）。

-   *公共声明：* 公共声明是 JWT 可以随意定义的声明。公共声明的权威列表是 IANA JSON Web 令牌注册表，该注册表列出了所有已定义的声明。这是为了避免在两个 JWT 定义了相同的公共声明时发生冲突。另一种方法是通过一个能抵抗冲突的 URI（例如，包含 UUID 的 URI）来定义公共声明。

-   *私有声明：* 私有声明是各方同意定义以私下共享信息的自定义声明。这些声明既不是已注册的也不是公共的，因为它们既不属于 IANA 注册表，也不属于已注册声明的备选列表。

以下是一个表示载荷的 JSON 对象示例。

```
{
"iss": "jwt-drupal-backend.net",
"exp": 1534864265,
"name": "Preston So",
"admin": true
}
```

与 JWT 头部一样，JWT 载荷也是经过 base64url 编码的。对于已签名的令牌，头部和载荷都是全局可读的，这种情况只有在头部和载荷中包含的信息在 base64url 编码之前被加密时才能解决。

```
ewogICJpc3MiOiAiand0LWRydXBhbC1iYWNrZW5kLm5ldCIsCiAgImV4cCI6IDE1MzQ4NjQyNjUsCiAgIm5hbWUiOiAiUHJlc3RvbiBTbyIsCiAgImFkbWluIjogdHJ1ZQp9
```

第三部分也是最后一部分是*签名*，它由编码后的头部、编码后的载荷和密钥（服务器可用的签名，用于验证现有令牌并签署新令牌）的哈希值组成。您必须使用 JWT 头部中指定的算法来生成签名。签名保证了载荷中的任何数据都未被修改。

例如，如果您使用 HMAC SHA256 作为所选算法，您可以创建处理 base64url 编码和 HMAC SHA256 哈希的方法，并如下例所示使用它们，其中 `secret` 是一个密钥，由消费者提供并由服务器验证，用于创建数字签名的令牌。

```
const signature = HMACSHA256(
base64url(header) + '.' + base64url(payload),
secret
);
```

可以在这里看到一个示例签名：

```
jIyIIA6wMwGCLE2fyIU1f_Y9e3Nn4rPC3Ta1MzoAKLA
```

一旦有了哈希签名，您现在可以将所有三个字符串用句点连接起来，形成完整的 JWT。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.ewogICJpc3MiOiAiand0LWRydXBhbC1iYWNrZW5kLm5ldCIsCiAgImV4cCI6IDE1MzQ4NjQyNjUsCiAgIm5hbWUiOiAiUHJlc3RvbiBTbyIsCiAgImFkbWluIjogdHJ1ZQp9.jIyIIA6wMwGCLE2fyIU1f_Y9e3Nn4rPC3Ta1MzoAKLA
```

### 注意

IANA JSON Web 令牌注册表位于 [`https://www.iana.org/assignments/jwt/jwt.xhtml`](https://www.iana.org/assignments/jwt/jwt.xhtml)。`jwt.io` 调试器位于 [`http://jwt.io`](http://jwt.io)。

### JSON Web 令牌的工作原理

与 HTTP 基本身份验证和 OAuth 2.0 等替代方案相比，JWT 标准具有若干优势。首先，JWT 尺寸紧凑，开销有限，尤其是与安全断言标记语言（SAML）等替代方法相比——这意味着 JWT 可以在 URL、`POST` 参数或 HTTP `header` 中发送。其次，JWT 是自包含的，关于消费者的所有必需信息都存在于 JWT 载荷中。最后，JWT 在 CORS 中功能完备，因为基于令牌的身份验证系统在令牌位于 HTTP 头部时允许调用任何服务器。

JWT 被认为比基于会话的身份验证机制（如 cookie 和 OAuth 2.0）更安全，因为它们不需要在服务器上维护会话数据，从而无需在提供相同应用程序的服务器之间共享此类会话数据。此外，JWT 在载荷内表达了自身的到期日期，因此在会话到期时无需进行垃圾回收。

然而，对于解耦的 Drupal 实践者来说，可能更相关的是 JWT 是*无状态的*，因为消费者和服务器之间的 RESTful 通信要求发出的每个请求都附带一个有效的 JWT。由于 JWT 可以与每个请求一起传输，并且已经表达了关于用户和消费者的所有信息，因此无需从数据库中提取这些详细信息。

图 9-6 展示了一个典型的身份验证工作流程，其中身份验证提供者（在本例中为 Drupal）接收对 JWT 的请求，使用提供的密钥创建数字签名的 JWT，并将该 JWT 签发给消费者以存储在客户端。从那时起，消费者向服务器发出请求，在服务器上对 JWT 进行验证和确认。然后服务器将响应传输给消费者。

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig6_HTML.jpg](img/468476_1_En_9_Fig6_HTML.jpg)

图 9-6

Drupal 中的 JWT 身份验证工作流程，Drupal 同时充当身份验证和 Web 服务提供者。当消费者使用有效凭据请求新令牌时，Drupal 的 JWT 身份验证提供者会创建并向消费者签发一个新的 JWT。然后，消费者在其发出的每个 API 请求中都使用此 JWT。



### 安装 JSON Web Token

Drupal 对 JSON Web Token 的实现需要 `Key` 模块，并可通过常规模块安装流程进行安装，但有一个重要注意事项。由于 JWT 自带 `composer.json` 文件，并依赖 `firebase/php-jwt` PHP 库，你可以使用 Composer Manager 模块来安装这个第三方库，或者采用基于 Composer 的自定义方式，例如以下命令。

```
$ composer config repositories.drupal composer https://packages.drupal.org/8
$ composer require drupal/jwt
$ drush en -y jwt
```

我们还需要启用 JWT 模块的两个内部子模块，用于处理 Drupal 对 JWT 的签发和消费。

```
$ drush en -y jwt_auth_consumer jwt_auth_issuer
```

如前几节所述，Drupal 除了响应消费者请求外，其主要职责是提供能够签名或验证 JWT 的密钥，这一功能由 `Key` 模块处理。完成 JWT 模块的安装后，我们需要通过导航到**配置** ➤ **系统** ➤ **密钥**（`/admin/config/system/keys`）来创建一个新密钥，在此处可以添加新的或现有的密钥。

JWT 在使用的哈希算法方面为我们提供了选择，而 Drupal 对 JWT 的实现包含了其中两种选项：JWT HMAC 密钥和 JWT RSA 密钥。我们将在下一节逐一介绍。

### 注意

JSON Web Token 认证模块是 Drupal 对 JWT 的实现，可在 [Drupal.org](http://drupal.org) 的 [`https://www.drupal.org/project/jwt`](https://www.drupal.org/project/jwt) 获取。`Key` 模块可在 [Drupal.org](http://drupal.org) 的 [`https://www.drupal.org/project/key`](https://www.drupal.org/project/key) 获取。`firebase/php-jwt` 库可在 GitHub 的 [`https://github.com/firebase/php-jwt`](https://github.com/firebase/php-jwt) 获取。

### 警告

Drupal 中的 JWT 模块仍为 alpha 版本，未经极度谨慎和小心，不得在生产环境中直接使用。

### 创建 JWT HMAC 和 JWT RSA 密钥

JWT 模块文档推荐使用基于文件的密钥，无论你使用 HMAC 还是 RSA。请记住，在 Drupal 中，密钥与 JWT 密钥是同义词。

为了在 Drupal 中使用 HMAC 签发和验证 JWT，你可以生成一个 256 位、base64 编码的基于文件的密钥。

```
$ head -c 64 /dev/urandom | base64 -w 0 > /path/to/private/keys/jwt.key.txt
```

以下是一个 HMAC 密钥示例：

```
md64thgk0icWTrZ5B4Yb45nvYwPcaWnrn/82lJW0DW4piGQcU2TC/BL/lOZLsiwnSn0dinr1rZOmww0nZ9Aurg==
```

### 注意

如果你收到错误信息 `base64: invalid option`，你可能需要使用 `-b (--break)` 选项而不是 `-w (--wrap)` 选项，这取决于你使用的 `base64` 命令行界面版本。BSD 允许使用 `-w`，而 GNU 允许使用 `-b`。在两个版本中，`-b` 和 `-w` 标志都用于处理编码时的换行。

为了在 Drupal 中使用 RSA 签发和验证 JWT，你可以生成一个 2048 位的基于文件的密钥。

```
$ openssl genrsa -out private.key 2048 > /path/to/private/keys/jwt.key.txt
```

创建好你计划提供给消费者的密钥后，并指定了它是应作为配置存储还是通过文件引用后，导航到**配置** ➤ **系统** ➤ **密钥**（`/admin/config/system/keys`），在此处你可以指定密钥是应作为配置存储（这种情况下你需要在表单中直接提供值）还是作为文件引用（已位于服务器上）。如图 9-7 所示。

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig7_HTML.jpg](img/468476_1_En_9_Fig7_HTML.jpg)

图 9-7

在此示例中，我们使用配置来提供密钥，这意味着我们需要将其存储在 Drupal 的配置中，而不是作为文件。先前 HMAC 密钥创建命令的结果反映在“密钥值”字段中。

然后，导航到**配置** ➤ **系统** ➤ **JWT 认证**（`/admin/config/system/jwt`），在此处你可以直接提供密钥（以便存储在配置中）或引用一个可用的文件，如图 9-8 所示。^(⁴⁴)

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig8_HTML.jpg](img/468476_1_En_9_Fig8_HTML.jpg)

图 9-8

一旦让 Drupal 知道该密钥，我们便将其用作 HMAC SHA256 算法的专用密钥。

### 签发和验证 JWT

请记住，我们必须启用 JWT 认证签发者模块，Drupal 才能使用此密钥对 JWT 进行签名并签发它们。启用该签发者模块后，它会暴露一个位于 URI `/jwt/token` 的端点，该端点将为已获得凭证并对其发起请求的消费者生成 JWT。

当我们以用户身份登录后导航到 `/jwt/token` 时，或者当我们使用适当的凭证对 `/jwt/token` 执行 `GET` 请求时，我们会收到一个如下所示的令牌，采用预期的 JWT 标准格式，如图 9-9 所示。有了这个 JWT，我们现在就能够从消费者端向 Drupal 发起经过 JWT 认证的请求。

![../images/468476_1_En_9_Chapter/468476_1_En_9_Fig9_HTML.jpg](img/468476_1_En_9_Fig9_HTML.jpg)

图 9-9

要获取你的令牌，请在提供密钥（JWT 密钥）并启用 JWT 认证签发者模块后，在浏览器中导航到 `/jwt/token`。

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE1MzQ4ODc2MzMsImV4cCI6MTUzNDg5MTIzMywiZHJ1cGFsIjp7InVpZCI6IjEifX0.jIyIIA6wMwGCLE2fyIU1f_Y9e3Nn4rPC3Ta1MzoAKLA
```

### 发起 JWT 认证的请求

下一步是发起一个经过 JWT 认证的请求，以从消费者端检索内容。尽管我们的 Drupal 站点现已配置为可签发和接受 JWT，但我们尚未配置我们的 REST 资源以用于 Drupal 的 JWT 认证。要解决此问题，我们需要遵循在第七章中执行的相同步骤，来配置 Drupal 将如何处理经过认证的请求。幸运的是，第八章中介绍的 REST UI 可以使此过程更加简单，正如我们本章前面在 OAuth 2.0 中看到的那样。

回忆一下我们如何安装并启用核心 REST 模块和 REST UI。

```
$ composer require drupal/restui
$ drush en -y rest restui
```

导航到 Drupal 中的**配置** ➤ **REST**（`/admin/config/services/rest`）以访问 REST UI，在此处我们可以选择核心 REST 可用的任何资源。出于我们有限的目的，我们可以选择编辑“内容”的设置。在“资源内容设置”页面（`/admin/config/services/rest/resource/entity%3Anode/edit`）上，假设你已经正确配置了 JWT 模块，你将看到“认证提供者”列表中多了一个 `jwt_auth` 选项。启用 `jwt_auth` 选项将允许所有内容资源通过 JWT 进行认证。

要测试 JWT 认证的请求，我们可以修改访问内容的权限，使得没有更高访问权限角色的匿名用户无法执行内容的检索（`GET`）。导航到**人员** ➤ **权限**（`/admin/people/permissions`）并禁用匿名用户的**查看已发布内容**权限。

要检索令牌，请以管理员身份登录后导航到 `/jwt/token` 并复制该字符串。然后，就像我们在前几节中使用 OAuth 2.0 一样，为你的消费者添加一个带有 `Bearer` 前缀的 `Authorization` 标头，如下所示。收到 JWT 认证的请求后，Drupal 将验证令牌并提供相应的响应。

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE1MzQ4ODc2MzMsImV4cCI6MTUzNDg5MTIzMywiZHJ1cGFsIjp7InVpZCI6IjEifX0.jIyIIA6wMwGCLE2fyIU1f_Y9e3Nn4rPC3Ta1MzoAKLA
```



### 注意

有关 Drupal 的 JWT 实现内部工作原理以及如何从 PHP 角度生成和验证 JWT 的更多信息，请参阅 Edward Chan 关于在 Drupal 上发出 JWT 认证请求的精彩教程，网址为 [`https://www.mediacurrent.com/blog/using-json-web-tokens-jwt-authenticate-requests-rest-resources-drupal-8`](https://www.mediacurrent.com/blog/using-json-web-tokens-jwt-authenticate-requests-rest-resources-drupal-8)。

### 结论

身份验证对于确保用户的隐私和安全至关重要，尤其是在像解耦式 Drupal 这样更加面向服务——因此本质上安全性更低——的架构中。在本章中，我们详细审视了向 Drupal 8 请求进行身份验证的最常见方法，包括基本身份验证、OAuth2 持有者令牌身份验证和 JWT。此外，我们还介绍了 Drupal 8 中基于 Cookie 的身份验证，这对于需要利用 Drupal 渲染页面的渐进式解耦实现非常有用。

在第 3 部分中，我们将深入探讨 Drupal 的 Web 服务模块（无论是核心模块还是贡献模块）如何处理和响应来自消费者的请求。我们将介绍两种截然不同的使用核心 REST 检索和操作内容的方法：一种更传统的、面向开发者的方法，以及另一种利用 Views 模块、对站点构建者和内容创建者更友好的方法。然后，我们将依次讨论前面提到的每个贡献模块，即 JSON API、RELAXed Web Services 和 GraphQL，评估它们的异同，以便我们能够就选择哪种 Web 服务解决方案做出明智的决策。

脚注 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17

