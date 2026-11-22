# ohos-porting-skills

把其他平台的应用移植或复刻到 OpenHarmony 的通用 Skill。

A general skill for porting or replicating applications from other platforms onto OpenHarmony.

**持续更新中。** 步骤和说明会随实践与官方文档的更新而修订，请把它当作可执行的起点，并以当前目标设备和官方文档为准。

**Work in progress, continuously updated.** The pipeline and explanations will change as practice and official documentation evolve. Treat this repository as a working starting point, and verify it against your target device and the current official docs.

本仓库由 [苗俊祥](https://github.com/jansin) 开源，许可为 [Apache License 2.0](LICENSE)。

Maintained by [Miaojunxiang (苗俊祥)](https://github.com/jansin). Licensed under the [Apache License 2.0](LICENSE).

---

## 中文

### 这个仓库是什么

这是一份给人和编码助手共用的操作手册，用来把 Android、Linux、Windows、Web 或其他平台上的应用做到 OpenHarmony 上。核心文件是 `skills/port-to-openharmony/SKILL.md`。请按其中的流水线顺序执行；每一步都有目的、做法、检查项、常见失败和进入下一步的标准。

它不是某个账号上的定时任务，也不是一份项目作品集。

### 为什么需要这份手册

移植工作容易在顺序和前置条件上反复出错：壳工程还没有确认，就开始编原生库；应用还不能在设备上安装启动，就开始接完整手势；输入还不稳，就去处理文件目录；每个设备再单独复制一套工程。与此同时，`runtimeOS` 与 SDK 不匹配、签名类型不对、交叉编译把 OpenHarmony 当成 Linux，也会让「已经能编过」的结果在安装时失败。

这份 Skill 把这类反复出现的判断，收成一条可以持续修订的通用流程。

### 推荐流程

详细步骤见 `skills/port-to-openharmony/SKILL.md`。主干顺序如下。

1. **确认或创建壳工程。** 若已有可用的 DevEco / HAP 工程，直接复用，不要重复创建。新建时一并核对 `runtimeOS`、SDK / API 级别和签名类型，并准备好 Java 与官方签名工具。HarmonyOS 与 OpenHarmony 的打包假设不能混用。
2. **判断开源与否，再选择实现路径。** 先确认许可证是否允许计划中的使用；品牌资源未必随代码开源，派生应用在未授权时应更换名称和图标。
   - 闭源软件：用 CraftAnything 分析界面、交互和可感知行为，再在 OpenHarmony 上重新实现，而不是拷贝原程序内部实现。
   - 开源软件：TypeScript 若能与 ArkTS / ArkUI 兼容，则迁移界面和业务；C / C++ / Rust 等则用官方工具链交叉编译为 so 后集成。交叉编译时要显式指定 ohos 目标，避免被识别成 Linux 而误拉桌面依赖。优先把最小 NAPI 库链入现有 native 入口，先打通闭环，再扩展界面。
3. **完成集成并做 HAP 验证。** 原生路径用 NAPI 对接 so，需要画面时用 XComponent 渲染，再打出已签名的 HAP。只有在目标设备上能安装、能启动，才进入下一步。开发期推送 so 可以加速调试，但不能代替正式签名安装。
4. **对接触控、鼠标等输入。** 需要时再补手写笔或压感。完整处理事件，不要只保留最后一次回调。
5. **对接保存、选择，并明确走应用沙箱还是公共目录。** 默认使用沙箱，需要公共目录时再申请相应权限。
6. **一次开发，多端部署。** 在同一工程上适配手机、平板等不同屏幕、输入和权限差异，并在各目标形态上回归主路径。

许可、签名、事件完整性和大内存评估等关键检查，已经写进对应步骤，而不是单独另起一条并行流程。

### 如何使用

```bash
git clone https://github.com/jansin/ohos-porting-skills.git
```

编码助手可将 `skills/port-to-openharmony/` 复制到 skills 目录，或直接引用该 `SKILL.md`。阅读时建议先看本页流程，再按 Skill 逐步执行，并对照 `checklists/`。

### 仓库结构

```text
ohos-porting-skills/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── skills/port-to-openharmony/      # 通用移植 Skill
├── checklists/                      # 公开或分发前的检查清单
└── templates/                       # 可选记录模板
```

### 贡献与许可

欢迎修正过时说明和补上可复用的检查项。请不要提交密钥、证书、内部路径或设备信息，也不要把本仓库写成某一个项目的专题文档。详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

本仓库采用 Apache License 2.0，许可范围仅覆盖本仓库自己的内容。被移植的上游项目仍遵循其原有许可。

常用官方文档入口：

- [OpenHarmony 开发者文档](https://docs.openharmony.cn/)
- [应用包签名工具概述](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/hapsigntool-overview.md)
- [NDK 工程构建概述](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/build-with-ndk-overview.md)
- [Node-API 开发规范](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/napi-guidelines.md)
- [自定义渲染（XComponent）](https://gitcode.com/openharmony/docs/blob/OpenHarmony-6.0-Release/zh-cn/application-dev/ui/napi-xcomponent-guidelines.md)
- [应用文件访问](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/file-management/app-file-access.md)

---

## English

### What this repository is

This is a shared playbook for people and coding assistants who need to port or replicate an existing application — from Android, Linux, Windows, the Web, or elsewhere — onto OpenHarmony. The main document is `skills/port-to-openharmony/SKILL.md`. Follow that pipeline in order. Each stage states its purpose, how to do the work, what to check, common failures, and the bar for moving on.

It is not a scheduled workflow, and it is not a portfolio of past projects.

### Why this playbook exists

Ports often stall on sequencing and prerequisites. Native libraries get built before a shell project is settled. Full gesture handling starts before the package can be installed. File-access policy is designed before input is stable. A separate tree appears for every device. In parallel, a mismatched `runtimeOS` or SDK, the wrong signature type, or a cross build that treats OpenHarmony as ordinary Linux can make a successful compile fail at install time.

This skill collects those recurring checks into one pipeline that can be revised over time.

### Recommended pipeline

See `skills/port-to-openharmony/SKILL.md` for the executable detail. The backbone is:

1. **Confirm or create the shell project.** Reuse an existing DevEco / HAP project when it is already usable; do not create a second one. When creating or checking a shell, align `runtimeOS`, the SDK / API level, and the signature type, and make sure Java and the official signing tools are available. Packaging assumptions for HarmonyOS and OpenHarmony are not interchangeable.
2. **Decide whether the source is open, then choose an implementation path.** Confirm that the license allows the intended use. Brand assets are not always covered by the same license; a derived app should use its own name and icons unless it has permission.
   - Closed-source software: use CraftAnything to study the UI, interactions, and observable behavior, then reimplement on OpenHarmony. Do not copy internals from the original program.
   - Open-source software: if TypeScript is compatible with ArkTS / ArkUI, migrate the UI and business logic; if the core is C, C++, or Rust, cross-compile it with the official toolchain into `.so` libraries and integrate them. Set an explicit ohos target so the build is not mistaken for Linux and does not pull in desktop dependencies. Prefer linking a minimal NAPI library into the existing native entry point to prove a closed loop before expanding the UI.
3. **Integrate and verify with a HAP.** On the native path, wire NAPI to the `.so` and use XComponent when a rendering surface is needed, then produce a signed HAP. Continue only after it installs and starts on the target device. Pushing a `.so` during development can speed up debugging; it does not replace a signed installation.
4. **Hook up touch, mouse, and other input.** Add stylus or pressure support only when needed. Process the full event sequence; do not keep only the last callback.
5. **Add save and file picking, and decide between the app sandbox and a public directory.** Prefer the sandbox, and request public-storage permissions only when the product actually needs them.
6. **Develop once, deploy to multiple form factors.** Adapt the same project to phones, tablets, and other screen, input, and permission differences, and re-check the main path on each intended device.

Licensing, signing, event completeness, and large-memory limits are built into the relevant stages rather than treated as a separate track.

### How to use

```bash
git clone https://github.com/jansin/ohos-porting-skills.git
```

Coding assistants can copy `skills/port-to-openharmony/` into a skills directory, or reference that `SKILL.md` directly. For human readers, start with this overview, then follow the skill and the checklists.

### Repository layout

```text
ohos-porting-skills/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── skills/port-to-openharmony/      # General porting skill
├── checklists/                      # Pre-publication checklist
└── templates/                       # Optional note template
```

### Contributing and license

Corrections and reusable checks are welcome. Do not commit secrets, certificates, internal paths, or device identifiers, and do not turn this repository into a special-topic document for one project. See [CONTRIBUTING.md](CONTRIBUTING.md).

The repository is licensed under Apache License 2.0. That license covers only this repository. Upstream projects you port remain under their own licenses.

Official documentation:

- [OpenHarmony documentation](https://docs.openharmony.cn/)
- [Application package signing overview](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/hapsigntool-overview.md)
- [NDK project build overview](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/build-with-ndk-overview.md)
- [Node-API development guidelines](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/napi-guidelines.md)
- [Custom rendering (XComponent)](https://gitcode.com/openharmony/docs/blob/OpenHarmony-6.0-Release/zh-cn/application-dev/ui/napi-xcomponent-guidelines.md)
- [Application file access](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/file-management/app-file-access.md)
