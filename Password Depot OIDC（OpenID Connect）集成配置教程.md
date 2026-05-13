# Password Depot OIDC（OpenID Connect）集成配置教程（用于 Enterprise Server 测试）

OpenID Connect（OIDC）是一个基于 OAuth 2.0 的身份认证协议，**为企业提供与各类现代身份提供方（IdP）的标准对接能力**。通过 OIDC，Password Depot 允许你使用来自 **Auth0、Entra ID、PingIdentity 或其他兼容 OIDC 服务的账号**直接登录和同步用户，无需在 Password Depot 中重复建立账户[reference:0]。

> **官方指引**：本配置界面与说明均基于 Password Depot 官方帮助文档——[Import from OpenID Connect](https://www.password-depot.de/en/documentation/onlinehelp-enterprise-server/openid-connect-synch-en.htm)。当用户希望通过 **单点登录（SSO）** 访问 Enterprise Server 时，必须进行 OIDC 导入配置[reference:1]。

---

## 一、核心前提：在身份提供方（IdP）中注册 Password Depot

在所有 OIDC 配置之前，无论你使用哪个身份提供方，都必须先在 IdP 端完成 Password Depot 的 **应用注册（App Registration）**，并获取以下关键信息：

| 必填项 | 说明 |
|-------|------|
| **Application (Client) ID** | IdP 为 Password Depot 分配的唯一客户端标识符 |
| **Client Secret** | 用于 API 请求的客户端密码（密钥管理应遵循安全实践，通常通过安全的密钥保管库注入，切勿明文硬编码[reference:2]） |
| **Redirect URL / Callback URI** | IdP 在认证完成后将用户重定向回的地址；格式必须完全一致[reference:3] |

> 以 **Microsoft Entra ID** 为例，将重定向 URI 配置为 `https://{你的PD服务器地址}:25019/oidc/callback`，并在 **API 权限**中添加 `User.Read.All`、`openid` 和 `profile`，随后必须点击 **“授予管理员同意”**[reference:4]。若希望启用隐式授权流（Implicit Flow）以支持 Password Depot 早期的测试版本，需在应用注册的 **身份验证 → 隐式授权和混合流** 中勾选“访问令牌”与“ID 令牌”[reference:5]。

---

## 二、Password Depot Server Manager 中的 OIDC 配置

在 Password Depot Server Manager 主界面，点击顶部菜单 **工具 → 从 OIDC 导入（Import from OIDC）**，即可打开 OIDC 配置向导[reference:6]。

在向导窗口中，点击 **新建** 按钮添加一个新的身份提供方配置。

“新建”窗口包含以下三个选项卡[reference:7]：

### 2.1 【General】基本设置选项卡

| 字段 | 说明 |
|------|------|
| **名称（Name）** | 当前 IdP 配置的别名，内部使用，例如 `MyCompany Entra ID`[reference:8] |
| **提供商（Provider）** | 下拉列表可选 `OIDC`（通用）、`Entra ID`、`Auth0`、`PingIdentity` 等[reference:9] |
| **发现端点（Discovery endpoint）** | OIDC 发布元数据（认证、令牌、用户信息端点）的标准 URL。格式为 `https://{域名}/{租户ID}/.well-known/openid-configuration`[reference:10] |
| **应用程序(客户端) ID（Application (client) ID）** | 上一步注册过程中从 IdP 获得的 Client ID[reference:11] |
| **重定向 URL（Redirect URL）** | 认证成功后 IdP 回传控制权的地址，**必须与 IdP 上注册的 URI 完全一致**[reference:12] |
| **测试客户端登录（Test client login）** | 使用当前配置发起一次完整的 OIDC 认证测试，以验证参数正确性[reference:13] |

> **测试登录注意事项**：点击“测试客户端登录”时，若系统弹出浏览器 Auth0/Entra ID 等登录页，需要先在 IdP 中确认是否已正确配置重定向 URI 及静默许可。

### 2.2 【Advanced】高级设置选项卡

| 字段 | 说明 |
|------|------|
| **响应类型（Response type）** | 定义返回值类型；推荐 OIDC 授权码流程使用 `code`，避免使用 `id_token` 或 `token`[reference:14] |
| **作用域（Scopes）** | 设置请求的信息范围；至少应包含 `openid profile email`，其他如 `groups` 等可根据需求添加[reference:15] |
| **属性映射（Attributes mapping）** | 来自令牌的声明（Claims）映射到 Password Depot 内部用户属性（如邮箱、名、姓等）[reference:16] |

### 2.3 【Management API】管理 API 选项卡

这部分用于后台管理操作（例如批量导入或同步用户组）：

| 字段 | 说明 |
|------|------|
| **API 基本 URL（API Base URL）** | IdP 管理 API 的入口地址，一般可从 IdP 文档中查到[reference:17] |
| **客户端密钥（Client Secret）** | 在应用注册时生成的客户端密钥，用于 API 请求中获取访问令牌（承载在应用程序凭据中）[reference:18] |

> **填入客户端密钥后，必须点击“测试客户端登录”按钮验证是否配置成功，确保应用与 IdP 之间可以正常交换令牌**。

---

## 三、针对不同身份提供方的通用配置要点

### 3.1 使用 Microsoft Entra ID（原 Azure AD）作为 OIDC 提供方

1. 进入 **Microsoft Entra 管理中心** → **应用程序注册** → 新建注册。
2. 设置重定向 URI（例如 `https://<server_address>:25019/oidc/callback`）并保存。
3. 记录 **Directory (tenant) ID**、**Application (client) ID**，并生成 **Client Secret**。
4. 构造发现端点 URL：
