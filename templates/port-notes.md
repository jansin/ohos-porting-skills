# Optional port notes / 可选移植记录

若要把一次可公开的判断记录下来，可以复制本模板。不要编造未实际发生的步骤，也不要粘贴密钥、内部路径、设备序列号或完整设备日志。这份模板用于个人或团队留存过程，不是本仓库的案例集入口。

If you want to record a publicly shareable decision, copy this template. Do not invent steps that did not happen, and do not paste secrets, internal paths, device serial numbers, or full device logs. This template is for optional notes, not a case-study gallery in this repository.

## 标题 / Title

- 源项目类型（不必点名内部仓库） / Source project type (no need to name an internal repository):
- 是否开源及许可证 / Open-source or not, and license:
- 源平台 / Source platform:
- 目标系统、API 与设备形态 / Target OS, API level, and form factor:
- 是否复用了已有壳工程 / Reused an existing shell project:
- 作者与日期 / Author and date:

## 背景与目标 / Background and goal

说明本阶段要交付什么，例如：壳工程就绪、能安装并启动、或已打通一条主路径。

Say what this stage was supposed to deliver, for example: a usable shell, install-and-run, or one completed main path.

## 许可与品牌 / License and branding

- 上游许可 / Upstream license:
- 需要单独处理的商标或资源 / Trademarks or assets that needed a separate decision:

## 实现路径 / Implementation path

说明选择了哪条路（ArkTS / TypeScript 迁移、官方工具链编译 so 后经 NAPI / XComponent 集成，或闭源分析后复刻），以及为什么不走其他路。

Describe the path you chose (ArkTS / TypeScript migration, official-toolchain `.so` integration via NAPI / XComponent, or a closed-source reimplementation) and why other paths were not used.

## 关键步骤（可公开） / Publicly shareable steps

按壳工程、实现路径、HAP 验证、交互、文件与目录、多端部署的顺序写，重点写判断和结果，而不是把某台机器上的完整命令历史贴进来。

Write in pipeline order: shell, implementation path, HAP verification, input, file access, and multi-device deployment. Emphasize decisions and outcomes rather than a full command history from one machine.

1.
2.
3.

## 遇到的坑与处理 / Pitfalls and how they were handled

- 问题 / Problem:
- 表现 / Symptom:
- 处理 / Resolution:

## 结果与未完成项 / Results and follow-ups

- 已经验证的能力 / What was verified:
- 已知限制 / Known limits:
- 下一步 / Next steps:

## 参考 / References

只列出可公开访问的链接或文档。

List only publicly accessible links or documents.
