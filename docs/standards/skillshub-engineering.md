---
summary: "SkillsHub 工程规范 v1.0：仓库目录布局、根清单 skillshub.json、校验分层与规则总表、选中同步数据源、增量哈希、CI 与仓库治理、实施状态"
read_when:
  - 为 AtomUI.Skills 仓库做工程决策（目录、校验、CI、治理）或排查同步失败时阅读。
title: "SkillsHub 工程规范 正式标准"
---

# SkillsHub 工程规范 正式标准

> 状态：**标准 v1.0**（2026-10-24 定稿，首次发布）。在 2026-10-23 已实施的同步引擎基线之上对齐业界事实标准（Anthropic Agent Skills 规范，业界被 skills.sh、Claude Code、OpenClaw/ClawHub、OpenAI Codex、Google Gemini CLI 等共同采纳）。`AtomUI/AtomUI.Skills` 仓库当前为空，落地无迁移成本。新增校验在同步引擎升级前不生效，差异见 §实施状态。

## 标准体系

本规范是 SkillsHub 标准体系的工程总纲，与以下三份配套（同目录）：

| 文档 | 管什么 |
|------|--------|
| 本文件（工程规范） | 仓库整体长什么样：目录布局、根清单、校验规则总表、增量哈希、CI 与治理 |
| [技能结构与元信息描述](./skill-structure.md) | 单个技能长什么样：`<Key>/` 目录、SKILL.md frontmatter 字段语义 |
| [技能元信息描述](./skill-metadata.md) | `skill.json` schema：官网展示元信息、三语与回退、投影源 |
| [技能研发规范](./skill-authoring.md) | 技能内容怎么写好：正文结构、语言策略、质量流程、安全红线 |

## 设计原则

1. **仓库权威**：仓库是技能文案与结构的唯一权威；同步引擎按本规范解析，校验失败的技能不落库并逐条报错。
2. **业界事实标准兼容**：SKILL.md 遵循 Anthropic Agent Skills 规范（YAML frontmatter + Markdown 正文，字段语义见[技能结构与元信息描述](./skill-structure.md)）。业界 CLI（`npx skills add`、Claude Code 插件、ClawHub 等）可直接识别、安装本仓库技能，技能也能被 skills.sh 等目录收录。
3. **双元信息分层、单一权威**：frontmatter 只承载业界互操作所需最小集（`name` / `description` / `license`），由 `skill.json` 投影生成并由校验强制一致（投影规则见[技能元信息描述](./skill-metadata.md)）；官网展示元信息权威仍在 `skill.json`——避免双权威漂移。
4. **展示型目录、无执行载荷**：技能目录只含 Markdown 与图片资产，禁止脚本与可执行内容（业界 ClawHub 供应链安全事件的教训）；`SKILL.md` 的消费方在 Agent 客户端，本仓库不提供安装器与运行时（业界工具按规范自发现）。

## 非目标

- 不定义技能在 Agent 客户端的运行时行为（那是 `SKILL.md` 正文自身的约定）。
- 不在仓库维护分类树（分类在 WebOS 维护，`skill.json.category` 按 Key **硬引用**：引用的分类必须已在 WebOS 存在且启用，否则该技能同步失败并记错误日志；不存在「归未分类」的降级路径）。
- 不支持单技能多版本目录（版本只是展示字段）。
- 不做技能打包与安装器。
- 不引入 `scripts/` 等执行载荷（长期；本仓库是知识型技能目录）。`references/`（仅 Markdown 参考文件，渐进披露第三档）由[技能研发规范](./skill-authoring.md)引入，引擎支持待实施（见 §实施状态）。

## 仓库目录规范

```text
AtomUI.Skills/
├── README.md                   # 仓库说明：技能一览表 + 安装示例 + 徽章（业界惯例）
├── LICENSE                     # 仓库级许可（与 skill.json.license 对应）
├── .gitattributes              # 强制 UTF-8 / LF，防平台换行抖动造成假更新
├── skillshub.json              # 仓库级清单（可选，见 §根清单）
├── .github/
│   ├── workflows/validate.yml  # PR 契约校验流水线（见 §CI 校验与仓库治理）
│   └── CODEOWNERS              # 审查归属（治理要求）
├── template/                   # 新技能模板（业界惯例，anthropics/skills 同款定位）
│   ├── SKILL.md
│   ├── skill.json
│   └── assets/
└── skills/                     # 同步发现根（唯一发现单位）
    └── <Key>/                  # 单技能目录，结构见「技能结构与元信息描述」
```

要点：

- 发现单位：`skills/*/skill.json`；`template/` 与根文件不参与发现。
- `Key` 是系统级业务标识符（`^[A-Za-z_][A-Za-z0-9_]*$` ≤64，目录名 = `skill.json.key`），frontmatter `name` 由 Key 投影（kebab-lowercase）——细则见[技能结构与元信息描述](./skill-structure.md)。
- 同步分支固定取配置 `Modules:AiResources:Skills:Branch`（默认 `main`）；`skillshub.json.sync.branch` 仅是声明性默认，不覆盖服务端配置。
- 文本文件 UTF-8（无 BOM）、LF。

## 根清单 `skillshub.json`（可选）

仓库级身份与兼容声明，定位对标 Claude Code 插件市场的 `.claude-plugin/marketplace.json`；缺失 → 同步记警告（不失败，向后兼容）：

```json
{
  "schemaVersion": 1,
  "registry": {
    "name": "AtomUI Skills",
    "publisher": "AtomUI",
    "homepage": "https://atomui.net"
  },
  "compat": {
    "agentSkillsSpec": "^1",
    "agents": ["claude-code", "codex", "cursor"]
  },
  "sync": {
    "branch": "main",
    "discovery": "skills/*/skill.json"
  }
}
```

- `registry`：注册表身份；未来对外列入 skills.sh 等目录时的元信息来源。
- `compat`：兼容面声明；`agents` 为展示性声明，同步引擎不校验值域（与 `skill.json.agents` 定位一致）。
- `sync`：声明性默认，不覆盖服务端配置；非法或不认识字段 → 警告忽略。

## 校验分层（两阶段）

校验不是单一纯函数层：**结构校验**只看仓库内容，**状态校验**要查 WebOS 现状。二者输入不同，必须分层，禁止把 DB 查询塞进纯函数校验器。

| 阶段 | 输入 | 落点 | 职责 |
|------|------|------|------|
| 一、结构校验 | 目录 + `skill.json` + `SKILL.md` + `assets` | `SkillCatalogValidator`（纯函数） | Key/字段/长度/值域/安全条款 |
| 二、状态校验 | 解析结果 + WebOS 分类 Key 集合 | 同步执行器（有 DB 访问） | `category` 存在性与启用状态 |

状态校验的失败同样产出 `CatalogIssue`（`Level=Error`），并入该技能的 `Issues`，使其 `HasErrors` 为真 → 计入 `FailedKeys` → 不落库、逐条记错误日志。解析失败的技能**绝不判孤儿**（其 Key 仍在 `seenKeys` 中）。

## 校验规则总表（逐条独立，失败技能不落库、不阻断整批）

同步引擎已实施的基线规则全部保留；本标准新增/收紧的规则以「待实施」标注：

| 类别 | 规则 | 实施状态 |
|------|------|------|
| 标识 | `key` 正则；目录名一致；仓库内唯一（重复 → 后者失败并报 key 冲突） | 已实施 |
| 一致性 | frontmatter 存在且可解析；`name` ≡ Key 投影；`description` ≡ `en.summary`；`license` 两侧同填或同空；仓库内 `name` 唯一（细则见[技能结构与元信息描述](./skill-structure.md)） | 待实施 |
| 语言 | 必含 `en`；`en.summary` 必填非空；`zh-Hans`/`zh-Hant` 缺失回退标注（见[技能元信息描述](./skill-metadata.md)） | 待实施（收紧） |
| 文本 | `name` 非空 ≤128；`summary` ≤300；`description` ≤64KB | 已实施 |
| 集合 | `tags`/`platforms`/`agents` ≤16 去重；`platforms` 值域校验 | 已实施 |
| 文件 | `SKILL.md` 必填 ≤256KB；`assets/**` 合计 ≤2MB；扩展名 ∈ {webp, png, svg} | 已实施 |
| 安全 | 技能目录内禁 symlink；禁可执行文件（扩展名 `.sh .ps1 .exe .bat .cmd .dll .so .dylib` 或带可执行位）；禁隐藏文件/目录（`.` 开头） | 待实施 |
| 根清单 | `skillshub.json` 缺失 → 警告；存在但非法 → 警告忽略其声明 | 待实施 |
| 引用 | `category` 未声明 → 该技能失败（`CategoryRequired`） | 已实施 |
| 引用 | `category` 已声明但 WebOS 无此分类 → 该技能失败（`CategoryUnknown`） | 已实施 |
| 引用 | `category` 已声明但分类被禁用 → 该技能失败（`CategoryDisabled`） | 已实施 |
| 支撑文件 | `references/**` 仅 `.md`、SKILL.md 引用深度一层且 ≤8 个（见[技能研发规范](./skill-authoring.md)） | 待实施 |
| 致命 | 整仓解析出 0 个合法技能 → `030006`，run Failed，且不执行孤儿扫描 | 已实施 |

## 增量哈希（`SourceContentHash`）

```
SHA256(
  canonicalJson(skill.json 语义字段集：key, category, version, author, license,
                repositoryUrl, tags, platforms, agents, priority, languages[*])
  + SHA256(SKILL.md bytes)
  + Σ(按路径排序) path + ":" + SHA256(asset bytes)
)
```

- canonical 序列化：键排序、UTF-8、无缩进——避免格式抖动造成假更新；
- frontmatter 属于 `SKILL.md` 字节，天然参与哈希，无需单列；
- 哈希相同 → `Unchanged`（只更新 `LastSyncedTime`）；
- 哈希变化但某资产字节未变 → 该资产不重新上传（按资产哈希比对）。

## CI 校验与仓库治理

业界惯例（anthropics/skills、obra/superpowers 等内容仓均自含校验）：

- **PR 校验**：`.github/workflows/validate.yml` 对变更运行契约校验器，与同步引擎**同一套规则、单一来源**（工具形态见 §开放问题）；校验红 → 阻止合并。
- **分支保护**：`main` 禁 force push；合并要求 PR + CI 绿 + CODEOWNERS 审查通过。
- **README**：维护技能一览表（Key / 名称 / 分类 / 版本）与安装示例：按仓库安装 `npx skills add AtomUI/AtomUI.Skills`（交互选择），或单技能全路径 `npx skills add AtomUI/AtomUI.Skills/skills/<Key>`（业界 CLI 子路径不做 `skills/` 前缀回退，须写全）。
- **提交约定**：一个技能一个目录一次提交；内容变更同步递增 `version`。

## 选中同步的数据源（与用户手册一致）

`SkillSyncMode.SelectedSkills` 的可选清单来自**已入库的技能列表**，与用户手册「增量发布」按已同步快照选章节的做法一致：**先全量同步一次，之后才能按技能增量同步**。

- **不做同步前发现**：不去仓库实时捞清单。手册的 `GetTocAsync` 同样只读已同步快照（无 `ActiveSnapshotId` 即报 `ManualSelectedDocumentSyncRequiresSnapshot`），技能侧对齐该模型。
- **不引入独立目录文件**：`skills/*/skill.json` 已是完整的技能清单，再维护一份 `catalog` 只会多出一个会漂移的真相源，破坏设计原则 3。
- **首次同步**：空库时无可选项，必须先 `Full` 同步一次。
- **未入库的技能选不到**：与手册一致的取舍——换取交互一致性，且不新增任何接口与代理。

## 模块联动

- 仓库内容无状态机；库内状态变化（Add/Update/Unchanged/Orphan 与孤儿翻转）见 [sync.md](../sync.md)。
- 本规范无独立 UI；同步失败清单在 WebOS 同步记录抽屉展示，错误摘要使用解析器输出的结构化原因（`CatalogIssue.Rule` + 明文 `Message`）。**报错日志不做本地化**——日志是排查产物而非界面文案，明文原因比译文更利于定位。
- 解析入口：`SkillCatalogParser.Parse(checkoutRoot)` → `ParsedSkill(Key, SourcePath, Manifest, SkillMd, Assets, Errors)`；校验在 `SkillCatalogValidator`（纯函数）。数据写入：`Skill.Source*`、`SkillContent`（三语）、资产经 Foundation 上传后只存资源 ID。

## 实施状态

同步引擎已实施 2026-10-23 基线（目录规范、`skill.json` 基础校验、增量哈希）；以下为本标准 v1.0 新增、待落地：

| 项 | 落点 |
|----|------|
| frontmatter 解析与一致性校验 | `SkillCatalogParser` / `SkillCatalogValidator` |
| `en.summary` 必填收紧 | `SkillCatalogValidator` |
| 安全条款（symlink / 可执行 / 隐藏文件检测） | `SkillCatalogParser` |
| `skillshub.json` 读取 + 缺失警告 | `SkillCatalogParser` |
| `references/` 目录解析、`.md` 白名单、纳入增量哈希 | `SkillCatalogParser` / `SkillCatalogValidator`（见[技能研发规范](./skill-authoring.md)） |
| 独立校验 CLI（供 CI 使用，与 SyncEngine 同源规则） | 新工具项目（形态见 §开放问题） |
| 仓库初始化：README / LICENSE / .gitattributes / skillshub.json / template / CODEOWNERS / validate.yml | `AtomUI/AtomUI.Skills` |

## 测试矩阵

见 [tests.md](../tests.md) §解析器（已实施基线全规则矩阵 + 脏 JSON 降级 + 零技能致命 + 哈希稳定性）+ 待补充：frontmatter 一致性（缺失/不匹配/name 冲突）、安全条款（symlink/可执行/隐藏文件各一例）、根清单缺失警告、`references/` 校验。

## 开放问题

- CI 校验 CLI 形态（dotnet tool / 独立脚本）与发布方式。
- `references/` 的启用时点：格式与写作规范已定（见[技能研发规范](./skill-authoring.md)），引擎支持是否随首批技能落地待定。
