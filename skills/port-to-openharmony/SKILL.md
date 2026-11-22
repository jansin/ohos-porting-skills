---
name: port-to-openharmony
description: Use this skill when the user wants to port or replicate an existing application from another platform (such as Android, Linux, Windows, or the Web) onto OpenHarmony. Follow the pipeline in order: collect source and target-device information, reuse or create the HAP shell, choose an open-source or closed-source implementation path, integrate and verify with a signed HAP, then add input, file access, and multi-device deployment.
---

# 将应用移植或复刻到 OpenHarmony

本 Skill 给出一条可执行的通用流水线，用于把其他平台上的应用移植或复刻到 OpenHarmony。请按步骤顺序推进：前一步未达到进入标准，不要开始下一步。做法保持通用，不要绑定某一个上游项目，也不要在文档或脚本里写死某台开发机的绝对路径。

本仓库持续更新中。分发或公开前请对照 [checklists/pre-publication.md](../../checklists/pre-publication.md)。官方说明以 [OpenHarmony 开发者文档](https://docs.openharmony.cn/) 当前版本为准，并使用与目标设备匹配的 SDK。

---

## 流水线

1. 确认或创建壳工程。
2. 判断开源与否，并选择实现路径（闭源复刻，或按语言做 TypeScript / 原生移植）。
3. 完成集成，打出已签名的 HAP，在设备上做「能装能跑」的初步验证。
4. 对接触控、鼠标等输入。
5. 对接保存、选择，并确定沙箱或公共目录策略。
6. 在同一工程上完成一次开发、多端部署。

开始工作前，先向使用者确认下列信息。缺一项就先问清楚，不要用猜测代替。

## 开始前需要确认的信息

- **源项目：** 开源仓库或源码目录，或用于分析的闭源应用（安装包或可运行程序）。同时确认是否已有可复用的 HAP / DevEco 壳工程。
- **许可：** 若为开源，许可证是否允许计划中的修改与分发；品牌、图标和名称是否可沿用。
- **目标设备：** 系统是 OpenHarmony 还是 HarmonyOS、API 级别、设备形态（手机、平板等）和 ABI。这些会决定 `runtimeOS`、SDK 和签名方式。
- **构建环境：** 与目标匹配的 SDK、DevEco 或等价工具、签名所需的 Java 运行时，以及仅保存在本地的证书和 Profile。


---

## 第 1 步：确认或创建壳工程

**目的**

所有后续集成都落在同一个 DevEco / HAP 壳工程里，避免并行建两套工程把工作拆散。

**做法**

1. 先检查当前工作区是否已经有可用的壳工程：能否打开，是否包含可打包的 entry 模块，以及 `runtimeOS`、compileSdk / compatibleSdk、包名和 ABI 是否已经填写。
2. **若壳工程已经可用，直接复用，不要再建一套。** 只需核对它是否仍能编译，以及配置是否仍与目标设备一致。
3. 若还没有壳工程，再创建最小骨架：包名、`runtimeOS`、SDK 与 ABI、本地签名配置。签名证书、密钥和 Profile 只放在本机，不要写入仓库。
4. 打包和签名环境需要准备好官方 SDK、以及运行应用包签名工具所需的 Java 运行时。官方签名工具以 Java 或配套命令行方式调用，缺少 Java 时往往在出包阶段才暴露。

**检查项**

- `runtimeOS`、SDK 和 API 级别与目标设备一致。面向 HarmonyOS 的工程不能默认拿到 OpenHarmony 设备上安装，反之亦然。
- 签名类型与目标设备匹配。调试签名、厂商或企业设备管理策略、以及设备 API 低于包声明的情况，都可能导致安装失败。面向 OpenHarmony 设备分发时，应使用与该系统匹配的应用证书、已签名的 Provision Profile，并关注设备是否要求代码签名（codesign）。可参考官方 [应用包签名工具概述](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/hapsigntool-overview.md)：只有经过签名的应用才能在真机上安装、运行和调试；对无需跨级别申请 ACL 权限的应用，DevEco Studio 也提供自动化签名。
- 配置写在工程文件或可替换的本地配置中，而不是某台机器的绝对路径。

**常见失败与处理**

- 能编译但不能安装：优先核对 `runtimeOS`、API 级别、签名证书 / Profile 类型和包名冲突，而不是先改业务代码。
- SDK 过旧或 API 声明高于设备能力：更换与设备匹配的 SDK，或降低声明并回归验证。
- 签名工具报错或找不到 Java：先补齐工具链，再重新出包。

**进入下一步的标准**

壳工程已经确定（复用或新建完成），配置与目标系统一致，并且签名材料可在本地使用。这一步还不必做出完整功能。

---

## 第 2 步：判断开源与否，并选择实现路径

**目的**

先分清「移植已有源码」还是「按行为复刻」，再按技术栈选择实现方式，避免在错误前提上投入大量改造。

**做法**

1. 判断源项目是否开源，并阅读其许可证：是否允许修改、分发以及计划中的使用方式。若包含多种许可，按目录确认适用范围，并保留版权、LICENSE 和 NOTICE。
2. 把代码许可和品牌资源分开看。图标、启动图、字体和产品名称往往并不随代码一并授权。派生应用在未获得授权时，应使用自己的名称和图标。
3. 按下面的分支继续。

### 2.1 闭源项目：先分析行为，再在 OpenHarmony 上复刻

闭源软件没有可分发的源码，不能把原程序的实现拷进工程。本流程以 **CraftAnything** 作为分析入口：用它观察并记录目标软件的界面结构、交互路径和可感知行为，再选用合适的语言，在壳工程中重新实现。公开介绍见 [Craft Anything](https://craftanything.lol/) 与 [storytold/craft-anything](https://github.com/storytold/craft-anything)。

分析结论应能回答：主路径是什么、关键界面有哪些状态、输入和输出是什么。然后再选择实现语言：

- 适合用 ArkTS / TypeScript 表达的界面和业务，走第 2.2.1 节。
- 需要保留或重写为 C / C++ / Rust 等原生核心的能力，走第 2.2.2 节。

### 2.2 开源项目：先判断技术栈，再选择移植方式

#### 2.2.1 TypeScript / ArkTS 路径

先做兼容性判断，再迁移，不要假设「源码是 `.ts`」就可以直接运行：

- **语法：** ArkTS 以 TypeScript 为基础，但并不覆盖全部 TypeScript 特性。列出必须改写的语法。
- **依赖：** Node.js、浏览器 DOM、以及常见前端框架和普通 npm 包，都不能默认视为可用。逐个确认是否有 ArkTS 或 OpenHarmony 可替换实现。
- **UI 与系统能力：** 界面迁到 ArkUI；文件、网络、权限等改为 OpenHarmony API。

若判断为兼容，或只需有限改写即可兼容：把源码调整为可编译的 ArkTS，用 ArkUI 替换原 UI 框架，用系统能力替换原平台 API，并纳入壳工程编译。

若判断为不兼容：在壳工程中用 ArkTS 复刻主路径，或把必须保留的重计算改走原生路径，不要把原来的 TypeScript 运行时硬塞进 HAP。

#### 2.2.2 C / C++ / Rust 等原生路径

使用 OpenHarmony **官方编译工具链**进行交叉编译，生成可被 HAP 加载的 `.so`，再集成到第 1 步的壳工程中。官方 NDK 工程通常以 CMake 构建，并通过 `ohos.toolchain.cmake` 指定 `OHOS_PLATFORM`、`OHOS_ARCH`（例如 `arm64-v8a`）等参数，详见 [使用命令行 CMake 构建 NDK 工程 / NDK 工程构建概述](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/build-with-ndk-overview.md)。

特别注意：部分构建系统或语言生态仍可能把 OpenHarmony 目标识别为普通 Linux，从而拉取 GTK、fontconfig、winit 等桌面依赖。应显式切换到 ohos 目标，关闭不适用的桌面后端，并使用与 SDK 配套的 sysroot，而不是主机上的 GNU/Linux 或 Android 库。

集成时优先做**最小闭环**：先把一个最小的 NAPI 动态库（例如只导出初始化、处理、销毁）链入壳工程已有的 native 入口（如 `libentry`），确认可以加载，再逐步扩大接口。不要一开始就重写全部界面。

若开源项目既不是可兼容的 TypeScript，也不能直接编成 so，先判断能否改写到上述两条路径之一；不能时，按复刻方式在壳工程中实现主路径。

**检查项**

- 许可和品牌结论已经写明，派生应用的命名与图标不误用未授权资源。
- 闭源路径有界面与行为分析，而不是猜测实现细节。
- TypeScript 路径有兼容性结论；原生路径已使用官方工具链，且未误开桌面后端。
- 产物已经进入同一个壳工程。

**常见失败与处理**

- 许可证不允许计划中的分发方式：停止搬代码，先调整目标。
- 交叉编译突然出现 GTK / X11 一类依赖：回到目标检测和条件编译，确认 ohos 目标生效。
- so 能编出但壳工程打包时找不到：检查 ABI、库文件位置和模块打包配置。

**进入下一步的标准**

实现路径已经选定，相应代码或 so 已进入壳工程，并且可以开始做 HAP 级的集成与验证。

---

## 第 3 步：集成并对 HAP 做初步验证

**目的**

把上一阶段的实现接到可安装的应用包上，并在真机上证明「能装能跑」。这是后续交互和文件能力的前提。开发期用 `hdc` 推送 so 或做热替换可以加快迭代，但**不能替代**正式的签名安装验证。

**做法**

1. **原生路径：** 通过 NAPI 把 ArkTS 与 so 对接，需要画面或预览的部分用 XComponent 作为原生渲染表面。官方要求中包括：`napi_env` 与 JS 对象不可跨线程使用；耗时逻辑应放到异步任务里，并在完成回调中回到 JS 线程更新结果；一个 so 通常对应一个注册模块，且模块名与库名一致。参见 [Node-API 开发规范](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/napi-guidelines.md) 与 [自定义渲染（XComponent）](https://gitcode.com/openharmony/docs/blob/OpenHarmony-6.0-Release/zh-cn/application-dev/ui/napi-xcomponent-guidelines.md)。
2. **TypeScript 路径：** 以 ArkUI 渲染界面。若本阶段没有 so，不必为了形式补一条空的 NAPI / XComponent 链路。
3. 先验证一条最小主路径（启动后能看到核心界面，或原生表面已经出现），不要在这一步展开全部面板。
4. 使用与目标设备匹配的证书和 Profile 签名，编译 HAP，在目标设备上安装并启动。

**检查项**

- HAP 能安装、能冷启动，并且主路径的初步结果可见。
- 原生模块加载成功，没有因模块名、ABI 或缺失依赖库而失败。
- 耗时初始化没有放在 UI 线程上同步执行。
- 事件处理覆盖完整队列或完整回调序列，而不是只保留「最后一次」通知。轮询间隔过长、准备阶段阻塞界面、多实例或多套数据只处理了第一项、以及底层回传消息被丢弃，都可能表现为界面卡住或长时间空等。
- 涉及采集、编码或图像质量的阈值，不要为了缩短等待而随意放宽。
- 大图、多层缓存或较长的撤销栈，要按目标设备内存做上限评估。

**常见失败与处理**

- 能编过但安装失败：回到第 1 步的 `runtimeOS`、API 和签名检查。
- 能安装但立即退出：查看启动日志，优先检查 native 库加载、NAPI 注册和权限声明。
- 界面已起来但结果不更新：检查事件是否被覆盖或丢失，以及异步回调是否回到正确线程。
- 仅在推送 so 后可用、重新签名安装后失败：以签名安装的结果为准，补齐打包配置后再验证。

**进入下一步的标准**

已签名的 HAP 在目标设备上能够安装并启动，主路径的初步结果可见。未达到这一点，不要开始完整的输入对接。

---

## 第 4 步：对接触控、鼠标等输入

**目的**

在应用已经能装能跑之后，再把主路径所需的输入接完整。

**做法**

1. 将点击、滑动、滚轮和指针事件接到 ArkUI 或 XComponent 的输入回调中，并处理系统返回。
2. 先保证主路径上的单点触控和基本指针操作可用，再考虑多点触控；手写笔和压感作为按需增强，而不是这一步的前置条件。
3. 输入事件同样要完整处理，不要只取最后一次采样。若通过轮询获取状态，间隔必须足以跟上连续手势。
4. 在真机上验证前后台切换后，输入仍然可用。

**检查项**

- 主路径可以用触控完成基本操作。
- 目标形态包含鼠标或指针时，对应操作可用。
- 手势过程中没有明显丢点、卡死或误触。

**常见失败与处理**

- 鼠标可用但触控无效，或相反：检查事件来源是否只接了一种输入设备。
- 滑动一段后不再响应：检查是否有事件队列被清空、或原生表面失去焦点。

**进入下一步的标准**

主路径上的基本交互已经可用。输入仍不稳定时，不要开始做保存和目录策略。

---

## 第 5 步：对接保存、选择与目录策略

**目的**

在交互可用之后，再补齐文件的打开、选择和保存，并明确数据放在应用沙箱还是公共目录。

**做法**

1. 默认将应用文件放在应用沙箱中。只有产品确实需要访问用户公共目录时，再申请相应权限，并处理用户拒绝后的路径。
2. 打开或选择外部文件时，优先使用系统文件选择器，根据用户授权访问所选文件，而不是假设可以遍历整机目录。应用文件访问的官方说明见 [应用文件访问](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/file-management/app-file-access.md)。
3. 保存、另存为和选择的结果要能回到主路径（例如保存后能再次打开）。
4. 在说明里写清本阶段支持的文件来源、权限范围和失败时的提示。

**检查项**

- 能完成一次符合策略的选择或保存。
- 权限不足或用户取消时，有可理解的失败路径，而不会静默崩溃。
- 没有把调试用的宽权限或设备路径写进仓库。

**常见失败与处理**

- 选择器能打开但无法再次访问：确认是否需要持久化授权，以及 URI 的有效期。
- 在一台设备上可用、在另一台设备上被拒绝：检查权限模型、设备策略和是否误用了公共目录。

**进入下一步的标准**

主路径在输入和文件访问上已经打通，目录策略已经明确。

---

## 第 6 步：一次开发，多端部署

**目的**

在同一套壳工程和核心实现上，适配手机、平板等不同设备形态，而不是为每个端再做一次独立移植。OpenHarmony 的设计目标包含「一次开发，多端部署」：通过统一的用户程序框架、Ability 框架和 UI 框架，使同一应用在多终端上保持一致性，并支持在开发过程中预览不同终端的能力适配情况。说明见 [OpenHarmony 技术特性](https://docs.openharmony.cn/)。不同设备可裁剪的组件不同，可用 API 也可能不同，需要在目标设备上实测，而不是只看单一机型的结果。

**做法**

1. 核对这些设备的系统类型、API 级别、屏幕规格和输入形态（触控为主，或触控加指针）。
2. 在同一代码库中用布局、资源和必要的条件编译做适配，避免复制出互不相通的工程。
3. 在计划支持的每类设备上安装已签名的 HAP，回归启动、交互、保存或选择。
4. 对外分发或开源前，执行 [checklists/pre-publication.md](../../checklists/pre-publication.md)。

**检查项**

- 计划支持的设备形态都能安装并跑通已验证的主路径。
- 屏幕规格、输入方式和权限差异已记录，不默认「同一个包在所有设备上行为完全相同」。
- 仓库中没有密钥、证书、内部路径和设备标识。

**常见失败与处理**

- 只在开发用机上通过：补测计划中的其他形态，尤其是 API 级别或输入方式不同的设备。
- 平板可用、手机布局不可用：回到自适应布局和断点，而不是再拆一个工程。

**完成标准**

同一工程能够面向计划中的多类设备给出可安装的 HAP，主路径回归通过，并且公开内容已经过清理。

---

## 参考

- [OpenHarmony 开发者文档](https://docs.openharmony.cn/)
- [应用包签名工具概述](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/hapsigntool-overview.md)
- [NDK 工程构建概述](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/build-with-ndk-overview.md)
- [Node-API 开发规范](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/napi-guidelines.md)
- [自定义渲染（XComponent）](https://gitcode.com/openharmony/docs/blob/OpenHarmony-6.0-Release/zh-cn/application-dev/ui/napi-xcomponent-guidelines.md)
- [应用文件访问](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/file-management/app-file-access.md)
