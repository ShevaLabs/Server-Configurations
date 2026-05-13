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
4. 构造发现端点 URL：https://login.microsoftonline.com/{租户ID}/v2.0/.well-known/openid-configuration
5. 在 Password Depot OIDC 向导中填入上述四组关键信息。
6. 若遇到 `AADSTS700054: response_type 'id_token' not enabled` 错误，请在 Entra ID 应用注册 → **身份验证** → **隐式授权和混合流** 中勾选 **ID 令牌**，或在清单中将 `oauth2AllowIdTokenImplicitFlow` 设为 `true`。

### 3.2 使用 Auth0 作为 OIDC 提供方

1. 在 [Auth0 管理仪表板](https://manage.auth0.com/) 中创建 **Regular Web Application**（常规 Web 应用）。
2. 在应用 **设置** → **允许回调 URL** 中添加重定向 URI：`https://<PD服务器地址>:25019/oidc/callback`。
3. 记录 **Domain**、**Client ID** 和 **Client Secret**。
4. 发现端点 URL 格式为：https://{你的Auth0认证域}/.well-known/openid-configuration

5. 在 Password Depot OIDC 向导的 **Management API** 选项卡中，API Base URL 留空基本可用，并填入 Client Secret[reference:19]。

### 3.3 使用 Keycloak 自建本地 OIDC 提供方

Keycloak 是目前主要推荐的开源自建 IdP 选择（社区活跃且持续更新），如果你需要完全掌控数据和协议行为，可以使用 Docker 快速搭建一个测试环境[reference:20]。

1. 以 Docker 启动 Keycloak（示例命令）[reference:21]：
```bash
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev
```
2. 登录管理控制台 http://localhost:8080，创建一个新的 Realm（租户）。

3. 在 Clients 菜单中创建一个新的 OIDC Client：

- 设置 Client ID

- 启用 Client authentication

- 添加重定向 URI：https://<PD服务器地址>:25019/oidc/callback

4. 记录 Client ID 和 Client Secret（也可以在 Credentials 中查看）。

5. 在 Realm Settings → General 中找到发现端点（通常是 http://localhost:8080/realms/{你的Realm名}/.well-known/openid-configuration）。

6. 在 Password Depot OIDC 向导中填入上述信息进行导入。

### 3.4 使用其他通用 OIDC 服务

如果选择 OIDC 作为提供商（通用）：在向导的 General 选项卡的 Provider 下拉菜单中，手动选择 OIDC（而非 Entra ID 或 Auth0），然后手动填入发现端点、Application (client) ID、Client Secret 以及重定向 URL。

在 Advanced 选项卡中，确保 Scopes 包含 openid profile email；在 Management API 选项卡中，根据 IdP 文档填写 API Base URL 和 Client Secret。

## 四、导入 OIDC 用户与同步管理

### 4.1 从 OIDC 导入用户

1. 在 Server Manager 中，进入 `工具 → 从 OIDC 导入`。
2. 选择已创建的 OIDC 身份提供方，点击 **导入**。
3. 系统将调用 OIDC IdP 的 API，列出可用的用户组和用户对象。
4. 勾选需要导入的目标对象，点击 **导入** 完成用户同步。

> **导入检查**：确保在 IdP 侧已为所选用户分配至少一个应用角色，并且 `email` 不为空，否则 Password Depot 可能会跳过这些用户。

### 4.2 配置自动同步（可选）

虽然 Password Depot 官方文档没有详细描述 OIDC 的完整自动同步机制，但从测试角度可以验证：

1. 进入 `管理 → 服务器设置 → OpenID Connect`（如有相应选项卡），观察是否存在类似 AD / Entra ID 的自动同步间隔设置。
2. 如果不可用，可通过 **计划任务** + **命令行客户端（例如 pdcmd）** 实现周期性的 `Import from OIDC`。
3. 如果配置成功，OIDC 中修改的用户属性（如显示名称）应能定期同步到 Password Depot 用户列表中。

---

## 五、客户端登录测试

### 5.1 通过 Password Depot Client 登录

1. 打开 Password Depot **Windows Client Corporate Edition**。
2. 在登录界面选择认证方式：
   - **OpenID Connect**（通用 OIDC）
   - 或直接从 UI 中选择 **Entra ID** / **Auth0**（若提前用对应 IdP 配置了 OIDC）。
3. 输入服务器地址和端口，点击登录。
4. 浏览器将重定向到 IdP 的登录页，使用导入用户的 IdP 凭证进行验证。
5. 认证成功后浏览器提示返回应用程序，客户端自动完成登录。

### 5.2 在 Server Manager 中测试

1. 登出当前 Server Manager 会话。
2. 在登录下拉列表中选择 **OpenID Connect**。
3. 输入服务器端口，点击登录自动触发 OIDC 流程。
4. 完成身份提供方的授权后返回 Server Manager，并以对应 OIDC 用户身份进入管理控制台。

---

## 六、常见问题排查

### 6.1 导入用户失败：403 Forbidden

- 最常见的原因是缺少 **管理员授权（Admin Consent）** 或未在 IdP 中为应用配置 **User.Read.All**（针对 Microsoft Entra ID）。解决方法：在 IdP 端确认 **应用程序权限** 已正确授予并触发管理员同意。

### 6.2 登录时报错 `AADSTS700054`

- 提示 `response_type 'id_token' not enabled for the application`。解决方法：在 Entra ID 的应用注册 → **身份验证** → **隐式授权和混合流** 中勾选 **ID 令牌**。或者在清单中将 `oauth2AllowIdTokenImplicitFlow` 设置为 `true` 后重新测试。

### 6.3 重定向 URI 不匹配错误

- 检查 Password Depot 里 **Redirect URL** 是否与该身份提供方侧注册的回调地址完全一致，路径和端口均不能多一个或少一个字符。

### 6.4 相同用户从不同 IdP 导入后无法选择正确的认证方式（已知缺陷）

- **已知缺陷**（BUG-SERVER-07）：如果同一外部用户通过 Entra ID 和 OIDC 分别导入，用户列表的“身份验证”列会同时显示两种认证方式，且属性的 **Account** 选项卡下“OK”按钮完全禁用，无法保存修改。在官方修复之前，建议仅使用一种导入路径，避免出现用户状态卡死的问题。

### 6.5 自动同步不生效

- 如果 OIDC 的自动同步界面不可用，可考虑通过外部调度器（Windows 任务计划器、Jenkins 等）调用 `import from OIDC` 命令行实现周期同步。同时留意 IdP 侧访问令牌是否过期，如果过期需要刷新获取新的 Client Secret。

---

## 七、安全快照与验证建议

- **环境快照**：在完成 OIDC 配置和用户导入后，建议对虚拟机或物理服务器做一次完整快照，以防 OIDC 参数冲突时无法恢复。
- **定期检查 OIDC 连接**：建议建立回归测试计划，定期执行“测试客户端登录”，以确保 IdP 和 Password Depot 之间的信任关系未因证书更换或 Client Secret 过期而失效。
- **分级策略**：生产环境推荐优先在测试环境中逐步推广 OIDC SSO，验证所有 IdP → Password Depot → 客户端登录链路完全无异常后再投入正式使用。

---

**文档版本**：1.0  
**最后更新**：2026-05-13  
**适用版本**：Password Depot Enterprise Server 19.1.0  
**兼容 OIDC 提供方**：Microsoft Entra ID、Auth0、Keycloak、PingIdentity，或任何兼容 OIDC 协议的自建 IdP
