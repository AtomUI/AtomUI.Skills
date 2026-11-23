# AtomUI Skills

[English](README.en.md)

面向 AtomUI 生态开发的 AI 编程助手技能库。提供 Git 提交消息起草、下游项目的 AtomUI 依赖升级与 API 迁移等工作流。

技能遵循 Agent Skills 开放规范，支持 Claude Code、Codex 和 Cursor。 [Agent Skills 规范](https://agentskills.io/specification)

## 快速开始

在要使用技能的项目目录运行命令。需要 Node.js 和 `npx`。

**查看可用技能**

```bash
npx skills add AtomUI/AtomUI.Skills --list
```

**交互式安装**

```bash
npx skills add AtomUI/AtomUI.Skills
```

**按名称安装单个技能**

```bash
npx skills add AtomUI/AtomUI.Skills --skill git-commit
```

`--skill` 使用下表中的技能名，对应 `SKILL.md` 的 `name` 字段。

## 技能一览

<!-- 技能列表由 catalog.json、SKILL.md 和 skill.json 生成。 -->

| 技能 | 用途 | 分类 | 版本 |
| --- | --- | --- | --- |
| [`git-commit`](skills/Development/git-commit/SKILL.md) | 检查 Git 暂存改动，按仓库约定起草提交消息，并在用户明确授权后创建和核验本地提交。 | `Development` | 1.0.0 |
| [`upgrade-atomui-package-references`](skills/Development/upgrade-atomui-package-references/SKILL.md) | 仅用于 AtomUI 框架源码仓库以外的下游项目，覆盖「请升级 AtomUI 到最新版本」及相关 NuGet 引用升级和 API Break 迁移。 | `Development` | 1.0.0 |

## 贡献指南

1. 选用 atomui.net 已启用的顶层分类，在 `skills/<category>/<skill-key>/` 创建技能目录。 [技能模板](template/AtomUIOSS/)
2. 编写 `SKILL.md` 和 `skill.json`，使目录名、技能 Key 和 `name` 一致。 [技能编写规范](docs/standards/skill-authoring.md) · [元数据规范](docs/standards/skill-metadata.md)
3. 将技能登记到对应分类，并核对登记路径和实际目录。 [目录清单](catalog.json) · [工程规范](docs/standards/skillshub-engineering.md)

## 许可证

[MIT License](LICENSE)
