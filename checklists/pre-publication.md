# Pre-publication cleanup checklist / 开源前清理清单

在把移植结果或衍生仓库公开之前，逐项检查。本清单配合 `skills/port-to-openharmony/SKILL.md` 使用。

Use this checklist before you publish a port or a derived repository. It accompanies `skills/port-to-openharmony/SKILL.md`.

## 许可与品牌 / License and branding

- [ ] 已确认上游许可允许你计划中的修改、分发和（如适用）商业使用。
- [ ] 已保留上游要求的版权、许可和 NOTICE。
- [ ] 已区分「代码开源」和「品牌资源开源」；若商标、图标、启动图或名义未授权，已更换应用名和图标。
- [ ] 文档没有把个人移植宣传成上游或平台的官方产品。

- [ ] The upstream license allows the intended modification, distribution, and (if applicable) commercial use.
- [ ] Required copyright, license, and NOTICE files from upstream have been kept.
- [ ] "The code is open source" is not confused with "the brand assets are open source". The app name and icons have been changed if the trademark, logos, splash screens, or naming are not licensed.
- [ ] The documentation does not present a personal port as an official upstream or platform product.

## 密钥与签名 / Secrets and signing

- [ ] 没有提交私钥、证书、`.p12` / `.p7b` / `.jks` / `.keystore`、token、口令或 `.env`。
- [ ] 签名材料只留在本地安全位置；仓库里如需说明签名步骤，只写流程，不写口令和文件内容。
- [ ] CI 或脚本中的密钥改为从密钥库或环境注入，而不是写在文件里。

- [ ] No private keys, certificates, `.p12` / `.p7b` / `.jks` / `.keystore` files, tokens, passwords, or `.env` files are committed.
- [ ] Signing material stays in a local, safe place. If the repository must explain signing, it describes the procedure only, not passwords or file contents.
- [ ] Secrets used by CI or scripts are injected from a secret store or the environment, not hardcoded.

## 路径、设备与日志 / Paths, devices, and logs

- [ ] 文档和脚本使用可配置的路径或占位符，而不是某台开发机上的绝对路径。
- [ ] 没有写入设备序列号、内部主机名、内网地址或未公开的厂商参数。
- [ ] 没有提交 hilog、tombstone、完整抓日志文件或带有个人/设备标识的调试转储。

- [ ] Docs and scripts use configurable paths or placeholders, not absolute paths from one developer machine.
- [ ] Device serial numbers, internal host names, private network addresses, and unpublished vendor parameters are absent.
- [ ] Hilog dumps, tombstones, full log captures, and debug dumps that identify a person or a device are not committed.

## 工程配置与验证 / Project configuration and verification

- [ ] `runtimeOS`、compileSdk 和目标 API 与公开声称的目标设备一致，并在文档中写明假设。HarmonyOS 与 OpenHarmony 的打包配置没有混用。
- [ ] 已在声称支持的设备形态上，用匹配的签名完成过安装和主路径验证。
- [ ] 本地覆盖文件（如 `local.properties`、本机 SDK 路径）未被跟踪。
- [ ] 若仓库包含子模块或大型原生目录，已说明获取方式，而不是把无关的完整工程复制进来。

- [ ] `runtimeOS`, compileSdk, and the target API match the publicly stated target device, and the assumptions are documented. Packaging settings for HarmonyOS and OpenHarmony have not been mixed.
- [ ] A signed package has been installed and the main path has been verified on the device types you claim to support.
- [ ] Local override files (such as `local.properties` or machine-local SDK paths) are not tracked.
- [ ] If the repository uses submodules or a large native tree, the document explains how to obtain them, instead of copying an unrelated full project into the tree.
