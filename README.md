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

单个技能（需写全路径，含分类层；子命令不做 `skills/` 前缀回退）：

```bash
npx skills add AtomUI/AtomUI.Skills/skills/<category_key>/<skill_key>
```

## 目录文件 `catalog.json`

**`catalog.json` 是本库的目录权威**（git 提交时刻生效），负责四件事：

| 职责 | 说明 |
| --- | --- |
| 收录清单 | 哪些技能属于本库 |
| 分类归属 | 每个技能属于哪个分类（恰好一层） |
| 组内顺序 | `priority` + 数组顺序 |
| 前台白名单 | **不在 `catalog.json` 中出现的技能不上 atomui.net 前台** |

`catalog.json` 管「它在库里的位置与收录状态」，`skill.json` 管「它是什么」——两者字段不重叠，不构成第二真相源。

## 分类规范（atomui.net 强制）

**分类是 atomui.net 的强制规范**：分类**定义**权威在 atomui.net（WebOS `SkillCategory` + DataSeeding），
任何接入 WebOS 的 skillshub 都必须遵守。本仓库只**引用**分类，不得自造、不维护分类清单。

- 技能放在 `skills/<category_key>/<skill_key>/`，**恰好一层分类**。
- `category_key` 必须是 atomui.net 已定义且启用的**顶层**分类 Key。
- 分类归属由目录路径表达，并由 `catalog.json` 声明；两者必须一致。

## 技能一览

<!-- 本表由 catalog.json 生成，勿手改 -->

| Key | 名称 | 分类 | 版本 | priority |
| --- | --- | --- | --- | --- |
| [`hello_world`](skills/AtomUIOSS/hello_world) | Hello World 示例技能 | AtomUIOSS | 1.0.0 | 1000 |

## 目录结构

```text
AtomUI.Skills/
├── README.md          本文件
├── LICENSE            仓库级许可（MIT，与 skill.json.license 对应）
├── .gitattributes     强制 UTF-8 / LF，防平台换行抖动造成假更新
├── catalog.json       目录文件（收录 + 分类归属 + 顺序 + 前台白名单）
├── skillshub.json     仓库级清单（身份 / 兼容面 / 同步声明）
├── .github/           PR 契约校验与审查归属
├── template/          新技能模板（含分类目录示例）
└── skills/            同步发现根（唯一发现单位）
    └── <category_key>/        分类目录，恰好一层
        └── <skill_key>/       单技能目录
            ├── SKILL.md       技能正文（给 Agent 读）+ frontmatter
            ├── skill.json     官网展示元信息（三语，唯一权威）
            ├── references/    可选，仅 .md 长参考
            └── assets/        可选，webp / png / svg
```

## 新增技能

1. 从 `template/AtomUIOSS/` 复制其中的文件到 `skills/<category_key>/<skill_key>/`，两层目录名分别改成分类 Key 与技能 `Key`
   （`^[A-Za-z_][A-Za-z0-9_]*$`，≤64，**用下划线不用短横线**）。
2. 按[技能研发规范](docs/standards/skill-authoring.md)填 `SKILL.md` 与 `skill.json`。
3. 在根 `catalog.json` 的对应 `categories[].skills[]` 登记（`key` + 显式 `priority`，可选 `text`）。

**硬性前置**：`skills/<category_key>/` 的分类必须是 atomui.net **已创建并启用**的顶层分类 Key，
且技能必须在 `catalog.json` 中登记——否则同步失败或不上前台。

## 规范

本仓库内容受 SkillsHub 规范约束，四份配套文档：

| 文档 | 管什么 |
| --- | --- |
| [工程规范](docs/standards/skillshub-engineering.md) | 仓库整体：目录布局、目录文件 `catalog.json`、分类规范、根清单、校验规则、双内容哈希、CI 与治理 |
| [技能结构与元信息](docs/standards/skill-structure.md) | 单技能形态：一层分类目录、Key 与 name 投影、frontmatter 字段 |
| [技能元信息](docs/standards/skill-metadata.md) | `skill.json` schema、三语与回退、frontmatter 投影源、分类引用三态 |
| [技能研发规范](docs/standards/skill-authoring.md) | 内容怎么写：章节模板、尺寸预算、语言策略、安全红线 |

## 安全约定

本仓库是**知识型技能目录**：只含 Markdown 与图片资产，**禁止脚本与可执行内容**。
正文外链仅允许 `atomui.net` 与 `github.com/AtomUI*`，禁止远程指令与越权措辞。
