# AtomUI Skills

AtomUI 的知识型技能目录，遵循 [Agent Skills 开放标准](https://agentskills.io/specification)（YAML frontmatter + Markdown 正文），
可被 Claude Code、Codex、Cursor、Gemini CLI 等客户端直接识别与安装。

![license](https://img.shields.io/badge/license-MIT-green)
![agent-skills](https://img.shields.io/badge/agent--skills-spec%20^1-blue)

## 安装

整仓交互选择：

```bash
npx skills add AtomUI/AtomUI.Skills
```

单个技能（需写全路径，子命令不做 `skills/` 前缀回退）：

```bash
npx skills add AtomUI/AtomUI.Skills/skills/<Key>
```

## 技能一览

| Key | 名称 | 分类 | 版本 |
| --- | --- | --- | --- |
| _（暂无技能）_ | | | |

## 目录结构

```text
AtomUI.Skills/
├── README.md          本文件
├── LICENSE            仓库级许可（MIT，与 skill.json.license 对应）
├── .gitattributes     强制 UTF-8 / LF，防平台换行抖动造成假更新
├── skillshub.json     仓库级清单（身份 / 兼容面 / 同步声明）
├── .github/           PR 契约校验与审查归属
├── template/          新技能模板
└── skills/            同步发现根（唯一发现单位）
    └── <Key>/         单技能目录
        ├── SKILL.md   技能正文（给 Agent 读）+ frontmatter
        ├── skill.json 官网展示元信息（三语，唯一权威）
        ├── references/ 可选，仅 .md 长参考
        └── assets/    可选，webp / png / svg
```

## 新增技能

从 `template/` 复制一份，改目录名为技能 `Key`（`^[A-Za-z_][A-Za-z0-9_]*$`，≤64），
再按 [技能研发规范](docs/standards/skill-authoring.md) 填写内容。

**硬性前置**：`skill.json.category` 必须引用一个已在 WebOS **创建并启用**的分类 Key，
否则该技能同步会失败（未声明 / 不存在 / 被禁用分别报不同错误）。

## 规范

本仓库内容受 SkillsHub 规范约束，四份配套文档：

| 文档 | 管什么 |
| --- | --- |
| [工程规范](docs/standards/skillshub-engineering.md) | 仓库整体：目录布局、根清单、校验规则、增量哈希、CI 与治理 |
| [技能结构与元信息](docs/standards/skill-structure.md) | 单技能形态：目录布局、Key 与 name 投影、frontmatter 字段 |
| [技能元信息](docs/standards/skill-metadata.md) | `skill.json` schema、三语与回退、分类引用 |
| [技能研发规范](docs/standards/skill-authoring.md) | 内容怎么写：章节模板、尺寸预算、语言策略、安全红线 |

## 安全约定

本仓库是**知识型技能目录**：只含 Markdown 与图片资产，**禁止脚本与可执行内容**。
正文外链仅允许 `atomui.net` 与 `github.com/AtomUI*`，禁止远程指令与越权措辞。
