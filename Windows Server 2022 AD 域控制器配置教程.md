# Windows Server 2022 AD 域控制器配置教程（用于 Password Depot 测试）

本教程将指导您在 Windows Server 2022 上安装并配置 Active Directory 域服务（AD DS），将其提升为域控制器，创建组织单位和测试用户，并与 Password Depot Enterprise Server 集成，为后续的 AD 用户导入、SSO 登录及权限测试奠定基础。

---

## 一、环境准备

| 项目 | 推荐配置 |
|------|----------|
| 虚拟化平台 | VMware Workstation / Hyper‑V / VirtualBox |
| 操作系统 | Windows Server 2022 Datacenter（带 GUI） |
| 服务器名称 | `PD-AD01`（示例，可自定义） |
| 静态 IP | `10.8.0.3/24`（示例，根据实际网络调整） |
| DNS 服务器 | 指向域控自身 IP（如 `10.8.0.3`） |
| 域 FQDN | `acedev.com`（示例） |
| NetBIOS 名称 | `ACEDEV` |
| 管理员密码 | 设置强密码并记录 |

> **强烈建议**：在虚拟机中搭建测试环境，并在配置完成后立即**创建快照**，以便后续进行破坏性测试（如 SSPI 切换、域重命名）后快速恢复。

---

## 二、安装 AD DS 与 DNS 角色（图形化）

1. 登录 Windows Server 2022，打开 **服务器管理器**。
2. 点击右上角 **管理** → **添加角色和功能**。
3. 选择 **基于角色或功能的安装**，点击下一步。
4. 选择目标服务器，点击下一步。
5. 在 **服务器角色** 列表中勾选：
   - `Active Directory 域服务`
   - `DNS 服务器`
6. 弹出“添加功能”对话框时，点击 **添加功能**。
7. 一直点击 **下一步**，最后点击 **安装**。

安装完成后无需重启，但需要提升为域控制器。

---

## 三、提升为域控制器（图形化）

安装完成后，服务器管理器右上角会出现黄色感叹号，点击 **“将此服务器提升为域控制器”**。

- **部署配置**：选择 **添加新林**，根域名输入 `acedev.com`
- **域控制器选项**：
  - 林功能级别：`Windows Server 2016` 或更高
  - 域功能级别：`Windows Server 2016` 或更高
  - 指定 **DSRM 密码**（目录服务还原模式密码，需记住）
- **DNS 选项**：直接下一步（如有委派警告，忽略）
- **其他选项**：NetBIOS 域名自动生成 `ACEDEV`
- **路径**：保留默认 `C:\Windows\NTDS` 和 `C:\Windows\SYSVOL`
- 单击 **下一步**，最后点击 **安装**（服务器将自动重启）

---

## 四、验证域控制器

重启后使用域管理员账号登录（例如 `ACEDEV\Administrator`）。

- 运行 `dnsmgmt.msc` 检查 DNS 区域，应包含 `_msdcs.acedev.com` 等子域。
- 运行 `dsa.msc` 打开 **Active Directory 用户和计算机**。
- 运行 `netdom query fsmo` 查看 FSMO 角色分配。
- 运行 `dcdiag` 检查域控制器健康状态。

---

## 五、创建组织单位（OU）与测试用户

### 5.1 创建 OU（推荐）

打开 **Active Directory 用户和计算机**，右键域根 `acedev.com` → **新建** → **组织单位**，输入名称（如 `TestUsers`），点击确定。

### 5.2 创建单个测试用户

在目标 OU（或 `Users` 容器）上右键 → **新建** → **用户**，填写：
- 姓名：`Test01`
- 用户登录名：`test01`
- 设置密码，并勾选 **密码永不过期** 或 **用户下次登录时须更改密码**（根据需要）。

### 5.3 批量创建测试用户（PowerShell）

以下脚本在 `CN=Users` 容器中创建 `test01` 至 `test30` 共 30 个用户（无需 OU 也可使用）。

```powershell
# 批量创建30个测试用户（从 test01 到 test30）
1..30 | ForEach-Object {
    $num = "{0:D2}" -f $_
    $name = "test$num"
    $password = ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force
    New-ADUser -Name $name `
               -SamAccountName $name `
               -UserPrincipalName "$name@acedev.com" `
               -Path "CN=Users,DC=acedev,DC=com" `
               -AccountPassword $password `
               -Enabled $true `
               -PasswordNeverExpires $true
    Write-Host "已创建用户: $name"
}```
若已创建 OU（如 `OU=TestUsers,DC=acedev,DC=com`），则将 `-Path` 替换为对应 OU 路径。
## 六、配置 DNS 与网络

- 确保所有需要加域的客户端（包括 Password Depot Server 所在机器）的 DNS 服务器指向域控 IP（如 `10.8.0.3`）。
- 在客户端上可使用 `nslookup acedev.com` 验证解析是否正确。

---

## 七、与 Password Depot Enterprise Server 集成

### 7.1 启用 AD 认证

1. 打开 **Password Depot Server Manager**。
2. 进入 `管理 → 服务器设置 → 连接`，在 **支持的认证方式** 中勾选：
   - `集成Windows SSO`
   - `Windows域凭据`
3. 点击确定保存。

### 7.2 导入 AD 用户

1. 在 Server Manager 中，点击 `工具 → 从Active Directory导入`。
2. 选择 **资源管理器模式** 或 **搜索模式**。
   - **资源管理器模式**：浏览 AD 树形结构，勾选需要导入的用户或组。
   - **搜索模式**：按条件筛选用户（如 `(objectCategory=person)`）。
3. 点击 **导入**。导入成功后，用户会出现在 `用户` 列表中，`身份验证` 列显示 `Active Directory`。

### 7.3 配置自动同步（可选）

进入 `管理 → 服务器设置 → Active Directory`，勾选 **“与AD自动运行同步”**，设置同步间隔（如 60 分钟），并选择对“在AD中找不到的用户”的操作（忽略、禁用或删除）。

---

## 八、常见问题与排查

### 8.1 AD 用户无法登录 Password Depot 客户端

**错误信息**：`Unable to log in to Password Depot Enterprise Server`

**排查步骤**：
1. 确认客户端 DNS 指向域控，能解析域 FQDN。
2. 确认在 Server Manager 中已启用 `集成Windows SSO` 和 `Windows域凭据`。
3. 确认客户端时间与域控时间差不超过 5 分钟（Kerberos 要求）。
4. 尝试使用不同的用户名格式：`test01`、`ACEDEV\test01`、`test01@acedev.com`。
5. 检查 SPN 注册（见 8.2）。

### 8.2 SPN 注册问题（Kerberos 认证失败）

如果使用集成 Windows 认证失败，常见原因是缺少 SPN。

在域控上以管理员身份运行命令提示符：

```cmd
setspn -A HTTP/pdserver.acedev.com:25019 ACEDEV\svc_pd```

### 8.3 导入用户超过许可证限制

导入时若用户数超过已安装许可证数量（如 20 个），系统仍会导入成功，但在分配角色时报错。这是已知问题（BUG‑SERVER‑03），建议在导入前控制用户数量或升级许可证。

### 8.4 无法加域（客户端）

- 检查客户端 DNS 是否指向域控。
- 检查防火墙是否开放 AD 相关端口（TCP/UDP 53、88、135、389、445、464 等）。
- 尝试使用 `ACEDEV\Administrator` 账号加域。

---

## 九、安全快照

完成域控配置并验证 Password Depot 集成后，**立即在虚拟机管理程序中创建快照**。后续任何破坏性测试（如修改 SSPI 模式、执行域重命名等）都可以随时恢复到此快照。

---

## 十、参考资料

- [Microsoft 官方：安装 Active Directory 域服务](https://learn.microsoft.com/zh-cn/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services)
- [Password Depot 在线帮助：Active Directory 集成](https://www.password-depot.com/en/documentation/onlinehelp-enterprise-server)

---

**文档版本**：1.0  
**最后更新**：2026-05-13  
**适用版本**：Windows Server 2022 / Password Depot Enterprise Server 19.1.0
