# WinUHid (UMDF 2.23 兼容版)

本仓库是 [cgutman/WinUHid](https://github.com/cgutman/WinUHid) 的 fork，在原版基础上把 UMDF 驱动重新编译为 **UMDF 2.23**，让老版本 Windows 10 也能加载，并补齐了 **GitHub Actions 在线编译 + 自签名 + release 发布** 的完整链路。

## 为什么需要这个 fork

- 上游 `cgutman/WinUHid` 官方**不发布二进制**，且默认用最新的 UMDF（2.33）。
- UMDF 2.33 编译的驱动只能在 **Windows 11 21H2 及以上** 加载；在 Windows 10 上会因 `UmdfLibraryVersion` 高于系统 UMDF runtime 而报 `0xC0000365`（Event 219）装不上。
- 本 fork 把驱动改编译为 **UMDF 2.23**，可在 **Windows 10 1709 及以上**（含 19045 / 22H2）加载。

## 与上游的差异

| 项 | 上游 `cgutman/WinUHid` | 本 fork |
|---|---|---|
| UMDF 版本 | 随 WDK（默认 2.33）| 固定 **2.23** |
| 二进制发布 | 无 | release 发布 `WinUHid_Win10_2.23.zip` |
| 编译 | 需本地装 WDK | GitHub Actions + NuGet WDK 在线编译 |
| 签名 | 无 | 自签证书（20 年，带 RFC3161 时间戳）|

## 下载使用

1. 从 [Releases](https://github.com/8099bb4X21/WinUHid/releases) 下载 `WinUHid_Win10_2.23.zip`。
2. 解压到任意目录（路径不要含中文）。
3. 右键 `Run-Install.cmd` → **以管理员身份运行**。
4. 看到 `Result: OK` 即完成（驱动 + 证书 + 用户态库已部署）。
5. 双击 `Run-Status.cmd` 确认 `device reachable`。

**证书指纹（用于核对包的身份）**：`9946D2514F32DC2A4115C280B8DE1A14ABBA72D7`
（可用 `certutil -dump WinUHidPublisher.cer` 核对。）

## 如何自编译（无需本地装 WDK）

1. fork 本仓库。
2. 打开 **Actions** → **Build WinUHid (UMDF 2.23 compat)** → **Run workflow**（选分支 `main`）。
3. 跑完在 **Artifacts** 里下载 `WinUHid_Win10_2.23.zip`。

编译全部在 GitHub 官方 runner + NuGet 官方 WDK 包（`Microsoft.Windows.WDK.x64`）上完成，全程可审计。

## 如何自签名

1. 在你本机生成一张 20 年代码签名证书（**私钥只留在你这台机器**）：

   ```powershell
   $cert = New-SelfSignedCertificate -Type CodeSigningCert `
     -Subject "CN=你的名字 WinUHid" `
     -NotAfter (Get-Date).AddYears(20) `
     -CertStoreLocation Cert:\CurrentUser\My `
     -KeyExportPolicy Exportable -KeyAlgorithm RSA -KeyLength 2048

   $pwd = ConvertTo-SecureString -String "强密码" -Force -AsPlainText
   Export-PfxCertificate -Cert $cert -FilePath "$env:USERPROFILE\winuhid.pfx" -Password $pwd

   # 输出 base64（复制完整结果）
   [Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\winuhid.pfx"))
   ```

2. 在 fork 里配两个 Actions Secrets：
   - `WINUHID_PFX_BASE64` ← 上一步输出的 base64
   - `WINUHID_PFX_PASSWORD` ← 密码原文

3. 触发 workflow，产出的 zip 就是用**你自己的证书**签名的驱动。

## License

MIT（同上游 `cgutman/WinUHid`）。驱动的说明见 `packaging/安装说明.txt`。