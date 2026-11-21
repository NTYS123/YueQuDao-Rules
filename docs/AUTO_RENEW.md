# 阅趣岛规则自动续签（私钥只在本机）

## 运行方式

现有手机 APK 1.0.2193 已经内置 GitHub Raw URL 与 Ed25519 公钥，**续签同一套受支持的规则不需要重新打包 App**。

1. **Windows 私钥保持本地。** 不要将 `remote_rules_ed25519.pem` 上传到 GitHub、聊天、APK 或 GitHub Actions Secrets。
2. 在电脑上安装 [GitHub CLI](https://cli.github.com/)（命令行工具 `gh`），并完成一次 `gh auth login` 登录。
3. 从本次交付的补丁中将 `09_auto_renew_rules.py`、`09_install_rules_scheduler.ps1` 放到本地阅趣岛工程的 `scripts` 目录（`playback_rules_publish.py` 原本就在同一目录）。
4. PowerShell 执行：
   ```powershell
   powershell -NoProfile -ExecutionPolicy Bypass -File "D:\YueQuDao_20261121_Integrated\scripts\09_install_rules_scheduler.ps1"
   ```
5. 安装程序会先验证现有的签名规则和公私钥匹配，成功后创建**当前用户的每日中午和登录后**定时任务 `YueQuDao-SignedRules-Renew`。
6. 当当前**已验签的**远程规则剩余不超过 72 小时时，定时程序自动生成递增版本、在本地签名并用 `gh api` 更新 `main/signed.json`，随后重新下载核对。其余时间只检查，不反复提交。
7. 运行日志保存在 `%USERPROFILE%\.yuequdao\renew.log`。状态 `No renewal needed` 表示期限未到；`PUBLISHED` 表示续签成功。可以在任务计划程序里检查任务状态。

## 不能承诺的行为

- **电脑必须开机并登录、网络可用、GitHub CLI 登录仍有效**；电脑长期关闭时不能进行本地签名，也不会凭空出现云端签名。
- 本仓库的 GitHub Actions **只检查发布文件是否可读取、是否即将过期，并在异常时创建 Issue**。它不持有签名私钥，不会修改规则。
- 定时续签**只延长已签署规则的有效期**，绝不会自动发现新接口、猜测上游签名算法或绕过源站授权。上游改动需要经过适配、测试和人工确认后，才能另行签发新规则。
- 规则最长有效期 7 天。如果未及时续签，App 会继续尝试内置路由，不会因为远程规则到期而直接停止运行。
- 如密钥丢失或疑似泄露，应停止续签，重新生成可信密钥、更新 App 内公钥并重新发布安装包；不可直接把 GitHub 上的未签名 JSON 当成可信配置。

## 安全门槛

定时程序有如下保护：只认固定仓库/文件；先用本地公钥验证 GitHub 原规则；只续签已验证且符合白名单的三个既有接口；拒绝签名错误、公私钥不匹配、回退到旧版本的异常；GitHub 提交需要现有文件 SHA，防止覆盖并发更新；发布后再从 GitHub 拉回字节并验证一致性。首次安装前执行 dry run，不通过则不注册定时任务。

GitHub Actions 使用公开签名配置中提供的 **未验签时间元数据**用于提醒，不代表加密验证成功；实际规则信任由 App 和 Windows 本地签名程序负责。
