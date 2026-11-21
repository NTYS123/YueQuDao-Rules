# 阅趣岛远程规则仓库配置

此仓库仅用于发布**公开、经 Ed25519 数字签名的红果 V5 接口规则**。不存放 APK、完整应用源码、Cookie、令牌或任何签名私钥。

## 重要边界
- `endpoints.example.json` 是参考模板，**不是**手机会自动应用的规则。
- 手机订阅的文件名为根目录 `signed.json`，必须使用本地私钥按阅趣岛 1.0.2193 的 `scripts/playback_rules_publish.py sign` 格式签名并发布。
- 应用需在**重新编译时**嵌入此仓库的 Raw URL 和对应公钥；旧 APK 不会因仓库创建就自动启用远程更新。
- 目前的客户端只允许预置的两个 `fqnovel.com` 主机和固定接口路径前缀，不能靠改配置处理任意新主机、新认证方式或新签名算法。
- 客户端大约每 6 小时拉取一次；配置要求 `expires_unix` 有效，不能依靠一次发布无限期生效。有效规则失效后应退回内置接口。
- 私钥请只存于所有者自己的电脑；不要上传本仓库、聊天、日志、APK 或 CI 构建产物。

## 一次性配置（在 Windows 电脑）
1. 将阅趣岛 1.0.2193 完整候选工程解压至 `D:\YueQuDao_20261121_Integrated`。
2. 使用该工程的 `scripts/08_prepare_remote_publisher.ps1`，`-RulesUrl` 设置为：
   `https://raw.githubusercontent.com/NTYS123/YueQuDao-Rules/main/signed.json`
3. 初始化工具将私钥保存在用户目录 `%USERPROFILE%\.yuequdao\private\remote_rules_ed25519.pem`，URL 和公钥写入 `%USERPROFILE%\.yuequdao\trusted_rules.json`。
4. 将 `endpoints.example.json` 作为已知接口参考；使用项目自带的 `playback_rules_publish.py sign` 生成 `signed.json` 并在本机先验证签名。
5. 将**只有签名信封**的 `signed.json` 上传到仓库根目录。然后运行原项目的 1.0.2193 严格构建/测试脚本，使 APK 嵌入可信 URL 和公钥，真机验证签名更新、失效回退。

## 自动化范围
自动故障检测和已知接口切换可在 APP 内完成；GitHub 负责托管**已经发布**的规则。GitHub 不会自己发现新上游协议，也不会替所有者持有本地私钥签名并发布。未来如增加 GitHub Actions，可先做只读检测、候选规则与告警，再由所有者验证并签名发布；切勿未经验证自动执行来源不明的脚本。
