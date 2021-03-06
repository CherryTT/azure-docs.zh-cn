---
title: 可在 Azure Active Directory B2C 中使用的应用程序类型 | Microsoft Docs
description: 了解有关可在 Azure Active Directory B2C 中使用的应用程序类型。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/01/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 7671a0a99e12463fcce5ff33fbcba7e8677dde05
ms.sourcegitcommit: 00dd50f9528ff6a049a3c5f4abb2f691bf0b355a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/05/2018
ms.locfileid: "51006188"
---
# <a name="applications-types-that-can-be-used-in-active-directory-b2c"></a>可在 Azure Active Directory B2C 中使用的应用程序类型

Azure Active Directory (Azure AD) B2C 支持各种新式应用程序体系结构的身份验证。 所有这些体系结构都以行业标准协议 [OAuth 2.0](active-directory-b2c-reference-protocols.md) 或 [OpenID Connect](active-directory-b2c-reference-protocols.md) 为基础。 本文档介绍了独立于首选语言或平台可构建的应用程序类型。 在开始构建应用程序之前，不妨从中了解一些高级方案。

必须通过 [Azure 门户](https://portal.azure.com/)将使用 Azure AD B2C 的每个应用程序注册到 [Azure AD B2C 租户](active-directory-b2c-get-started.md)中。 应用程序注册过程将收集和分配一些值，例如：

* 用于唯一标识应用程序的应用程序 ID。
* 可用于将响应定向回应用程序的**回复 URL**。

发送到 Azure AD B2C 的每个请求都指定了一个 **策略**。 策略控制 Azure AD 的行为。 也可以使用这些终结点来创建一系列高度可自定义的用户体验。 常见的策略包括注册、登录和配置文件编辑策略。 如果不熟悉策略，请先了解 Azure AD B2C 的 [可扩展策略框架](active-directory-b2c-reference-policies.md) ，再继续下一步。

每个应用程序的交互遵循类似的高级模式：

1. 应用程序将用户定向到 v2.0 终结点以执行[策略](active-directory-b2c-reference-policies.md)。
2. 用户根据策略定义完成策略。
3. 应用程序从 v2.0 终结点接收安全令牌。
4. 应用程序使用该安全令牌访问受保护的信息或受保护的资源。
5. 资源服务器验证安全令牌，确认是否可以授予访问权限。
6. 应用程序定期刷新安全令牌。

根据要构建的应用程序类型，这些步骤可能稍有不同。

## <a name="web-applications"></a>Web 应用程序

对于托管在服务器中通过浏览器访问的 Web 应用程序（包括 .NET、PHP、Java、Ruby、Python 和 Node.js），Azure AD B2C 支持使用 [OpenID Connect](active-directory-b2c-reference-protocols.md) 实现所有用户体验。 这包括登录、注册和配置文件管理。 在 OpenID Connect 的 Azure AD B2C 实现中，Web 应用程序通过向 Azure AD 发出身份验证请求，来发起这些用户体验。 请求的结果是 `id_token`。 此安全令牌代表用户的标识。 它还以声明形式提供用户的相关信息：

```
// Partial raw id_token
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...

// Partial content of a decoded id_token
{
    "name": "John Smith",
    "email": "john.smith@gmail.com",
    "oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
    ...
}
```

请参阅 [Azure AD B2C 令牌参考](active-directory-b2c-reference-tokens.md)，详细了解应用程序可用的令牌和声明类型。

在 Web 应用程序中，每次执行[策略](active-directory-b2c-reference-policies.md)都要采用以下高级步骤：

1. 用户浏览到 Web 应用程序。
2. Web 应用程序将用户重定向到指示要执行的策略的 Azure AD B2C。
3. 用户完成策略。
4. Azure AD B2C 将 `id_token` 返回到浏览器。
5. `id_token` 发布到重定向 URI。
6. 验证 `id_token` 并设置会话 Cookie。
7. 安全页返回至用户。

使用从 Azure AD 收到的公共签名密钥来验证 `id_token` ，就足以验证用户的标识。 这也会设置可在后续页面请求中用于识别用户的会话 Cookie。

若要查看此方案的实际运行情况，请尝试运行 [入门部分](active-directory-b2c-overview.md)中提供的 Web 应用程序登录代码示例之一。

除了简化登录，Web 服务器应用程序可能还需要访问后端 Web 服务。 在此情况下，Web 应用程序可以执行稍有不同的 [OpenID Connect 流](active-directory-b2c-reference-oidc.md)，使用授权代码和刷新令牌来获取令牌。 以下 [Web API 部分](#web-apis)描述了此方案。

## <a name="web-apis"></a>Web API

可使用 Azure AD B2C 保护 Web 服务，例如应用程序的 RESTful Web API。 Web API 可以使用 OAuth 2.0 保护其数据，使用令牌对传入的 HTTP 请求进行身份验证。 Web API 的调用方在 HTTP 请求的授权标头中附加一个令牌：

```
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
```

然后，Web API 就可以使用此令牌来验证 API 调用方的标识，并从令牌中编码的声明里提取调用方的相关信息。 请参阅 [Azure AD B2C token reference](active-directory-b2c-reference-tokens.md)（Azure AD B2C 令牌参考），详细了解应用可用的令牌和声明类型。

Web API 可从许多类型的客户端（包括 Web 应用程序、桌面和移动应用程序、单页应用程序、服务器端守护程序，甚至其他 Web API）接收令牌。 下面是 Web 应用程序调用 Web API 的完整流程示例：

1. Web 应用程序执行策略，用户完成用户体验。
2. Azure AD B2C 将 `access_token` 和授权代码返回到浏览器。
3. 浏览器将 `access_token` 和授权代码发布到重定向 URI。
4. Web 服务器验证 `access token` 并设置会话 Cookie。
5. `access_token` 向 Azure AD B2C 提供授权代码、应用程序客户端 ID 和凭据。
6. `access_token` 和 `refresh_token` 返回到 Web 服务器。
7. 使用授权标头中的 `access_token` 调用 Web API。
8. Web API 对令牌进行验证。
9. 安全数据返回到 Web 服务器。

有关授权代码、刷新令牌的详细信息和获取令牌的步骤，请参阅 [OAuth 2.0 protocol](active-directory-b2c-reference-oauth-code.md)（OAuth 2.0 协议）。

若要了解如何使用 Azure AD B2C 保护 Web API，请查看 [入门部分](active-directory-b2c-overview.md)中的 Web API 教程。

## <a name="mobile-and-native-applications"></a>移动和本机应用程序

安装在设备中的应用程序（例如移动和桌面应用程序）通常需要代表用户访问后端服务或 Web API。 可将自定义的标识管理体验添加到本机应用程序，使用 Azure AD B2C 和 [OAuth 2.0 授权代码流](active-directory-b2c-reference-oauth-code.md)安全调用后端服务。  

在此流中，应用程序执行[策略](active-directory-b2c-reference-policies.md)，在用户完成策略之后，从 Azure AD 接收 `authorization_code`。 `authorization_code` 表示应用程序有权代表当前登录的用户调用后端服务。 然后，该应用程序即可在后台将 `authorization_code` 交换成 `id_token` 和 `refresh_token`。  应用程序可以在 HTTP 请求中使用 `id_token` 向后端 Web API 进行身份验证。 它还可以使用 `refresh_token` 获取新的 `id_token`（如果旧令牌已过期）。

## <a name="current-limitations"></a>当前限制

### <a name="application-not-supported"></a>不支持的应用程序 

#### <a name="daemonsserver-side-applications"></a>守护程序/服务器端应用程序

包含长时运行进程或不需要用户操作的应用程序还需要通过其他方法访问受保护的资源，例如 Web API。 这些应用程序可使用应用程序的标识（而不是用户的委派标识）并使用 OAuth 2.0 客户端凭据流来进行身份验证和获取令牌。 客户端凭据流与代表流不同，代表流不会应用于服务器到服务器的身份验证。

虽然 Azure AD B2C 当前不支持客户端凭据流，但可使用 Azure AD 设置客户端凭据流。 Azure AD B2C 租户与 Azure AD 企业租户共享某些功能。  使用 Azure AD B2C 租户的 Azure AD 功能支持客户端凭据流。 

若要设置客户端凭据流，请参阅 [Azure Active Directory v2.0 和 OAuth 2.0 客户端凭据流](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds)。 如 [Azure AD 令牌参考](https://docs.microsoft.com/azure/active-directory/develop/active-directory-token-and-claims)中所述，身份验证成功后会收到格式化的令牌，以便 Azure AD 可使用它。

#### <a name="web-api-chains-on-behalf-of-flow"></a>Web API 链（代理流）

许多体系结构包含需要调用另一个下游 Web API 的 Web API，这两者都受 Azure AD B2C 的保护。 此方案常见于包含 Web API 后端的本机客户端。 然后，此 Web API 将调用 Azure AD 图形 API 等 Microsoft 联机服务。

可以使用 OAuth 2.0 JWT 持有者凭据授权（也称为“代理流”）来支持这种链接的 Web API 方案。  但是，Azure AD B2C 中目前尚未实现代理流。

### <a name="reply-url-values"></a>回复 URL 值

使用 Azure AD B2C 注册的应用目前仅限使用一组有限的回复 URL 值。 Web 应用和服务的回复 URL 必须以方案 `https` 开头，并且所有回复 URL 值必须共享一个 DNS 域。 例如，无法注册具有以下回复 URL 的 Web 应用：

`https://login-east.contoso.com`

`https://login-west.contoso.com`

注册系统会将现有回复 URL 的完整 DNS 名称与要添加的回复 URL 的 DNS 名称相比较。 如果满足以下任一条件，添加 DNS 名称的请求会失败：

- 新回复 URL 的完整 DNS 名称与现有回复 URL 的 DNS 名称不匹配。
- 新回复 URL 的完整 DNS 名称不是现有回复 URL 的子域。

例如，如果应用具有以下回复 URL：

`https://login.contoso.com`

可以向其添加，如下所示：

`https://login.contoso.com/new`

在这种情况下，DNS 名称将完全匹配。 或者，可以执行下面的操作：

`https://new.login.contoso.com`

在这种情况下，将引用 login.contoso.com 的 DNS 子域。 如果希望应用使用 login-east.contoso.com 和 login-west.contoso.com 作为回复 URL，必须按以下顺序添加这些回复 URL：

`https://contoso.com`

`https://login-east.contoso.com`

`https://login-west.contoso.com`

可以添加后两个回复 URL，因为它们是第一个回复 URL (contoso.com) 的子域。 

创建移动/本机应用程序时，可以定义**重定向 URI** 而不是**重播 URL**。 选择重定向 URI 时，有两个重要的考虑事项：

- **唯一**：每个应用程序的重定向 URI 的方案应是唯一的。 在示例 `com.onmicrosoft.contoso.appname://redirect/path` 中，`com.onmicrosoft.contoso.appname` 为方案。 应遵循此模式。 如果两个应用程序共享同一方案，用户会看到“选择应用”对话框。 如果用户的选择不正确，登录会失败。
- **完整**：重定向 URI 必须同时包含方案和路径。 路径必须在域之后包含至少一个正斜杠。 例如，`//contoso/` 有效，而 `//contoso` 会失败。 请确保重定向 URI 中没有下划线等特殊字符。

### <a name="faulted-apps"></a>出错的应用

不应按以下方式编辑 Azure AD B2C 应用程序：

- 在其他应用程序管理门户（如 [应用程序注册门户](https://apps.dev.microsoft.com/)）中编辑。
- 使用图形 API 或 PowerShell 编辑。

如果在 Azure 门户外部编辑 Azure AD B2C 应用程序，它将成为出错的应用程序，并且不再可用于 Azure AD B2C。 需删除应用程序，然后重新创建它。

若要删除应用程序，请转到[应用程序注册门户](https://apps.dev.microsoft.com/)并在该处删除应用程序。 若要使应用程序可见，须为该应用程序的所有者（而不仅仅是租户管理员）。

