# Microsoft Entra ID 配置教程（用于 Password Depot 集成测试）

Microsoft Entra ID（原 Azure AD）是 Microsoft 基于云的标识管理产品，可作为 Password Depot Enterprise Server 的身份提供方，允许用户使用其 Microsoft 凭证登录 Password Depot。本教程基于 Password Depot Enterprise Server 19.1.0，指导您快速搭建一个免费的 Entra ID 测试环境，完成应用注册与集成配置。

---

## 一、前提条件

| 项目 | 要求 |
|------|------|
| Microsoft 账户 | 任一有效的个人或工作账户（如 Outlook 邮箱） |
| 浏览器 | 推荐 Chrome / Edge / Firefox 最新版本 |
| 网络 | 能够访问 Azure 门户（portal.azure.com）及 Microsoft Entra 管理中心 |
| Password Depot Server | 已安装 Enterprise Server 19.1.0 并拥有管理员权限 |

> 本教程使用 **免费的 Microsoft Entra ID 租户（Free tier）**，每个租户默认可创建 **50,000 个** Microsoft Entra 资源[reference:0]，足以满足测试需求[reference:1]。建议使用独立的 Microsoft 账户创建测试租户，与生产环境隔离。

---

## 二、创建免费 Entra ID 租户

### 2.1 登录 Azure 门户

1. 使用您的 Microsoft 账户登录 [Azure 门户](https://portal.azure.com)
2. 搜索并选择 **Microsoft Entra ID** 服务

### 2.2 创建新租户

1. 导航到 **Entra ID** → **概述 (Overview)** → **管理租户 (Manage tenants)**[reference:2]
2. 点击顶部 **创建 (Create)** 按钮
3. 在 **基本信息 (Basics)** 选项卡中，选择租户类型为 **Microsoft Entra ID**[reference:3]
4. 填写组织名称（如 `PasswordDepotTest`）和初始域名（如 `yournametest.onmicrosoft.com`）
5. 选择国家/地区（建议选择实际所在区域）
6. 点击 **下一步**，完成配置后点击 **创建**

> 创建完成后，新建租户会出现在管理租户列表页面。**创建租户的用户会自动分配全局管理员 (Global Administrator) 角色**[reference:4][reference:5]。

### 2.3 切换到新租户

在 Azure 门户右上方点击您的账户头像，选择 **目录 + 订阅 (Directories + subscriptions)**，切换至刚创建的新租户。

---

## 三、在 Entra ID 中创建测试用户

### 3.1 进入用户管理页面

在 Azure 门户的 **Microsoft Entra ID** 服务中，导航到 **管理 (Manage)** → **用户 (Users)** → **所有用户 (All users)**。

### 3.2 创建新用户

1. 点击 **新建用户 (New user)** → **创建新用户 (Create new user)**
2. 填写用户信息（以 `test01` 为例）：[reference:6]
   - **用户主体名称 (User principal name)**：`test01@yourtenant.onmicrosoft.com`
   - **显示名称 (Display name)**：`Test User 01`
   - **密码设置**：勾选 **自动生成密码 (Auto-generate password)** 或自行设置
3. 可根据需要为用户分配角色。对于完成 Entra ID 导入后的用户权限，建议先不授予目录级别的高级角色，以更好测试 Password Depot 内部权限隔离。
4. 点击 **创建**

重复上述步骤，创建多个测试用户（如 `test02`、`test03`）用于后续测试。

### 3.3 批量创建用户（可选）

使用 PowerShell Graph API 脚本可在 Entra ID 中自动化批量创建用户。

---

## 四、注册 Password Depot 企业应用程序

**官方文档引用**：本步骤详细说明参见 Password Depot 官方帮助文档《[Adding Password Depot Enterprise Server as an Enterprise Application in Microsoft Entra ID](https://www.password-depot.de/en/documentation/onlinehelp-enterprise-server/add-pd-to-azure-en.htm)》。

### 4.1 通过应用程序库添加

1. 在 Microsoft Entra ID 服务中，导航到 **管理 (Manage)** → **企业应用程序 (Enterprise applications)**
2. 点击 **新建应用程序 (New application)**
3. 在 **从库中添加 (Add from the gallery)** 搜索框中输入 `Password Depot Enterprise Server`，按 Enter 搜索[reference:7]
4. 从搜索结果中选择 **Password Depot Enterprise Server**，点击 **添加 (Add)**[reference:8]

> **注意**：应用添加到企业应用程序后，必须通过 **分配用户和组 (Assign users and groups)** 功能将管理员分配给该应用。只有这样，管理员才能在 Password Depot 中执行从 Entra ID 导入用户的操作[reference:9]。

### 4.2 若应用程序不在库中（备选方法）

如果 Password Depot Enterprise Server 未出现在应用库中（可能是由于区域或某些企业限制），请使用以下方法：
- 在 Password Depot Server Manager 中，导航到 **管理 (Manage)** → **服务器设置 (Server Settings)** → **Microsoft Entra ID**
- 点击 **新建... (New...)**，使用管理员账户登录
- 该应用程序将**自动添加**到企业应用程序列表中[reference:10]

### 4.3 将管理员分配给应用程序

- 在 **企业应用程序 (Enterprise applications)** 中找到刚添加的 Password Depot Enterprise Server 应用
- 点击 **分配用户和组 (Assign users and groups)**，添加您的测试租户全局管理员账户，使其能够执行 Entra ID 导入[reference:11]。
- 如果**找不到分配入口**，需确保您登录的账号已分配该企业应用程序的用户访问权限。

---

## 五、获取 OIDC 配置所需的关键信息

Password Depot 支持通过 **OpenID Connect (OIDC)** 与 Entra ID 集成。完成应用注册后，需要向 Password Depot Server Manager 提供以下参数[reference:12]：

| 参数 | 说明 | 获取位置 |
|------|------|----------|
| **Directory (tenant) ID** | Entra ID 租户的唯一标识符 | Azure 门户 → Microsoft Entra ID → **概述 (Overview)** → **租户 ID (Tenant ID)** |
| **Application (client) ID** | 应用程序的唯一标识符 | Azure 门户 → **应用注册 (App registrations)** → 选择应用程序 → **概述 (Overview)** → **应用程序(客户端) ID (Application (client) ID)** |
| **Client Secret** | 应用程序的客户端密钥 | Azure 门户 → **应用注册 (App registrations)** → 选择应用程序 → **证书和密码 (Certificates & secrets)** → **新建客户端密码 (New client secret)** → 立即复制保存[reference:13] |
| **Discovery endpoint** | OpenID Connect 配置元数据端点 | `https://login.microsoftonline.com/{tenant_id}/v2.0/.well-known/openid-configuration`（将 `{tenant_id}` 替换为 Directory (tenant) ID） |

> **Client Secret 生成后只显示一次，请务必即时保存**。该密钥将用于 Password Depot OIDC 向导中，用于完成 API 请求认证[reference:14]。若丢失，只能重新生成新的 Client Secret。

---

## 六、在 Password Depot Server 中配置 Entra ID 集成

此部分包含 **从 Azure AD 导入 (Import from Azure AD)** 和 **从 OIDC 导入 (Import from OIDC)** 两种连接的配置方式。根据测试目的的不同（普通 Entra ID 用户导入 vs. OIDC 身份验证），可择一或全部配置。

官方文档指出，Entra ID 用户**只能通过此导入过程**添加，不能在 Server Manager 中手动创建[reference:15]。

### 6.1 配置 Entra ID 连接（从 Azure AD 导入）

1. 打开 **Password Depot Server Manager**
2. 导航到 **工具 (Tools)** → **从 Entra ID 导入 (Import from Entra ID)**[reference:16]
3. 若未列出组织机构，点击 **新建... (New...)** 添加新租户[reference:17]
4. 使用**管理员账户**（在步骤 4.3 中已分配给企业应用程序的账户）登录[reference:18]
5. 系统会提示进行**第二步验证**（MFA），按照指引完成验证。这是 Microsoft 安全策略的强制要求，无法绕过[reference:19]。
6. 成功登录后，进入搜索页面
   - **搜索并选择用户/组**：在搜索框中输入关键字，或留空后点击 **立即搜索 (Search now)** 以显示所有可用条目
   - **勾选**：选择要导入的用户和组，支持多选和全选右击操作[reference:20]
7. **处理已删除对象 (Process Deleted Objects)**
   - 如果启用了 **检查已删除对象 (Check deleted objects)** 选项且检测到已删除对象，系统会显示一个额外的操作页面[reference:21]
   - 对于每个检测到的已删除对象，可分别选择执行操作（忽略、禁用或删除），然后点击对应操作按钮完成最终批量导入[reference:22]
8. 点击 **导入 (Import)** / **下一步 (Next)**，等待导入完成
9. 导入完成后，检查该用户属性下的 **Entra ID** 选项卡，显示已从 Entra ID 自动填充的属性。相应的 Account 选项卡中 **Entra ID** 复选框应默认选中[reference:23]。


### 6.2 配置 OIDC 连接（从 OIDC 导入）——使 Entra ID 用户通过 OIDC 身份验证

如果希望用户在 Password Depot 客户端使用 **OpenID Connect** 方式进行现代 SSO 身份验证而非传统 Entra ID 导入，可将 Entra ID 配置为 OIDC 身份提供方，通过 OIDC 标准和 Password Depot 连接。

这一方式使用的是 **OIDC 授权码流程 (Authorization Code Flow)**，而非传统 Azure AD 导入所使用的专有 API 流程。两者的主要差异如下：
- **Entra ID 导入 (Import from Entra ID)**：Password Depot Server 通过 Microsoft Graph API 拉取用户信息，更适合后台同步和用户管理
- **OIDC 导入 (Import from OIDC)**：仅需 Discovery endpoint 和 Client ID/Secret 即可完成身份验证，用户在 Password Depot 客户端选择 **OpenID Connect** 方式进行 SSO 登录

配置步骤如下[reference:24]：

1. 在 Server Manager 中导航到 **工具 (Tools)** → **从 OIDC 导入 (Import from OIDC)**
2. 点击 **新建... (New...)**，在弹出的窗口 **General** 选项卡中
   - **名称 (Name)**：输入一个内部标识名称，如 `My Entra ID OIDC`
   - **提供商 (Provider)**：从下拉菜单中选择 **Entra ID**；如果产品未来按标准 OIDC 通用配置显示，也可选择 **OIDC** 手动填写[reference:25]
   - **发现端点 (Discovery endpoint)**：格式为 `https://login.microsoftonline.com/{tenant_id}/v2.0/.well-known/openid-configuration`，将 `{tenant_id}` 替换为您在第五步获取的 **Directory (tenant) ID**[reference:26]
   - **应用程序(客户端) ID (Application (client) ID)**：填入第五步记录的 **Application (client) ID**[reference:27]
   - **重定向 URL (Redirect URL)**：此字段会自动填充，但应确保与 Entra ID 中配置的重定向 URI 一致。若不一致，请在 Entra ID 应用注册的 **身份验证 (Authentication)** → **重定向 URI (Redirect URIs)** 中添加并匹配[reference:28]
3. 切换到 **高级 (Advanced)** 选项卡
   - **响应类型 (Response type)**：保持默认 `code`；**不要选择** `token` 或 `id_token`，因为现代 OIDC 集成推荐的是授权码流程，而非隐式流程
   - **Scopes (Scopes)**：请确认包含 **`openid` `profile` `email`**；如不完整，手工添加这些值。这将决定从 Entra ID 请求的用户信息字段[reference:29]
   - **属性映射 (Attributes mapping)**：默认映射通常可满足需求，可根据需要调整[reference:30]
4. 切换到 **管理 API (Management API)** 选项卡
   - **API 基础 URL (API Base URL)**：在将 Entra ID 用作 OIDC 提供方时，与 Microsoft Graph API 交互的后台管理通常需要额外端点。请务必填写该 Base URL[reference:31]
   - **客户端密钥 (Client Secret)**：填入第五步生成的 **Client Secret**，用于在后台应用 API 请求中获取访问令牌[reference:32]
5. 点击 **测试客户端登录 (Test client login)** 验证配置是否正确。测试登录成功后，即可执行从 OIDC 导入 Entra ID 用户。

**启用 `id_token` 以支持隐式授权流（针对较早版本 Password Depot）**
- 若 Password Depot 应用程序仍依赖隐式令牌流程（通常用于较早的客户端或特定配置），则可能在登录过程收到 **AADSTS700054**：`response_type 'id_token' is not enabled for the application` 错误[reference:33]。
- **解决方案**：在 Entra ID 中找到该应用注册，在 **身份验证 (Authentication)** 页面 → **隐式授权和混合流 (Implicit grant and hybrid flows)** 部分 → 勾选 **访问令牌 (access tokens)** 和 **ID 令牌 (ID tokens)**，或编辑清单 (Manifest) 将 `oauth2AllowImplicitFlow` 和 `oauth2AllowIdTokenImplicitFlow` 设置为 `true`[reference:34][reference:35]。
- 同时，需确保在 API 权限中添加了 Microsoft Graph 权限（以应用程序身份委托 `User.Read.All`、`Group.Read.All` 等）并 **授予管理员同意**[reference:36]。

### 6.3 验证导入的用户

在 Server Manager 的 **用户 (Users)** 区域，检查导入的用户：
- 对于通过 Entra ID 导入的用户：**身份验证 (Authentication)** 列显示 `Entra ID`，属性中的 **Account** 选项卡应默认勾选 `Entra ID` 选项[reference:37]
- 对于通过 OIDC 导入的用户：**身份验证 (Authentication)** 列显示 `OpenID Connect`，属性中的 **Account** 选项卡应默认勾选 `OpenID Connect` 选项[reference:38]

---

## 七、启用自动同步

1. 在 Server Manager 中导航到 **管理 (Manage)** → **服务器设置 (Server Settings)** → **Entra ID** 选项卡
2. 找到 **同步 (Synchronization)** 部分
3. 勾选 **启用自动同步 (Automatically run synchronization with AD)**，并设置同步间隔（如 60 分钟）
4. 点击 **确定 (OK)** 保存设置

> **注意事项**：
> - 自动同步功能仅能同步用户的属性变更（如显示名称、部门、邮箱地址等），**无法自动删除已在 Entra ID 中被删除的用户**。如需删除已删除的 Entra ID 用户，可在手动导入时启用 **检查已删除对象 (Check deleted objects)** 进行处理[reference:39]，或通过 Server Manager 手动禁用/删除[reference:40]。
> - 同步前至少已对每个用户完成至少一次**全量手动导入 (Import from Entra ID)**，以确保用户信息完整同步到 Server Manager。


---

## 八、测试 Entra ID 登录

### 8.1 在 Server Manager 中测试 Entra ID 登录

1. 退出当前 Server Manager 会话
2. 在登录界面，选择 **认证方式 (Authentication)** 下拉菜单中的 **Microsoft Entra ID**[reference:41]
3. 输入服务器地址和端口
4. 用户名会自动填充保存的设置，无需输入密码，认证由 Microsoft Entra ID 处理[reference:42]
5. 点击 **登录 (LOG IN)** 并完成 Microsoft 认证流程

### 8.2 在 Windows 客户端中测试 Entra ID 登录

1. 打开 Password Depot Windows Client Corporate Edition
2. 在登录界面，选择认证方式为 **Entra ID** 或 **OpenID Connect**（根据配置选择）[reference:43]
3. 系统将重定向到 Microsoft 登录页面，输入 Entra ID 用户的凭证
4. 认证成功后返回客户端，可访问已分配的服务器数据库

> 在客户端登录 OIDC 时，若报错 AADSTS50058 请尝试删除 Password Depot 中保存的 OIDC 配置，**重新创建**一个新的 OIDC 连接，并确保所有授权凭据已刷新[reference:44]。

---

## 九、常见问题排查

### 9.1 AADSTS50058：静默登录请求失败

**错误信息**：`AADSTS50058: A silent sign-in request was sent but no user is signed in.`

**原因**：浏览器中未检测到有效的用户会话 Cookie，通常是因为用户未登录、Cookie 被清除，或使用了不同浏览器的隐私模式。Password Depot 向导尝试在后台静默获取令牌时失败[reference:45]。

**解决方案**：
- 在 Password Depot 导入向导中删除当前 Entra ID 配置，**重新创建**一个新的连接，强制进行一次全新、交互式的登录（非静默），让浏览器弹出登录窗口，完成交互式登录
- 使用 **无痕/隐私模式 (InPrivate / Incognito Mode)** 浏览器重新执行导入向导
- 清除浏览器缓存，或切换至其他浏览器[reference:46]
- 确保已在 `portal.azure.com` 交互式登录过一次，完成 OAuth 授权该应用访问您的租户数据


### 9.2 AADSTS700054：response_type 'id_token' 未启用

**错误信息**：`AADSTS700054: response_type 'id_token' is not enabled for the application.`

**原因**：Password Depot 应用尝试使用隐式授权流（Implicit Flow）请求 `id_token`，但应用程序未启用该功能[reference:47]。

**解决方案**：
- 在 Entra ID 应用注册的 **身份验证 (Authentication)** 页面，找到 **隐式授权和混合流 (Implicit grant and hybrid flows)** 部分，勾选 **ID 令牌 (ID tokens)** 复选框[reference:48][reference:49]
- 如果复选框不可见，通过编辑 **清单 (Manifest)**，将 `oauth2AllowIdTokenImplicitFlow` 属性设置为 `true`


### 9.3 403 Forbidden：导入用户失败

**错误信息**：导入用户时报 `403 Forbidden`。

**原因**：Password Depot 应用程序缺少必要的 Microsoft Graph API 权限，或权限已添加但未获得管理员同意（Admin Consent）[reference:50]。

**解决方案**：
- 在 Entra ID 应用注册的 **API 权限 (API permissions)** 页面，点击 **添加权限 (Add a permission)** → **Microsoft Graph** → **应用程序权限 (Application permissions)**
- 添加以下权限：`User.Read.All`（读取所有用户的基本信息）、`Group.Read.All`（读取组信息，如有需要），以及在授权页面启用 `openid`、`profile`、`email`[reference:51]
- 点击 **授予管理员同意 (Grant admin consent)** 按钮，确保状态从“未授予”变为“已授予”


### 9.4 重定向 URI 不匹配

**错误信息**：登录时提示“重定向 URI 与注册的 URI 不匹配”。

**原因**：Password Depot 中配置的重定向 URI 与 Entra ID 应用注册中配置的 URI 不一致[reference:52]。

**解决方案**：
- 在 Password Depot OIDC 配置中找到 **重定向 URL (Redirect URL)** 字段，复制完整地址
- 在 Entra ID 应用注册的 **身份验证 (Authentication)** 页面，找到 **Web** 平台的重定向 URI 列表，添加该地址并保存
- 格式通常为 `https://{PD Server 地址}:25019/oidc/callback`


### 9.5 MFA 导致导入阻塞

**在 Entra ID 导入向导中**，管理员登录时必须完成第二步验证，这是 Microsoft 安全政策的强制要求，无法绕过。因此在导入前请确保：
- 管理员账户已注册并配置了验证方法（如手机 Authenticator 应用验证或短信验证）
- 在开始导入流程前，在 Entra ID 门户中提前交互式登录一次，完成任何额外的验证提示

### 9.6 导入后用户无法登录

**原因**：通常由 SPN 未注册、Kerberos/NTLM 配置冲突或时间不同步导致。

**排查步骤**：
- 确认用户已在 Server Manager 中正确导入，且 **Account** 选项卡中的认证方式已设置为 **Entra ID** 或 **OpenID Connect**（视导入方式而定）
- 与 Active Directory 集成相比，Entra ID 登录无需 SPN 注册，但需要确保在 `Server Settings → Connections → Supported Authentications` 中已勾选了相关的 Entra ID 及 OIDC 认证方法[reference:53]
- 检查客户端与服务器之间的时间偏差（理想情况下小于 5 分钟，时间偏差过大会影响 OIDC / SAML 颁发的时效性令牌验证）
- 在隐式或独立 OIDC 集成场景中，确认应用注册中的端点配置（Discovery endpoint、Redirect URL）与 Password Depot 中的配置完全一致[reference:54]
- 检查服务器和客户端之间的网络可达性，包括 25019 端口的连通性
- 测试不同的用户名格式，如 `username@tenant.onmicrosoft.com`、仅用户名等，尤其针对 OIDC 流程时，保留 UPN 格式的用户主体名称，尽量确保命名规范一致

---

## 十、环境清理与注意事项

- **自动同步**：Entra ID 的自动同步仅同步属性变更，不会自动删除已在 Entra ID 中删除的用户，请在手动导入时启用“检查已删除对象”进行处理[reference:55]
- **许可证限制**：免费 Entra ID 租户默认最多可创建 50,000 个目录对象，由于 Password Depot 自身的 Enterprise Server 许可证用户数量（如 20 个用户）也可能成为限制因素，导入前请规划好需要同步的用户数量，避免超出 Password Depot 许可证容限[reference:56]）
- **多租户管理**：测试完成后如需删除租户，可在 Entra ID 管理页面操作
- **浏览器安全性**：建议使用最新版 Chrome 或新版 Edge 进行配置，避免使用 IE 或旧版浏览器
- **API 权限范围**：Password Depot 集成 Entra ID 需要的是 **应用程序权限**（Application Permission），而不是委托权限（Delegated Permission）。务必在 API 权限页面中启用 **应用程序权限**，并完成 **授予管理员同意**[reference:57]
- **新建用户 vs. AD 同步**：新建测试用户时，请不要忘记同时为他们在 Entra ID 中设置 Email 字段——OIDC 的某些流程需要 email claim 来完成用户属性映射[reference:58]

---

**文档版本**：1.0  
**最后更新**：2026-05-11  
**适用版本**：Password Depot Enterprise Server 19.1.0，Microsoft Entra ID Free tier
