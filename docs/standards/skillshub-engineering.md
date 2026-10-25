---
summary: "SkillsHub 工程规范 v1.0：仓库目录布局、目录文件 catalog.json、atomui.net 强制分类规范、根清单 skillshub.json、校验分层与规则总表、双内容哈希、CI 与仓库治理、实施状态"
read_when:
  - 为 AtomUI.Skills 仓库做工程决策（目录、catalog、分类、校验、CI、治理）或排查同步失败时阅读。
title: "SkillsHub 工程规范 正式标准"
---

# SkillsHub 工程规范 正式标准

> 状态：**标准 v1.0**（2026-10-24 定稿；**2026-10-25 修订**：引入目录文件 `catalog.json`、一层分类目录、atomui.net 强制分类规范与双内容哈希）。按仓库约定**不递增版本号**。
> 基线对齐业界事实标准（Anthropic Agent Skills 规范，业界被 skills.sh、Claude Code、OpenClaw/ClawHub、OpenAI Codex、Google Gemini CLI 等共同采纳）。新增校验在同步引擎升级前不生效，差异见 §实施状态。

## 标准体系

本规范是 SkillsHub 标准体系的工程总纲，与以下三份配套（同目录）：

| 文档 | 管什么 |
|------|--------|
| 本文件（工程规范） | 仓库整体长什么样：目录布局、目录文件 `catalog.json`、分类规范、根清单、校验规则总表、双内容哈希、CI 与治理 |
| [技能结构与元信息描述](./skill-structure.md) | 单个技能长什么样：`<category>/<Key>/` 目录、SKILL.md frontmatter 字段语义 |
| [技能元信息描述](./skill-metadata.md) | `skill.json` schema：官网展示元信息、三语与回退、投影源 |
| [技能研发规范](./skill-authoring.md) | 技能内容怎么写好：正文结构、语言策略、质量流程、安全红线 |

## 设计原则

1. **仓库权威**：仓库是技能文案与结构的唯一权威；同步引擎按本规范解析，校验失败的技能不落库并逐条报错。
2. **业界事实标准兼容**：SKILL.md 遵循 Anthropic Agent Skills 规范（YAML frontmatter + Markdown 正文，字段语义见[技能结构与元信息描述](./skill-structure.md)）。业界 CLI（`npx skills add`、Claude Code 插件、ClawHub 等）可直接识别、安装本仓库技能。
3. **三层元信息分层、各自单一权威**：
   - `catalog.json` 管**库结构**——收录谁、属于哪个分类、什么顺序；
   - `skill.json` 管**技能是什么**——三语、标签、平台、版本；
   - SKILL.md frontmatter 管**业界互操作**——由 `skill.json` 投影生成并由校验强制一致。
   三层字段**不重叠**，因此不存在会漂移的第二真相源。
4. **分类是 atomui.net 的强制规范**：分类**定义**权威在 atomui.net（WebOS `SkillCategory` + DataSeeding），任何接入 WebOS 的 skillshub 都必须遵守；仓库只**引用**分类，不得自造、不维护分类清单。
5. **展示型目录、无执行载荷**：技能目录只含 Markdown 与图片资产，禁止脚本与可执行内容（业界 ClawHub 供应链安全事件的教训）；`SKILL.md` 的消费方在 Agent 客户端，本仓库不提供安装器与运行时。

## 非目标

- 不定义技能在 Agent 客户端的运行时行为（那是 `SKILL.md` 正文自身的约定）。
- **不在仓库定义分类**：不放 `categories.json` 之类的分类定义文件；分类 Key 的值域由 atomui.net 决定。
- **不做多层分类**：只允许一层分类目录，且只可引用顶层分类（WebOS `SkillCategory.ParentId` 保留但 skillshub 不跟随）。
- 不支持单技能多版本目录（版本只是展示字段）。
- 不做技能打包与安装器。
- 不引入 `scripts/` 等执行载荷（长期；本仓库是知识型技能目录）。`references/`（仅 Markdown 参考文件，渐进披露第三档）由[技能研发规范](./skill-authoring.md)引入，引擎支持待实施（见 §实施状态）。

## 仓库目录规范

```text
AtomUI.Skills/
├── README.md                   # 仓库说明：技能一览表（由 catalog.json 生成）+ 安装示例 + 徽章
├── LICENSE                     # 仓库级许可（与 skill.json.license 对应）
├── .gitattributes              # 强制 UTF-8 / LF，防平台换行抖动造成假更新
├── catalog.json                # 目录文件：收录清单 + 分类归属 + 顺序 + 前台白名单（见 §目录文件）
├── skillshub.json              # 仓库级清单（可选，见 §根清单）
├── .github/
│   ├── workflows/validate.yml  # PR 契约校验流水线（见 §CI 校验与仓库治理）
│   └── CODEOWNERS              # 审查归属（治理要求）
├── template/                   # 新技能模板
│   └── <category_key>/<skill_key>/
│       ├── SKILL.md
│       └── skill.json
└── skills/                     # 同步发现根（唯一发现单位）
    └── <category_key>/         # 分类目录，恰好一层
        └── <skill_key>/        # 单技能目录，结构见「技能结构与元信息描述」
```

要点：

- 发现单位：`skills/<category_key>/<skill_key>/skill.json`；`template/` 与根文件不参与发现。
- **布局硬约束**：技能必须在 `skills/<category>/<key>/` 两层之下；`skills/<key>/`（缺分类层）与 `skills/<a>/<b>/<key>/`（多一层）均校验失败。
- `category_key` = 分类 Key，`skill_key` = 技能 Key，两者均遵循 `^[A-Za-z_][A-Za-z0-9_]*$` ≤64，且目录名 = 对应声明值。
- 技能 `Key` 是系统级业务标识符，frontmatter `name` 由 Key 投影（kebab-lowercase）——细则见[技能结构与元信息描述](./skill-structure.md)。
- 同步分支固定取配置 `Modules:AiResources:Skills:Branch`（默认 `main`）；`skillshub.json.sync.branch` 仅是声明性默认，不覆盖服务端配置。
- 文本文件 UTF-8（无 BOM）、LF。

## 目录文件 `catalog.json`（必填）

**`catalog.json` 是 git 提交时刻的目录权威**，负责四件事：**收录清单**、**分类归属**、**组内顺序与目录显示名**、**前台白名单**（不在本文件中出现的技能不上 atomui.net 前台）。

### 形态

```json
{
  "schemaVersion": 1,
  "text": "AtomUI Skills",
  "categories": [
    {
      "key": "AtomUIOSS",
      "text": "AtomUI 开源项目",
      "skills": [
        { "key": "hello_world", "priority": 1000, "text": "Hello World 示例技能" },
        { "key": "avalonia_theme_tokens", "priority": 900 }
      ]
    }
  ]
}
```

| 节点 | 字段 | 必填 | 约束与语义 |
|------|------|------|------------|
| 根 | `schemaVersion` | ✓ | 固定 `1` |
| 根 | `text` | — | 库名，≤128；缺省取 `skillshub.json.registry.name` |
| 根 | `categories` | ✓ | 非空；**恰好一层**，不嵌套 |
| 分类 | `key` | ✓ | 引用 atomui.net 分类 Key（硬引用，三态见 §校验规则总表） |
| 分类 | `text` | — | 目录上的分类显示名，≤128；缺省回退 WebOS 分类名 |
| 分类 | `skills` | ✓ | 非空；**数组顺序 = 组内展示顺序的最终裁决** |
| 技能 | `key` | ✓ | 引用 `skills/<category_key>/<key>/`；仓库内唯一 |
| 技能 | `priority` | — | 显式整数，缺省 `1000`。**必须显式，不从数组下标推导**（否则插入一个技能会连坐重算后续全部技能） |
| 技能 | `text` | — | 目录显示名，≤128；缺省取 `skill.json` 当前语言 `name`（回退 `en`） |

### 权威边界

| 域 | 权威 |
|----|------|
| 分类**定义**（有哪些分类、启停） | **atomui.net**（WebOS `SkillCategory` + DataSeeding） |
| 分类**归属**（技能属于哪个分类） | `catalog.json` 声明 **+** 目录路径 `skills/<category>/<key>/`，两者必须一致 |
| 收录（哪些技能上台）与前台白名单 | **`catalog.json`** |
| 组内顺序、目录显示名 | **`catalog.json`**（`priority` / `text`） |
| 技能自身元信息（三语 name/summary/description、tags、platforms、agents、version、author、license、repositoryUrl） | **`skill.json`** |

### 与 `skill.json` 的一致性

`catalog.json` 与 `skill.json` **字段不重叠**，因此不是第二真相源。唯一交叠的分类归属由目录路径表达：catalog 声明的 `categories[].key` 必须等于该技能所在目录的 `<category_key>` 段（`CatalogCategoryMismatch`）。

这与用户手册 `docs-nav.json` / 文档正文的分工同构：**目录文件声明「它在库里的位置与收录状态」，单技能文件声明「它是什么」**。

### 读取方式

`catalog.json` 是单文件，同步引擎与 WebOS 目录浏览均可用 `git show <ref>:catalog.json` 直接从 git 对象库读取，**不需要 checkout**；`sourceCommit` 取同刻 `git rev-parse <ref>` 并锁定进同步任务。

## 分类规范（atomui.net 强制）

| 维度 | 规则 |
|------|------|
| 分类定义权威 | **atomui.net**（WebOS `SkillCategory` + DataSeeding 资产 `AtomUINET.DataSeeding/Assets/AiResources/Categories/categories.json`） |
| skillshub 的义务 | 只能引用**已定义且 `IsEnabled=true`** 的分类 Key；不得自造分类；仓库内不维护分类清单 |
| 归属表达 | 目录路径 `skills/<category_key>/<skill_key>/` **与** `catalog.json` 声明，二者一致 |
| 分类层数 | **恰好一层**；`CategoryNotTopLevel` 规则阻止引用带 `ParentId` 的非顶层分类 |
| 三态校验 | 未定义 / 不存在 / 已禁用分别产出不同错误；该分类下全部技能失败、不落库、**绝不判孤儿** |
| 删除守卫 | WebOS 侧沿用：有技能引用的分类不可删除，需要下线时**禁用**（详见[技能元信息描述](./skill-metadata.md) §分类删除守卫） |

「只做一层」是刻意收敛：仓库目录深度不与 WebOS 分类树深度耦合，避免分类树变化牵动仓库结构。

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
    "discovery": "skills/*/*/skill.json"
  }
}
```

- `registry`：注册表身份；未来对外列入 skills.sh 等目录时的元信息来源。
- `compat`：兼容面声明；`agents` 为展示性声明，同步引擎不校验值域（与 `skill.json.agents` 定位一致）。
- `sync.discovery`：声明性默认，**必须与本规范的发现单位一致**（`skills/*/*/skill.json`）；不一致 → 警告并以本规范为准。
- `sync`：声明性默认，不覆盖服务端配置；非法或不认识字段 → 警告忽略。

## 校验分层（两阶段）

校验不是单一纯函数层：**结构校验**只看仓库内容，**状态校验**要查 WebOS 现状。二者输入不同，必须分层，禁止把 DB 查询塞进纯函数校验器。

| 阶段 | 输入 | 落点 | 职责 |
|------|------|------|------|
| 一、结构校验 | `catalog.json` + 目录布局 + `skill.json` + `SKILL.md` + `assets` | `SkillCatalogValidator`（纯函数） | 布局/目录文件/Key/字段/长度/值域/安全条款 |
| 二、状态校验 | 解析结果 + WebOS 分类 Key 集合与启用态 | `SkillCategoryReferenceValidator`（有 DB 访问） | `category` 存在性、启用态、是否顶层 |

状态校验的失败同样产出 `CatalogIssue`（`Level=Error`），并入该技能的 `Issues`，使其 `HasErrors` 为真 → 计入 `FailedKeys` → 不落库、逐条记错误日志。解析失败的技能**绝不判孤儿**（其 Key 仍在 `seenKeys` 中）。

## 校验规则总表（逐条独立，失败技能不落库、不阻断整批）

| 类别 | 规则 | 实施状态 |
|------|------|----------|
| 布局 | 技能位于 `skills/<category>/<key>/` 恰好两层（`LayoutDepth`） | 待实施 |
| 目录文件 | `catalog.json` 缺失 → 整仓 Fatal（`CatalogMissing`） | 待实施 |
| 目录文件 | `catalog.json` 非法 JSON → 整仓 Fatal（`CatalogInvalidJson`） | 待实施 |
| 目录文件 | catalog 列了 key 但无对应技能目录（`CatalogKeyUnknown`，catalog 级错误） | 待实施 |
| 目录文件 | 同一 `key` 在 catalog 中出现多次（`CatalogKeyDuplicate`） | 待实施 |
| 目录文件 | catalog 声明的分类 ≠ 目录路径 `<category_key>`（`CatalogCategoryMismatch`） | 待实施 |
| 目录文件 | `text` > 128（`CatalogTextTooLong`） | 待实施 |
| 目录文件 | 技能在 `skills/` 下存在但未被 catalog 收录（`Uncatalogued`，Warning） | 待实施 |
| 标识 | `key` 正则；目录名一致；仓库内唯一（重复 → 后者失败并报 key 冲突） | 已实施 |
| 一致性 | frontmatter 存在且可解析；`name` ≡ Key 投影；`description` ≡ `en.summary`；`license` 两侧同填或同空；仓库内 `name` 唯一（细则见[技能结构与元信息描述](./skill-structure.md)） | 待实施 |
| 语言 | 必含 `en`；`en.summary` 必填非空；`zh-Hans`/`zh-Hant` 缺失回退标注（见[技能元信息描述](./skill-metadata.md)） | 待实施（收紧） |
| 文本 | `name` 非空 ≤128；`summary` ≤300；`description` ≤64KB | 已实施 |
| 集合 | `tags`/`platforms`/`agents` ≤16 去重；`platforms` 值域校验 | 已实施 |
| 文件 | `SKILL.md` 必填 ≤256KB；`assets/**` 合计 ≤2MB；扩展名 ∈ {webp, png, svg} | 已实施 |
| 安全 | 技能目录内禁 symlink；禁可执行文件（扩展名 `.sh .ps1 .exe .bat .cmd .dll .so .dylib` 或带可执行位）；禁隐藏文件/目录（`.` 开头） | 待实施 |
| 根清单 | `skillshub.json` 缺失 → 警告；存在但非法 → 警告忽略其声明；`sync.discovery` 与发现单位不一致 → 警告 | 待实施 |
| 引用 | `category` 未声明（无分类目录层 / catalog 未归类）→ 该技能失败（`CategoryRequired`） | 已实施 |
| 引用 | `category` 已声明但 WebOS 无此分类 → 该技能失败（`CategoryUnknown`） | 已实施 |
| 引用 | `category` 已声明但分类被禁用 → 该技能失败（`CategoryDisabled`） | 已实施 |
| 引用 | `category` 引用非顶层分类 → 该技能失败（`CategoryNotTopLevel`） | 待实施 |
| 支撑文件 | `references/**` 仅 `.md`、SKILL.md 引用深度一层且 ≤8 个（见[技能研发规范](./skill-authoring.md)） | 待实施 |
| 致命 | 整仓解析出 0 个合法技能 → `030006`，run Failed，且不执行孤儿扫描 | 已实施 |

## 内容哈希（双哈希）

### 单技能：`SourceContentHash`

```
SHA256(
  canonicalJson(skill.json 语义字段集: key, version, author, license,
                repositoryUrl, tags, platforms, agents, priority, languages[*])
  + canonicalJson(该技能的 catalog 字段集: category, priority, text)
  + SHA256(SKILL.md bytes)
  + Σ(按路径排序) path + ":" + SHA256(asset bytes)
)
```

- canonical 序列化：键排序、UTF-8、无缩进——避免格式抖动造成假更新；
- frontmatter 属于 `SKILL.md` 字节，天然参与哈希，无需单列；
- **不含兄弟位置**：在 catalog 中插入、删除或重排其他技能，不会连坐重算本技能的哈希；
- 哈希相同 → `Unchanged`（只更新 `LastSyncedTime`）；
- 哈希变化但某资产字节未变 → 该资产不重新上传（按资产哈希比对）。

### 版本级：`SkillRepositoryVersion.LastFullSyncCatalogHash`

```
SHA256(
  canonicalJson(Σ 按 (category, order) 排序的 {category, key, order, priority})
)
```

- **不含显示名 `text`**——改个标题不该触发「结构变化」告警；
- 用途：`SelectedSkills` 模式下若与当前 catalog 结构哈希不一致，记 Warning「目录结构已变化，增量不会收录新增技能，建议全量同步」；
- 写入时机：`Full` run 到达 `Succeeded`（含部分失败）时更新。

## 同步语义

| 主题 | 规则 |
|------|------|
| 发现 | 先读 `catalog.json` 得收录集合，再扫 `skills/<category>/<key>/` 得 `ParsedSkill`；两者双向对齐 |
| 成员判定 | `SkillSyncPolicy` 以 **catalog 收录集合**为准，不再是「扫到的目录集合」 |
| 未收录技能 | `skills/` 下存在但 catalog 未列 → 标记 `IsListed=false`，**入库可查、前台不展示**；前台生效展示 = `IsListed && IsEnabled && !IsOrphaned` |
| ref 锁定 | 选择对话框记录 `sourceCommit`，创建 run 一并提交，Worker **checkout 到该 commit** 而非分支头；commit 不可达 → run Failed 并提示重新选择 |
| 增量选中集 | 校验对齐**该 commit 的 catalog 收录集合**（不是库快照 vs 仓库清单）；缺失项记 warning 并计入 `Failed` |
| 首次同步 | 该版本无成功 `Full` run 时拒绝 `SelectedSkills`，提示先全量（对齐用户手册「先发布一次才有可选目录」） |
| 孤儿 | key 从 catalog **整体消失** → `Full` 模式判孤儿（连带 `IsEnabled=false`）；`SelectedSkills` 模式不判 |
| 结构变化 | `SelectedSkills` 下 `LastFullSyncCatalogHash` 不一致 → Warning，继续执行 |

## 选中同步的数据源（与用户手册一致）

`SkillSyncMode.SelectedSkills` 的可选清单来自 **`catalog.json`**（同一 `sourceCommit`），与用户手册「增量发布」按当前 TOC 勾选的做法对齐。

- **目录浏览不需要 checkout**：`catalog.json` 是单文件，`git show <ref>:catalog.json` 即可出树；
- **选择树、选中集校验、实际同步三者同一真相**（同一 `sourceCommit` 的 catalog）；
- **首次同步**：该版本无成功 Full 同步时无可选项，必须先 `Full` 同步一次；
- **结构变化**：catalog 成员/分类/顺序变化后增量同步不会收录新增技能，应执行全量。

## 模块联动

- 仓库内容无状态机；库内状态变化（Add/Update/Unchanged/Orphan、收录态与孤儿翻转）见 [sync.md](../sync.md)。
- 本规范无独立 UI；同步失败清单在 WebOS 同步记录抽屉展示，错误摘要使用解析器输出的结构化原因（`CatalogIssue.Rule` + 明文 `Message`）。**报错日志不做本地化**——日志是排查产物而非界面文案，明文原因比译文更利于定位。
- 解析入口：`SkillCatalogParser.Parse(checkoutRoot)` → `SkillCatalogManifest`（catalog 解析结果）+ `ParsedSkill(Key, CategoryKey, CatalogPriority, CatalogText, SourcePath, Manifest, SkillMd, Assets, Errors)`；校验在 `SkillCatalogValidator`（纯函数），分类状态校验在 `SkillCategoryReferenceValidator`。数据写入：`Skill.Source*`、`SkillContent`（三语）、`Skill.IsListed`，资产经 Foundation 上传后只存资源 ID。

## 实施状态

同步引擎已实施 2026-10-23 基线（旧目录规范、`skill.json` 基础校验、增量哈希）；以下为 2026-10-25 修订新增、待落地：

| 项 | 落点 |
|----|------|
| 仓库布局迁移：分类目录层 + `catalog.json` + `template/` + README 一览表改生成 | `AtomUI/AtomUI.Skills` |
| `catalog.json` 解析与双向对齐校验（`Catalog*` 规则族） | `SkillCatalogParser` / `SkillCatalogValidator` |
| 布局硬约束 `LayoutDepth`（恰好两层） | `SkillCatalogValidator` |
| 分类权威校验（`CategoryNotTopLevel`、三态触发改到 catalog/目录路径） | `SkillCategoryReferenceValidator` |
| 双内容哈希（`SourceContentHash` 加自身 catalog 字段、去兄弟位置；`LastFullSyncCatalogHash`） | `SkillCatalogParser` / `SkillRepositoryVersion` |
| ref 锁定（`sourceCommit` 透传 + Worker checkout 到该 commit） | `SkillSyncExecutor` |
| 收录态 `IsListed` 与前台生效展示过滤 | `Skill` / `SkillPublicAppService` |
| `SelectedSkills` 首次门禁与结构变化告警 | `SkillSyncExecutor` / `SkillRepositoryVersion` |
| WebOS 目录浏览接口 `GetCatalogAsync`（`git show` 单文件） | `ISkillSyncAdminAppService` |
| frontmatter 解析与一致性校验 | `SkillCatalogParser` / `SkillCatalogValidator` |
| `en.summary` 必填收紧 | `SkillCatalogValidator` |
| 安全条款（symlink / 可执行 / 隐藏文件检测） | `SkillCatalogParser` |
| `skillshub.json` 读取 + 缺失警告 + `discovery` 一致性 | `SkillCatalogParser` |
| `references/` 目录解析、`.md` 白名单、纳入增量哈希 | `SkillCatalogParser` / `SkillCatalogValidator`（见[技能研发规范](./skill-authoring.md)） |
| 独立校验 CLI（供 CI 使用，与 SyncEngine 同源规则） | 新工具项目（形态见 §开放问题） |

## 测试矩阵

见 [tests.md](../tests.md) §解析器（已实施基线全规则矩阵 + 脏 JSON 降级 + 零技能致命 + 哈希稳定性）+ 待补充：布局两层约束、`catalog.json` 规则族（缺失/脏 JSON/Key 未知/重复/分类不一致/未收录 Warning）、分类权威（非顶层引用）、双哈希（兄弟位置不连坐、`text` 不入结构哈希）、ref 锁定与首次门禁、结构变化告警、frontmatter 一致性、安全条款、根清单缺失警告、`references/` 校验。

## 开放问题

- CI 校验 CLI 形态（dotnet tool / 独立脚本）与发布方式。
- `references/` 的启用时点：格式与写作规范已定（见[技能研发规范](./skill-authoring.md)），引擎支持是否随首批技能落地待定。
- 多仓库/多版本下技能归属范围：`Skill` 表当前无 `RepositoryId`/版本外键，孤儿判定按全表；同步版本 B 可能孤儿化版本 A 独有的 Key。是否补维度待定。
