# ohos-porting-skills

把其他平台的应用移植或复刻到 OpenHarmony 的通用 Skill。

A general skill for porting or replicating applications from other platforms onto OpenHarmony.

**持续更新中。** 本仓库会根据实践不断修订，当前内容是可用的起点，不是最终定稿。

**Work in progress, continuously updated.** This repository is revised as practice accumulates. What you see now is a usable starting point, not a finished specification.

本仓库由 [苗俊祥](https://github.com/jansin) 开源，许可为 [Apache License 2.0](LICENSE)。

This repository is maintained by [Miaojunxiang (苗俊祥)](https://github.com/jansin) and licensed under the [Apache License 2.0](LICENSE).

---

## 中文

### 这个仓库是什么

`ohos-porting-skills` 提供一份给人和编码助手共同使用的通用操作手册。它要回答的问题很具体：当你手上已有 Android、Linux、Windows、Web 或其他平台上的应用，希望把它移植或复刻到 OpenHarmony 时，应该按什么顺序判断、避开哪些反复出现的坑、如何把工作拆成可以在设备上验证的阶段。

仓库的核心是一份通用 Skill：`skills/port-to-openharmony/SKILL.md`。它把过往在 OpenHarmony / 鸿蒙上做多类移植时反复遇到的问题，抽象成一套不绑定某个具体上游项目的步骤。它不是某个账号上的定时工作流，也不是一份项目作品集或案例陈列。

### 为什么做这个仓库

把已有应用做到 OpenHarmony 上，门槛往往不在某一个 API 的名字，而在一条很长的链：先判断源项目值不值得搬、许可和品牌能不能用、原有架构如何映射到 ArkTS / NAPI / 原生层，再到构建系统、`runtimeOS` 与 SDK、签名安装、输入和权限、内存与性能，以及开源或分发前的清理。

这些步骤在不同项目里会反复出现，但资料通常散落在官方文档、社区讨论和各人自己的笔记里。后来的人很难直接复用已经验证过的判断顺序，也容易在同一类问题上重新卡住：例如目标系统与 SDK 声明不一致、交叉编译时把 OpenHarmony 当成普通 Linux、把桌面依赖原样拉进来，或是能编过却装不上、能装上却无法完成一条主路径。

作者在 OpenHarmony / 鸿蒙上做过多种类型的移植实践。做这个仓库，是为了把这些经验收成一份可以持续修订的通用手册，而不是把某一次具体项目的过程公开成专题文档。

### 目的

本仓库的目的，是让把其他平台的应用移植或复刻到 OpenHarmony 变得更简单、更可重复。

它希望帮助开发者和他们使用的编码助手：

1. 先有一条通用主线，而不是每次都从空白对话里重新发明流程；
2. 在动手改代码之前，先处理许可、目标设备和架构映射，避免做了很久才发现前提不成立；
3. 用可在设备上观察的阶段来验收，例如「能安装并启动」「能完成一条输入到输出的主路径」，而不是以「代码已经迁了一半」作为完成标志。

本仓库不宣称某个上游项目已经官方支持 OpenHarmony，也不把个人实践写成平台产品说明。

### 这份 Skill 解决什么问题

通用 Skill 覆盖一次移植里通常都要面对的问题：

- 评估源项目是否适合移植或复刻，以及本阶段的目标是什么；
- 处理许可、商标和品牌资源；
- 把原有架构映射到 ArkTS 应用壳、NAPI 和必要的原生层；
- 配置构建、`runtimeOS`、SDK 与 API 级别；
- 完成签名、打包和安装；
- 适配输入、权限和生命周期；
- 按移动设备约束处理性能与内存；
- 在设备上验证，并在公开前做清理。

当前版本先把这条主线写清楚。专项说明以后如果有必要再单独补充，不会作为现在的目录入口。

### 如何使用

**1. 获取仓库**

```bash
git clone https://github.com/jansin/ohos-porting-skills.git
```

**2. 给编码助手使用**

以 Cursor 为例，把 `skills/port-to-openharmony/` 复制到项目或个人的 skills 目录，或在对话里直接引用这份 `SKILL.md`。其他同样支持 `SKILL.md` 的助手也可以用这种方式加载。请按 Skill 中的顺序工作，不要跳过许可、目标设备和安装验证。

**3. 给人阅读使用**

可以把本仓库当作操作手册。建议先读 README，再按 `skills/port-to-openharmony/SKILL.md` 逐步进行，并对照 `checklists/` 中的检查项。

### 仓库结构

```text
ohos-porting-skills/
├── README.md
├── LICENSE                          # Apache-2.0
├── CONTRIBUTING.md
├── skills/port-to-openharmony/      # 通用移植 Skill（本仓库的核心）
├── checklists/                      # 开源或分发前的检查清单
└── templates/                       # 可选用的记录模板
```

### 持续更新

本仓库标明「持续更新中」。通用步骤、检查清单和说明会随实践修订。使用时请以你面对的源项目版本、目标系统和 SDK 为准，把这里的内容当作起点。

欢迎在不泄露密钥、内部路径和设备信息的前提下提出修正。详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

### 许可

本仓库采用 Apache License 2.0。你可以自由使用、修改和分发这些 Skill 与文档，包括用于商业项目，只需保留许可声明与版权信息。本仓库的许可只覆盖本仓库自己的内容；你所移植的上游项目仍遵循其原有许可，品牌与商标通常还要单独处理。

---

## English

### What this repository is

`ohos-porting-skills` is a general playbook for people and coding assistants. It answers a concrete question: when you already have an application on Android, Linux, Windows, the Web, or another platform, and you want to port or replicate it on OpenHarmony, in what order should you decide, which recurring pitfalls should you avoid, and how should you split the work into stages that can be verified on a device?

The core of the repository is one general skill: `skills/port-to-openharmony/SKILL.md`. It distills problems that have repeatedly appeared in OpenHarmony / HarmonyOS porting work into a procedure that is not tied to any one upstream project. It is not a scheduled workflow bound to one account, and it is not a portfolio or a collection of case studies.

### Why this repository exists

Bringing an existing application to OpenHarmony is usually hard because of a long chain, not because of a single API name. You have to decide whether the source project is worth moving, whether the license and branding can be used, how the original architecture maps onto ArkTS, NAPI, and native code, and then deal with the build system, `runtimeOS` and SDK, signing and installation, input and permissions, memory and performance, and cleanup before publication or distribution.

These steps reappear across projects, while the useful notes are often scattered across official documentation, community threads, and private notebooks. Later work then has to rediscover a decision order that has already been tested, and it is easy to get stuck on the same class of problems: a target OS that does not match the SDK declaration, a cross build that treats OpenHarmony as ordinary Linux, desktop dependencies pulled in unchanged, or a tree that builds but will not install, or installs but cannot complete a main path.

The author has done several kinds of porting work on OpenHarmony / HarmonyOS. This repository exists to turn that experience into a general playbook that can be revised over time, not to publish any one project as a special-topic document.

### Purpose

The purpose of this repository is to make it simpler and more repeatable to port or replicate applications from other platforms onto OpenHarmony.

It is meant to help developers, and the coding assistants they use, to:

1. Follow a common main line, instead of inventing the process again in every new conversation;
2. Settle licensing, the target device, and the architectural mapping before changing a large amount of code, so the work is not undone by a false premise;
3. Accept a stage only when it can be observed on a device, such as "it installs and starts" or "it can complete one input-to-output path", rather than treating "half of the code has been moved" as done.

This repository does not claim that any upstream project officially supports OpenHarmony, and it does not present personal practice as a platform product.

### What this skill covers

The general skill covers the problems a port usually has to face:

- Judging whether the source project should be ported or replicated, and what this stage should deliver;
- Handling licenses, trademarks, and brand assets;
- Mapping the original architecture onto an ArkTS application shell, NAPI, and a native layer where needed;
- Configuring the build, `runtimeOS`, SDK, and API level;
- Signing, packaging, and installing;
- Adapting input, permissions, and lifecycle;
- Treating performance and memory as mobile-device constraints;
- Verifying on a device, and cleaning up before going public.

This version focuses on that main line. More specialized notes, if they become necessary, may be added later; they are not directory entries today.

### How to use

**1. Clone the repository**

```bash
git clone https://github.com/jansin/ohos-porting-skills.git
```

**2. Use the skill with a coding assistant**

In Cursor, copy `skills/port-to-openharmony/` into a project or personal skills folder, or reference that `SKILL.md` in the conversation. Other assistants that understand `SKILL.md` can load it in the same way. Follow the skill in order, and do not skip licensing, the target device, or install verification.

**3. Use it as a human-readable manual**

Read this README first, then work through `skills/port-to-openharmony/SKILL.md` and the checklists under `checklists/`.

### Repository layout

```text
ohos-porting-skills/
├── README.md
├── LICENSE                          # Apache-2.0
├── CONTRIBUTING.md
├── skills/port-to-openharmony/      # The general porting skill (the core of this repository)
├── checklists/                      # Checklists before publication or distribution
└── templates/                       # Optional templates for notes
```

### Continuously updated

This repository is marked "work in progress, continuously updated". The general steps, checklists, and explanations will be revised as practice continues. Always treat the material as a starting point, and verify it against the source version, target system, and SDK you actually have.

Corrections that do not leak secrets, internal paths, or device information are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

### License

This repository is licensed under the Apache License 2.0. You may use, modify, and redistribute these skills and documents, including in commercial projects, provided that you retain the license and copyright notices. The license covers only this repository's own content. Upstream projects you port remain under their original licenses, and brand or trademark questions usually need to be handled separately.
