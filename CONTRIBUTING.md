# Contributing

欢迎向 `ohos-porting-skills` 贡献。本文件同时说明中文和英文的贡献方式。

Contributions to `ohos-porting-skills` are welcome. This file describes how to contribute in both Chinese and English.

---

## 中文

本仓库的核心是一份持续更新的通用 Skill，用来把其他平台的应用移植或复刻到 OpenHarmony。贡献也应服务这个目标，而不是把仓库变成某一个具体项目的专题文档。

### 可以贡献什么

- 对 `skills/port-to-openharmony/SKILL.md` 中过时、含糊或缺失的通用步骤的修正；
- 在多类移植中反复出现、但尚未写入手册的判断或检查项；
- `checklists/` 与 `templates/` 的补全。

请把可复用的方法写进通用 Skill 或检查清单。不要把某一次移植的完整命令序列塞进 Skill，否则编码助手会把它当成必须逐步执行的脚本。

### 不接受的内容

- 密钥、证书、签名材料、令牌、口令，以及任何可以用来代表他人签名或登录的文件；
- 仅在某台机器上成立的绝对路径、内网地址、设备序列号；
- 未公开的厂商细节、未授权的内部文档摘录；
- 把本仓库变成某个私人工作流、定时任务，或某个具体上游项目的作品集。

### 建议的方式

1. Fork 本仓库并创建分支，或先提交 Issue 讨论后再发 Pull Request。
2. 修改 Skill 时保持 Cursor 兼容的 `SKILL.md` 格式：YAML frontmatter 中包含 `name` 与 `description`（`description` 要说明何时使用），正文写通用步骤。
3. 修改面向使用者的说明时，优先同时更新中文和英文；若暂时只能更新一种语言，请在 PR 里注明。
4. 提交前检查 diff：确认没有密钥、签名文件、本地路径和设备日志。

---

## English

The core of this repository is a continuously updated general skill for porting or replicating applications from other platforms onto OpenHarmony. Contributions should serve that goal, rather than turning the repository into a special-topic document for one project.

### What to contribute

- Corrections or missing general steps in `skills/port-to-openharmony/SKILL.md`;
- Judgments or checks that have shown up across more than one kind of port but are not yet in the playbook;
- Improvements to `checklists/` and `templates/`.

Put reusable methods in the general skill or the checklists. Do not paste a full command sequence from one port into the skill; coding assistants will otherwise treat it as a script they must follow step by step.

### What not to include

- Secrets, certificates, signing material, tokens, passwords, or any file that could be used to sign or log in as someone else;
- Absolute paths that only work on one machine, private network addresses, or device serial numbers;
- Unpublished vendor details, or excerpts from internal documents you are not authorized to share;
- Content that turns this repository into a private workflow, a cron job, or a portfolio for one specific upstream project.

### Suggested process

1. Fork the repository and open a branch, or file an issue first and then send a pull request.
2. Keep skills in the Cursor-compatible `SKILL.md` format: YAML frontmatter with `name` and `description` (the description must say when to use the skill), followed by generic steps.
3. When you change user-facing text, update both Chinese and English if you can. If you can only update one language for now, say so in the pull request.
4. Before you commit, inspect your diff for secrets, signing files, local paths, and device logs.
