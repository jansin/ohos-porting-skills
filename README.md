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

`ohos-porting-skills` 提供一份给人和编码助手共同使用的通用操作手册。它要回答的问题很具体：当你手上已有 Android、Linux、Windows、Web 或其他平台上的应用，希望把它移植或复刻到 OpenHarmony 时，应该按什么顺序做、每一步的通过标准是什么。

仓库的核心是一份通用 Skill：`skills/port-to-openharmony/SKILL.md`。请按其中的流水线顺序执行，不要跳步。它不是某个账号上的定时工作流，也不是一份项目作品集或案例陈列。

### 为什么做这个仓库

把已有应用做到 OpenHarmony 上，真正容易卡住的往往不是某一个 API 的名字，而是顺序不对：壳还没有，就开始编 so；HAP 还不能装，就开始做手势；交互还不通，就开始处理文件目录；每个设备再单独翻一套工程。

作者在 OpenHarmony / 鸿蒙上做过多种类型的移植实践。做这个仓库，是为了把这些经验收成一份可以持续修订的通用流水线，让后来的人和编码助手按同一条主线工作。

### 目的

本仓库的目的，是让把其他平台的应用移植或复刻到 OpenHarmony 变得更简单、更可重复。先准备壳，再按开源与否和语言选路径，用 HAP 在设备上验证，然后再做交互、文件访问和多端部署。

### 推荐流程

按下面的顺序执行，前一步没有通过，就不要进入下一步。详细的入口条件、产出物和通过标准见 `skills/port-to-openharmony/SKILL.md`。

1. **先判断 OpenHarmony 壳子是否已经创建成功。** 壳子指 DevEco / HAP 工程骨架。如果已经创建成功，就不再创建，直接复用；如果还没有，就先创建壳。
2. **再判断要移植或复刻的项目是开源还是不是开源。**
   - **如果不是开源的：** 通过 CraftAnything 分析当前软件的界面与行为，然后用其他语言在 OpenHarmony 上复刻实现。
   - **如果是开源的：** 查看当前代码是用什么语言写的。
     - 若是 TypeScript，并且与 OpenHarmony / ArkTS 兼容，则按照 TS 移植到 OpenHarmony 的方案，把代码迁进壳工程。
     - 若是 C、C++、Rust 等语言，则通过 OpenHarmony 的官方编译工具链交叉编译成 so，再集成到壳工程中。
3. **编译完毕、集成完毕后做初步验证。** 原生路径通过 Native 创建 NAPI 对接 so，UI 用 XComponent 渲染；然后编译成 HAP，在设备上验证。必须 HAP 能装能跑，才能进入下一步。
4. **初步验证成功后，对接触控、鼠标等交互方式。**
5. **交互对接完毕后，再对接保存、选择等能力，并明确走应用沙箱还是公共目录。**
6. **最后完成一次开发、多端部署。** 同一套工程适配手机、平板等不同设备，而不是为每个端再从头移植一次。

### 如何使用

**1. 获取仓库**

```bash
git clone https://github.com/jansin/ohos-porting-skills.git
```

**2. 给编码助手使用**

以 Cursor 为例，把 `skills/port-to-openharmony/` 复制到项目或个人的 skills 目录，或在对话里直接引用这份 `SKILL.md`。请按 Skill 中的流水线顺序工作，不要跳过壳检查、HAP 验证或交互阶段。

**3. 给人阅读使用**

先读本页的流程概述，再按 `skills/port-to-openharmony/SKILL.md` 逐步进行，并对照 `checklists/` 中的检查项。

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

本仓库标明「持续更新中」。通用流水线、检查清单和说明会随实践修订。使用时请以你面对的源项目版本、目标系统和 SDK 为准，把这里的内容当作起点。

欢迎在不泄露密钥、内部路径和设备信息的前提下提出修正。详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

### 许可

本仓库采用 Apache License 2.0。你可以自由使用、修改和分发这些 Skill 与文档，包括用于商业项目，只需保留许可声明与版权信息。本仓库的许可只覆盖本仓库自己的内容；你所移植的上游项目仍遵循其原有许可，品牌与商标通常还要单独处理。

---

## English

### What this repository is

`ohos-porting-skills` is a general playbook for people and coding assistants. It answers a concrete question: when you already have an application on Android, Linux, Windows, the Web, or another platform, and you want to port or replicate it on OpenHarmony, in what order should you work, and what counts as passing each stage?

The core of the repository is one general skill: `skills/port-to-openharmony/SKILL.md`. Follow that pipeline in order, and do not skip steps. It is not a scheduled workflow bound to one account, and it is not a portfolio or a collection of case studies.

### Why this repository exists

The part that usually goes wrong is the order of work, not the name of one API: compiling a `.so` before a shell exists, adding gestures before the HAP can be installed, handling file directories before input works, or starting a separate port for every device.

The author has done several kinds of porting work on OpenHarmony / HarmonyOS. This repository turns that experience into a general pipeline that can be revised over time, so later developers and coding assistants can follow the same main line.

### Purpose

The purpose of this repository is to make it simpler and more repeatable to port or replicate applications from other platforms onto OpenHarmony. Prepare the shell first, choose a path by license and language, verify with a HAP on a device, and only then add input, file access, and multi-device deployment.

### Recommended pipeline

Follow the steps below in order. Do not start the next stage until the current one has passed. Entry conditions, deliverables, and pass criteria are in `skills/port-to-openharmony/SKILL.md`.

1. **First check whether an OpenHarmony shell already exists.** The shell is a DevEco / HAP project skeleton. If it has already been created successfully, do not create another one; reuse it. If it does not exist, create the shell first.
2. **Then decide whether the project to port or replicate is open-source or not.**
   - **If it is not open-source:** use CraftAnything to analyze the current software's UI and behavior, then reimplement it on OpenHarmony in a suitable language.
   - **If it is open-source:** inspect the source language.
     - If it is TypeScript and compatible with OpenHarmony / ArkTS, follow the TypeScript-to-OpenHarmony porting path and move the code into the shell.
     - If it is C, C++, Rust, or a similar language, cross-compile it with the official OpenHarmony toolchain into `.so` libraries, then integrate them into the shell.
3. **After the build and integration are done, do an initial verification.** On the native path, create NAPI bindings to the `.so` and render the UI with XComponent; then compile a HAP and verify it on a device. Only when the HAP installs and runs should you continue.
4. **After that initial verification succeeds, hook up touch, mouse, and other input.**
5. **After input works, hook up save, file picking, and the choice between the app sandbox and a public directory.**
6. **Finally, complete one development effort with multi-device deployment.** Adapt the same project to phones, tablets, and other form factors, instead of starting a new port for each device.

### How to use

**1. Clone the repository**

```bash
git clone https://github.com/jansin/ohos-porting-skills.git
```

**2. Use the skill with a coding assistant**

In Cursor, copy `skills/port-to-openharmony/` into a project or personal skills folder, or reference that `SKILL.md` in the conversation. Follow the pipeline in order, and do not skip the shell check, HAP verification, or the input stage.

**3. Use it as a human-readable manual**

Read the pipeline overview on this page first, then work through `skills/port-to-openharmony/SKILL.md` and the checklists under `checklists/`.

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

This repository is marked "work in progress, continuously updated". The pipeline, checklists, and explanations will be revised as practice continues. Always treat the material as a starting point, and verify it against the source version, target system, and SDK you actually have.

Corrections that do not leak secrets, internal paths, or device information are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

### License

This repository is licensed under the Apache License 2.0. You may use, modify, and redistribute these skills and documents, including in commercial projects, provided that you retain the license and copyright notices. The license covers only this repository's own content. Upstream projects you port remain under their original licenses, and brand or trademark questions usually need to be handled separately.
