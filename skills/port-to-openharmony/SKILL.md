---
name: port-to-openharmony
description: Use this skill when porting or replicating an application onto OpenHarmony. Follow this pipeline in order: check or create the HAP shell, decide open-source vs closed-source, take the TS or native path, verify with a HAP, then add input, file access, and multi-device deployment.
---

# 将应用移植或复刻到 OpenHarmony / Port or replicate an app to OpenHarmony

按下面的流水线**顺序执行，不要跳步**。前一步没有达到通过标准，就不要进入下一步。步骤是通用的，不要绑定某一个具体上游项目，也不要把某台开发机上的绝对路径写进命令或文档。

Follow this pipeline **in order. Do not skip a step.** Do not start the next stage until the current one has met its pass criteria. Keep the steps generic: do not bind them to one upstream project, and do not hard-code absolute paths from one developer machine.

本仓库持续更新中。配套检查清单：[checklists/pre-publication.md](../../checklists/pre-publication.md)。

This playbook is continuously updated. Companion checklist: [checklists/pre-publication.md](../../checklists/pre-publication.md).

---

## 流水线总览 / Pipeline overview

1. 判断 OpenHarmony 壳子是否已经创建成功；已创建则不再创建，未创建则先创建壳。
2. 判断要移植或复刻的项目是开源还是闭源，并按语言选择路径。
3. 编译并集成完毕后，用 NAPI 对接 so（原生路径），UI 用 XComponent 渲染，再编译成 HAP 做初步验证。
4. 初步验证成功后，对接触控、鼠标等交互方式。
5. 交互对接完毕后，再对接保存、选择，以及沙箱或公共目录等文件策略。
6. 最后完成一次开发、多端部署。

1. Check whether an OpenHarmony shell already exists; reuse it if it does, and create one only if it does not.
2. Decide whether the source is open-source or closed-source, then choose the language path.
3. After the build and integration are done, wire NAPI to the `.so` (native path), render the UI with XComponent, and package a HAP for initial verification.
4. After that verification succeeds, hook up touch, mouse, and other input.
5. After input works, hook up save, file picking, and the sandbox versus public-directory policy.
6. Finally, complete one codebase with multi-device deployment.

---

## 第 1 步：判断并准备 OpenHarmony 壳子 / Step 1. Check or create the OpenHarmony shell

**入口条件 / Entry：** 已确定要把一个已有应用移植或复刻到 OpenHarmony，并且已明确目标设备或 API 级别。

**要做的判断：** 当前是否已经有可用的 OpenHarmony 壳子，也就是 DevEco 工程或 HAP 工程骨架（含应用/模块配置、可打包的 entry、能够对应到目标系统的 `runtimeOS` 与 SDK）。

- **如果壳子已经创建成功：不再创建。** 复用现有工程，只核对它是否仍能编译、签名和安装。不要另起一套平行工程把工作拆散。
- **如果还没有壳子：先创建壳。** 用 DevEco 或等价方式建立一个最小 HAP 工程骨架，配置好包名、`runtimeOS`、compileSdk / compatibleSdk、目标 ABI，并确保签名材料只留在本地、不进仓库。

**产出物 / Deliverables：** 一个可打开的壳工程；一份简短记录，说明壳是复用的还是新建的，以及目标系统和 API 级别。

**通过标准 / Pass criteria：** 壳工程存在且配置与目标设备一致。这一步还不必做出完整功能；但后面所有集成都落在这个壳里，而不是另外再建一个互不相干的工程。

If a usable shell already exists, do not create another one. If it does not, create a minimal HAP/DevEco skeleton first, and keep signing material local.

---

## 第 2 步：判断开源与否，并按语言选择路径 / Step 2. Open-source or not, then choose the language path

**入口条件 / Entry：** 第 1 步的壳已经就绪（复用或新建完成）。

先判断当前要移植或复刻的项目是**开源**还是**不是开源**。这里同时确认许可是否允许计划中的使用方式，以及品牌、图标、名义是否可沿用；许可或商标不允许时，应先改目标，而不是继续搬代码。

First decide whether the project is **open-source** or **not**. Also confirm that the license allows the intended use, and that branding can be reused; if it cannot, change the goal before moving code.

### 2.1 如果不是开源的 / If it is not open-source

通过 **CraftAnything** 分析当前软件，然后用其他语言进行实现。

CraftAnything 是本流程指定的闭源分析入口。请用它分析闭源软件的界面与行为，再选用合适的语言，在 OpenHarmony 上复刻实现。不要把闭源程序的内部实现原样拷进本工程。公开文档目前主要说明它可以研究已有软件并辅助做出新的实现；本 Skill 不编造更多产品细节，以你实际使用的版本为准。公开入口见 [Craft Anything](https://craftanything.lol/) 与 [storytold/craft-anything](https://github.com/storytold/craft-anything)。

选定实现语言之后：

- 若决定用 TypeScript / ArkTS 在壳里复刻，进入下面的 **TS 移植路径**（2.2.1），把复刻代码落到第 1 步的壳中。
- 若决定用 C / C++ / Rust 等实现核心，进入 **原生路径**（2.2.2），用官方工具链编出 so，再集成到壳中。

**产出物 / Deliverables：** 对闭源软件界面与行为的分析结论；选定的实现语言；开始落到壳工程中的复刻代码或原生核心。

**通过标准 / Pass criteria：** 已经明确「这是复刻而不是搬源码」，实现语言已选定，并且后续工作有明确的 TS 或原生分支可走。

Use CraftAnything as the closed-source analysis entry: analyze the app's UI and behavior, then reimplement it on OpenHarmony in a suitable language. Do not copy the closed-source internals into the project.

### 2.2 如果是开源的 / If it is open-source

查看当前代码是用什么语言写的，再选路径。保留上游要求的版权、许可和 NOTICE。

Inspect the source language, then choose a path. Keep the copyright, license, and NOTICE files required by upstream.

#### 2.2.1 TS 且与 OpenHarmony 兼容：按 TS 移植方案进行 / TypeScript compatible with OpenHarmony

**兼容性判断（必须先做）：**

- **语法：** ArkTS 以 TypeScript 为基础，但并不支持全部 TS 特性。先确认源码能否迁到 ArkTS，或需要改写哪些语法。
- **依赖：** Node.js、浏览器 DOM、React / Vue 等运行时，以及普通 npm 包，都不能默认当作 OpenHarmony 可用。要逐个核对是否有 ArkTS / OpenHarmony 可替换实现。
- **UI 与系统 API：** 界面需要落到 ArkUI；文件、网络、权限等要换成 OpenHarmony 能力。只是「文件扩展名是 `.ts`」并不等于可以直接运行。

若判断为**兼容，或经过有限改写后可兼容**：按照 TS 移植到 OpenHarmony 的方案，把代码迁进第 1 步的壳工程：

1. 把源码调整为 ArkTS 可编译的写法，去掉不兼容语法。
2. 用 ArkUI 替换原来的 UI 框架。
3. 用 OpenHarmony 系统能力替换 Node / 浏览器 / 其它平台 API。
4. 在壳工程里完成模块划分和编译配置，准备打 HAP。

若判断为**不兼容**：不要强行把原 TS 工程塞进 HAP。应改为在壳里用 ArkTS 复刻关键能力，或把必须保留的重逻辑改走原生路径，而不是假设原有 TS 运行时可以原样启动。

**产出物 / Deliverables：** 兼容性结论；已迁入或已改写到壳工程中的 ArkTS 代码。

**通过标准 / Pass criteria：** 壳工程能够编译这些 TS / ArkTS 代码，且主界面或主模块已经落在该壳中，可以进入第 3 步的 HAP 验证。

If the TypeScript is compatible with ArkTS/ArkUI (or can be made compatible with limited changes), port it into the shell. If it is not, do not force the original TS runtime into the HAP.

#### 2.2.2 C / C++ / Rust 等：官方工具链编译成 so，再集成 / C, C++, Rust, and similar

通过 OpenHarmony 的**官方编译工具链**进行交叉编译，编译成 `.so`，然后再集成到第 1 步的壳工程中。

1. 使用与目标设备匹配的官方 SDK / 工具链和 sysroot，不要把 OpenHarmony 目标当成普通 Linux，以免误拉 GTK、X11 等桌面依赖。
2. 先得到能被 HAP 加载的共享库（一个或多个 `.so`）。
3. 把 so 集成到已有或新建的壳工程中（放到模块约定的 native 库位置，并写好打包配置）。

**产出物 / Deliverables：** 官方工具链编出的 so；已写入壳工程的集成配置。

**通过标准 / Pass criteria：** so 已由官方工具链生成，并且已经进入壳工程，可以进入第 3 步用 NAPI 和 XComponent 对接。

Cross-compile with the official OpenHarmony toolchain into `.so` files, then integrate them into the shell. Do not treat the OpenHarmony target as ordinary Linux.

若开源项目是其它语言（既不是可兼容的 TS，也不是 C / C++ / Rust 一类可直接编 so 的语言），先判断能否改写到 TS 路径或原生路径；不能时，按复刻方式在壳里重新实现，不要跳过第 3 步的 HAP 验证。

---

## 第 3 步：NAPI / XComponent 对接，并编译 HAP 做初步验证 / Step 3. Wire NAPI and XComponent, then verify with a HAP

**入口条件 / Entry：** 第 2 步的编译和集成已经完成。也就是说：TS 路径的代码已落入壳工程并可编译；或原生路径的 so 已编出并已集成到壳中。

按路径完成对接，然后**编译成 HAP 进行验证**：

- **原生路径：** 通过 Native 创建 NAPI，对接已集成的 so；UI 用 XComponent 去渲染。不要在这一步开始做完整的手势或文件管理，先让核心画面或主处理结果能在设备上出现。
- **TS 路径：** 界面用 ArkUI 渲染。若这一阶段没有 so，则不必强行补一条空的 NAPI / XComponent 链路；直接在壳里编译 HAP 做验证。

**产出物 / Deliverables：** 已对接的工程（原生路径含 NAPI 与 XComponent）；一份可安装的 HAP。

**通过标准 / Pass criteria：** HAP 能在目标设备上**安装并启动**，并且能看到主路径的初步结果（例如核心界面出现，或原生渲染表面已经出来）。**只有 HAP 能装能跑，才能进入第 4 步的交互对接。** 能编过但装不上、或能装上却立即退出，都不算通过。

For the native path, create NAPI bindings to the `.so` and render with XComponent, then package a HAP. For the TS path, build the HAP from the ArkTS/ArkUI shell. The pass criterion is the same: the HAP must install and run on the target device before input work starts.

---

## 第 4 步：对接触控、鼠标等交互方式 / Step 4. Hook up touch, mouse, and other input

**入口条件 / Entry：** 第 3 步已经初步验证成功，HAP 能装能跑。

现在才对接触控、鼠标等交互方式。把点击、滑动、滚轮、悬停或指针等事件接到 ArkUI 或 XComponent 的输入回调上，并处理系统返回。先保证主路径上的基本操作可用，再考虑多点触控或压感。

**产出物 / Deliverables：** 主路径上可用的触控和（如需要）鼠标交互。

**通过标准 / Pass criteria：** 在设备上可以用触控完成主路径中的基本操作；若目标形态包含鼠标或指针，对应操作同样可用。交互还不通时，不要进入文件保存和目录策略。

Only after the HAP installs and runs, hook up touch, mouse, and other input. Do not start file-access work until this input path works.

---

## 第 5 步：对接保存、选择与目录策略 / Step 5. Hook up save, picking, and directory policy

**入口条件 / Entry：** 第 4 步的交互方式已经对接完毕，主路径基本操作可用。

在交互可用之后，再对接保存、选择等能力，并明确文件走**应用沙箱**还是**公共目录**：

- 保存、另存为、打开与选择文件，应通过系统文件选择或应用可见的目录来完成，而不是假设拥有整机文件系统。
- 默认优先使用应用沙箱。只有产品确实需要访问公共目录时，再申请对应权限，并处理用户拒绝后的路径。
- 在文档中写清本阶段支持的文件来源和权限范围。

**产出物 / Deliverables：** 保存与选择流程；关于沙箱或公共目录的明确策略。

**通过标准 / Pass criteria：** 用户能在设备上完成一次符合策略的保存或选择，并且权限不足时有可理解的失败路径。

After input works, add save and file picking, and decide whether files live in the app sandbox or a public directory. Prefer the sandbox unless the product truly needs public storage, and request only the permissions that path requires.

---

## 第 6 步：一次开发，多端部署 / Step 6. One development effort, multi-device deployment

**入口条件 / Entry：** 第 5 步已完成，主路径在交互和文件访问上已经打通。

最后再完成一次开发、多端部署。也就是：仍维护同一套壳工程和核心实现，针对手机、平板等不同设备形态做适配与打包，而不是为每个端再从头做一套移植。

1. 核对不同设备的 `runtimeOS`、API 级别、屏幕与输入形态（触控为主或触控加指针）。
2. 在同一代码库里用布局、资源或条件编译适配多端，而不是复制出互不相通的工程。
3. 在计划支持的每类设备上安装 HAP，回归主路径：启动、交互、保存或选择。
4. 对外分发或开源前，执行 [checklists/pre-publication.md](../../checklists/pre-publication.md)，确保没有密钥、签名材料、内部路径和设备信息。

**产出物 / Deliverables：** 同一套工程下的多端适配说明；在目标设备类型上可安装的 HAP。

**通过标准 / Pass criteria：** 计划支持的设备形态都能安装并跑通已验证的主路径；多端差异有记录，而不是默认同一个包在所有设备上行为完全相同。

Finally, deploy from the same project to multiple device types (for example phone and tablet). Do not start a separate port for each form factor. Re-verify the main path on each intended device, and clean up before publication.

---

## 注意事项 / Notes

- 签名证书、私钥和口令只留在本地，不要提交到仓库。
- 交叉编译时使用官方工具链与配套 sysroot；遇到异常依赖，先检查是否把 `ohos` 误判成了 Linux。
- 本流水线可以持续修订，但不要把某一次具体项目的命令历史写进本 Skill。
