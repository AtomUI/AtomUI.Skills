---
summary: "技能结构与 SKILL.md 元信息描述 v1.0：一层分类目录与单技能目录布局、Key 与 name 投影、frontmatter 字段语义（基线=Agent Skills 开放标准）、references 与 assets 规则"
read_when:
  - 编写、评审或校验单个技能的目录结构与 SKILL.md frontmatter 时阅读。
title: "技能结构与 SKILL.md 元信息描述 正式标准"
---

# 技能结构与 SKILL.md 元信息描述 正式标准

> 状态：**标准 v1.0**（2026-10-24 定稿；**2026-10-25 修订**：技能前增加一层分类目录，`skill.json` 移除 `category`）。按仓库约定**不递增版本号**。
> 与[SkillsHub 工程规范](./skillshub-engineering.md) 配套：工程规范管仓库整体（含 `catalog.json` 与分类规范），本规范管「一个技能长什么样」。校验规则总表与实施状态见工程规范，此处只定义结构与字段语义。

## 业界基线

SKILL.md 格式以 **Agent Skills 开放标准**（[agentskills.io/specification](https://agentskills.io/specification)，Anthropic 主导）为基线，业界已事实收敛：

| 厂商 | 采纳情况 | 对本规范的输入 |
|------|----------|----------------|
| Anthropic | 正典：SKILL.md = YAML frontmatter + Markdown；`name`/`description` 必填，`license`/`compatibility`/`metadata`/`allowed-tools` 可选；渐进披露三档 | 基线格式与字段语义 |
| OpenAI | 2025-12 起采纳同一标准（`SKILL.md` 大小写不敏感、每技能恰一份；frontmatter 校验引用 agentskills.io） | 单文件约束 |
| Google | Gemini CLI 采纳同一标准，技能跨 Claude Code / Codex / Gemini 不改可装 | 可移植性验证 |
| OpenClaw | 同一 SKILL.md 底座 + `metadata.openclaw` 扩展（`requires`/`install`/`os`）；发布时校验「声明元数据 vs 实际行为」一致性 | 扩展字段放 `metadata` 命名空间的先例 |

本规范不发明冲突格式：任何此处未规定的细节以开放标准为准；我们只做收紧（投影一致性）与本地化（Key 规范）。

> 分类目录层是 **AtomUI 站点侧**的组织约定，不进入 Agent Skills 开放标准的字段体系——它只影响路径，不影响 SKILL.md 与 `skill.json` 的语义。

## 目录结构（一层分类）

```text
skills/
└── <category_key>/                # 分类目录，恰好一层；= atomui.net 分类 Key
    └── <Key>/                     # 技能目录名 = Key，^[A-Za-z_][A-Za-z0-9_]*$ ≤64
        ├── SKILL.md               # 必填；技能正文（给 Agent 读）+ 规范 frontmatter
        ├── skill.json             # 必填；官网展示元信息（三语），见「技能元信息描述」
        ├── references/            # 可选；仅 .md 长参考（渐进披露第三档，引擎待实施）
        │   └── tokens.md
        └── assets/
            ├── cover.webp         # 列表卡封面（可选，建议 16:9）
            └── icon.svg           # 图标（可选）
```

### 布局硬约束

| 规则 | 说明 |
|------|------|
| 恰好两层 | 技能必须位于 `skills/<category_key>/<Key>/`；`skills/<Key>/`（缺分类层）与 `skills/<a>/<b>/<Key>/`（多一层）均校验失败（`LayoutDepth`） |
| 分类目录名 | = 分类 Key，`^[A-Za-z_][A-Za-z0-9_]*$` ≤64；必须是 atomui.net 已定义且启用的**顶层**分类 |
| 技能目录名 | = 技能 Key，`^[A-Za-z_][A-Za-z0-9_]*$` ≤64 |
| 分类归属 | 由**目录路径的 `<category_key>` 段**表达，并由 `catalog.json` 声明；两者必须一致 |
| 收录 | 技能是否上台由 `catalog.json` 决定；`skills/` 下存在但未收录的技能标记 `IsListed=false`，入库可查、前台不展示 |

分类层数与分类 Key 值域的完整规则见[工程规范](./skillshub-engineering.md) §分类规范（atomui.net 强制）。

## Key 与目录名

- 技能 `Key` 是系统级业务标识符：`^[A-Za-z_][A-Za-z0-9_]*$`（≤64），**不用短横线/点/中文**（AtomIdea 标识符规范，与 WebOS 分类 Key 同规范）。
- 技能目录名必须与 `skill.json.key` 完全一致；仓库内唯一。
- **分类目录名不参与 `name` 投影**——`name` 只由技能 `Key` 推导，与所在分类无关（技能换分类不改 `name`，避免破坏业界工具的安装路径稳定性）。

## frontmatter `name` 投影

- 投影规则：`Key` 经 `ToLowerInvariant()` 后把 `_` 替换为 `-`，满足业界 kebab-lowercase 惯例（例：`avalonia_theme_tokens` → `avalonia-theme-tokens`），同时满足开放标准 `name` 规则（小写字母/数字/短横线、不以短横线开头结尾、无连续短横线、≤64、与父目录对应——业界侧按目录名+投影一致性成立）。
- **仓库内 `name` 唯一**：大小写不同的 Key 可能投影到同一 `name`（`My_Skill` 与 `my_skill` 都得 `my-skill`），校验按 name 冲突报错。跨分类同样唯一。
- `name` 由技能目录名机器推导，**不手写**；命名从任务视角出发的写法建议见[技能研发规范](./skill-authoring.md)。

## SKILL.md frontmatter 规范

业界工具以 frontmatter 识别技能；缺失或非法 = 对业界工具不可见。格式与 Agent Skills 开放标准一致：

```markdown
---
name: avalonia-theme-tokens
description: AtomUI theme token conventions for Avalonia covering palette, sizing, spacing and dark-mode rules.
license: MIT
---

# Avalonia Theme Tokens（正文 · 给 Agent 读）

…Markdown 指令…
```

### 必填字段（我们的投影收紧）

| 字段 | 必填 | 规则 |
|------|------|------|
| `name` | ✓ | ≡ 技能 Key 的 kebab 投影（见上）；仓库内唯一；≤64 |
| `description` | ✓ | ≡ `skill.json.languages.en.summary`（英文，≤300，天然满足业界 ≤1024 上限）；三要素写法见[技能研发规范](./skill-authoring.md) |
| `license` | — | 若填，必须 ≡ `skill.json.license`（两侧同填或同空） |

> `category` **不是** frontmatter 字段，也已从 `skill.json` 移除：分类归属由目录路径 + `catalog.json` 表达（见上）。这使 `skill.json` 与 `catalog.json` 字段不重叠，避免双写。

### 业界可选字段（透传，不校验）

| 字段 | 开放标准语义 | 我们的态度 |
|------|--------------|------------|
| `compatibility` | ≤500 字符；环境前提（目标产品、系统包、网络需求） | 允许出现；知识型技能通常无需 |
| `metadata` | 字符串键值映射；客户端扩展元信息（OpenClaw 扩展即放此处） | 允许出现；扩展元信息建议放 `skill.json`（三语权威），避免双源 |
| `allowed-tools` | 空格分隔的预授权工具清单（实验性） | 允许出现；无脚本、无工具预授权诉求，通常不填 |

同步引擎对业界可选字段不消费、不校验（透传给客户端）。

### 正文约束

Markdown ≤256KB（硬上限，引擎校验）；frontmatter 缺失、YAML 解析失败或必填字段非法 → 该技能校验失败（逐条独立，不阻断整批）。正文写作规范（章节模板、语言策略、软预算）见[技能研发规范](./skill-authoring.md)。

## `references/`（引擎待实施）

承接超长参考内容（完整 token 表、全量组件规范），让 SKILL.md 正文保持导航层，对应渐进披露第三档：

- 仅 `.md` 文件；不与 `assets/` 混放；
- SKILL.md 内相对路径链接（`[完整 token 表](references/tokens.md)`），**引用深度一层**（SKILL.md → references/x.md，不再嵌套）；
- 单个 SKILL.md 引用的 references 文件 ≤8 个；
- 实施状态与哈希纳入见[工程规范](./skillshub-engineering.md) §实施状态；实施前先不提交该目录。

## `assets/`

- 扩展名 ∈ {webp, png, svg}；合计 ≤2MB（引擎校验）；
- `cover.webp`：官网列表卡封面，建议 16:9；`icon.svg`：图标（均可选）；
- 资产是展示资源，Agent 不消费；正文引用图片一律走官网域名，不依赖本地资产路径。

## 文件级约束

- 文本文件 UTF-8（无 BOM）、LF（仓库根 `.gitattributes` 强制）；
- 技能目录内禁 symlink、可执行文件与隐藏文件/目录（安全条款，见工程规范 §校验规则总表）。
