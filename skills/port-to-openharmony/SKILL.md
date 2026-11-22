---
name: port-to-openharmony
description: Use this skill when porting or replicating an existing application from another platform (such as Android, Linux, Windows, or the Web) to OpenHarmony. Follow this general procedure for licensing, architecture mapping, runtimeOS and signing, input and permissions, performance, on-device verification, and cleanup before publication.
---

# Port or replicate an app to OpenHarmony / 将应用移植或复刻到 OpenHarmony

这是本仓库的通用 Skill。把其他平台上的已有应用做到 OpenHarmony 时，先按这里的顺序工作。步骤是通用的，不要绑定某一个上游项目，也不要把某台开发机上的绝对路径写进命令。

This is the general skill in this repository. When you port or replicate an existing application onto OpenHarmony, follow this order. Keep the steps generic: do not bind them to one upstream project, and do not hard-code absolute paths from one developer machine.

配套材料：

- 公开或分发前的检查：[checklists/pre-publication.md](../../checklists/pre-publication.md)
- 可选的过程记录模板：[templates/case-study.md](../../templates/case-study.md)

---

## 1. 先确定本阶段要交付什么 / Decide what this stage must deliver

在改第一行适配代码之前，先写下目标、假设和本阶段明确不做的事。

Before changing any adaptation code, write down the goal, the assumptions, and what this stage will not do.

1. **源项目在哪个平台、以什么形态运行。** 例如 Android 应用、Linux 或 Windows 桌面程序、命令行工具、带原生引擎的应用，或 Web 应用。把构建系统、GUI 工具包、原生语言和关键依赖列出来。
2. **这次是移植、复刻，还是先做风险验证。** 移植保留尽可能多的原有实现；复刻则在 OpenHarmony 上重做体验，但可以对齐原有能力。若核心依赖还不确定能否替换，先把「能否链接、能否在设备上跑通一个入口」当作验证，而不是承诺完整产品。
3. **本阶段的可观察完成标准。** 例如：能在目标设备上安装并启动；能打开一种主要输入并得到对应输出；能完成一条用户主路径。不要用「代码已经迁了一半」当作完成。
4. **目标系统与 API 级别。** 先确定设备跑的是 OpenHarmony 还是 HarmonyOS，以及可用的 SDK 和 API 级别。不要假设两套运行时可以共用同一套打包配置。

如果许可、目标设备或核心依赖在这一步就无法成立，应停下来改目标，而不是继续搭工程。

If licensing, the target device, or a core dependency already rules the work out, stop and change the goal instead of scaffolding the project anyway.

---

## 2. 许可、品牌与出处 / License, branding, and attribution

1. 确认源项目许可证是否允许你计划中的修改、分发，以及是否包含闭源或商用。若仓库里有多种许可，按目录列出适用范围。
2. 保留上游要求的版权、许可文件和 NOTICE。
3. 把「代码可以开源或再分发」和「品牌资源可以使用」分开看。图标、启动图、字体、名义和商标往往并不随代码一并授权。派生应用应使用自己的名称和图标，除非已经获得相应授权。
4. 不要把一次社区或个人移植写成上游或平台的官方产品，也不要编造下载量、奖项或合作方。
5. 如果源项目没有可获得的完整源码，不要把它当作本移植的实现基础；那时更适合按「复刻」来规划，并避开受保护的品牌与专有资源。

公开仓库前，再走一遍 [checklists/pre-publication.md](../../checklists/pre-publication.md)。

---

## 3. 把原有架构映射到 OpenHarmony / Map the architecture onto OpenHarmony

按「改动面最小、又能在设备上验证主路径」来选，而不是先重写全部界面。

Prefer the smallest change that can still prove the main path on a device, rather than rewriting every screen first.

常见映射方式：

- **ArkTS / ArkUI 应用壳。** 启动、权限、生命周期、文件选择、设置和大部分界面，优先放在 HAP 的 ArkTS 层。
- **保留原生核心，经 NAPI 接入。** 若源项目已有 C / C++ / Rust 等引擎或算法库，尽量保留核心，用 NAPI 把初始化、输入、处理和导出等少量接口交给 ArkTS。先打通最小共享库，再扩大接口面。
- **替换平台后端，保留上层逻辑。** 若源项目已有图形、文件、窗口或输入的抽象层，为 OpenHarmony 增加后端，通常比从业务层重写更合适。
- **不要把桌面 GUI 栈原样搬过来。** GTK、X11、Qt 桌面后端、浏览器扩展式桌面 UI 等，在设备上往往没有对等环境。界面应落到 ArkUI；重计算和绘制留在原生层或系统能力中。
- **Web / WASM 套壳可以探路，但不要默认当作产品主线。** 性能、输入、文件系统和系统能力都容易受限，后面往往还是要回到原生或 ArkTS 路径。

选定策略后，画出三层：应用壳、桥接（NAPI 或其他官方接口）、原生或共享逻辑。同时写明本阶段不迁移的模块。

After choosing a strategy, sketch three layers: the application shell, the bridge (NAPI or another official interface), and the native or shared logic. Also write down the modules this stage will not migrate.

---

## 4. 构建、runtimeOS 与 API 级别 / Build, runtimeOS, and API level

1. 在应用和模块配置中明确 `runtimeOS`，并与真正要安装的设备一致。面向其中一侧的工程直接拿到另一侧设备上安装，常常会失败。
2. 核对 compileSdk、compatibleSdk / 目标 API 和 ABI（例如 arm64-v8a）是否与设备能力匹配。SDK 过旧或 API 声明过高，都会导致安装或启动失败。
3. 文档和脚本里用可配置的工作区、SDK 位置和设备标识，不要写死某台机器上的绝对路径。
4. **交叉编译时不要把 OpenHarmony 目标当成普通 Linux。** 构建系统若只识别 `unix` / `linux`，可能自动打开 GTK、X11、ALSA、systemd 等不适用的依赖。应为 OpenHarmony 增加明确的目标判断，并关闭桌面后端。
5. **工具链和 sysroot 必须成对。** 编译器、链接器、头文件和系统库应来自匹配的 OpenHarmony SDK，而不是主机上的 GNU/Linux 或 Android 库。
6. **第三方依赖要逐个审查。** 能换成系统能力或已验证的 OpenHarmony 库就换；不能换的再考虑交叉编译。禁止为了「先链接通过」而引入一整套其它平台的运行时。
7. 某些语言生态的目标三元组可能仍按 Linux 解析 `ohos`。遇到异常依赖时，先检查目标检测，再检查业务代码。

---

## 5. 签名、打包与安装 / Signing, packaging, and installation

1. 使用开发者自己的调试或发布证书，在本地完成签名。不要把证书、私钥或口令放进仓库。
2. 先在与配置匹配的设备上安装并确认能启动，再谈功能。安装失败时，优先检查 `runtimeOS`、API 级别、签名和包名冲突，而不是先改业务代码。
3. 说明安装和取日志的方法时，使用通用设备工具和可替换的设备标识，不要写入序列号。
4. 自动化脚本应从环境变量或本地未跟踪的配置文件读取 SDK、签名材料和设备标识。

---

## 6. 输入、权限与生命周期 / Input, permissions, and lifecycle

1. 按移动设备重新设计输入。触摸、多点、系统返回和前后台切换，通常不能从桌面鼠标和键盘逻辑直接搬过来。先保证单指点击、拖动和返回正确，再考虑手写笔压感或快捷键。
2. 权限按最小集申请，并处理用户拒绝后的路径。不要为了「以后可能用到」一次开齐相机、存储、位置、网络或后台任务。
3. 在真机上检查生命周期：冷启动、进入后台再回来、窗口或分屏变化、以及页面销毁后重建。从其它系统搬来的状态管理，常常在这里出问题。
4. 文件访问要符合应用沙箱。明确本阶段支持的文件来源（应用内、用户选择、只读示例），不要假设拥有整机文件系统。

---

## 7. 性能与内存 / Performance and memory

1. 按目标设备的内存和功耗来验收，而不是按桌面工作站的上限。为大图、长列表、媒体缓冲或撤销栈规定上限。
2. 页面隐藏或进入后台时，释放可以重建的资源；重新可见时再按需加载。
3. 原生库与 ArkTS 之间避免无必要的大块拷贝。在桥接层明确所有权：谁分配、谁释放、哪一侧持有长生命周期对象。
4. 若绘制路径不稳定，可以先用更保守的实现验证正确性，但要评估内存和帧率是否可接受，再决定是否继续作为产品路径。

---

## 8. 分期验证 / Verify in phases

推荐的最小分期如下。可以按源项目调整，但不要跳过「设备上的主路径」。

A suggested minimum sequence follows. Adjust it for the source project, but do not skip the on-device main path.

1. **前提成立：** 许可、目标设备、关键依赖和构建入口已经核对。
2. **最小可安装：** 一个能在目标设备上安装并启动的 HAP，哪怕功能还不完整。
3. **主路径闭环：** 在设备上完成一条输入到输出的路径（打开、处理或编辑、得到结果或保存）。
4. **交互可用：** 触摸、返回、生命周期、必要权限和失败提示可用。
5. **再扩展：** 其余界面、性能、可选输入或更多格式。

每一阶段都要有失败时的回退点：哪一层已经验证，出了问题先回到哪一步。

Each phase should have a fallback: which layer has already been verified, and where to return when something fails.

---

## 9. 开源或分发前的清理 / Cleanup before publication

公开、向他人分发或提交到远程仓库前，执行 [checklists/pre-publication.md](../../checklists/pre-publication.md)。至少确保：

- 没有密钥、证书和签名材料；
- 没有内部绝对路径、设备序列号和未公开的厂商细节；
- 许可与品牌处理已经写清楚；
- `runtimeOS` 和 API 假设与文档一致。

---

## 10. 记录与回馈本手册 / Record and improve this playbook

若这次移植有可公开、且不绑定内部环境的经验，可以用 [templates/case-study.md](../../templates/case-study.md) 记下判断和结果，重点写「为什么这样选」，而不是粘贴一整段只在某台机器上成立的命令。发现通用步骤有缺漏时，应直接修订本 Skill，保持中英文可用。
