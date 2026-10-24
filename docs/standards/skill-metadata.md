---
summary: "技能元信息描述 v1.0：skill.json schema、字段语义与约束、三语与回退规则、frontmatter 投影源、分类硬引用三态与删除守卫"
read_when:
  - 编写、评审或校验技能的 skill.json（官网展示元信息）时阅读。
title: "技能元信息描述 正式标准"
---

# 技能元信息描述 正式标准

> 状态：**标准 v1.0**（2026-10-24 定稿，首次发布）。与[SkillsHub 工程规范](./skillshub-engineering.md) 配套。`skill.json` 是官网展示元信息的**唯一权威**（三语、分类、标签、平台），同时是 SKILL.md frontmatter 的**投影源**；目录与 frontmatter 结构见[技能结构与元信息描述](./skill-structure.md)。

## Schema

```json
{
  "key": "avalonia_theme_tokens",
  "category": "design_system",
  "version": "1.0.0",
  "author": "AtomUI",
  "license": "MIT",
  "repositoryUrl": "https://github.com/AtomUI/AtomUI.Skills/tree/main/skills/avalonia_theme_tokens",
  "tags": ["theme", "tokens"],
  "platforms": ["Windows", "macOS", "Linux"],
  "agents": ["claude-code", "codex", "cursor"],
  "priority": 1000,
  "languages": {
    "zh-Hans": { "name": "…", "summary": "…", "description": "Markdown" },
    "zh-Hant": { "name": "…", "summary": "…", "description": "Markdown" },
    "en":     { "name": "…", "summary": "…", "description": "Markdown" }
  }
}
```

## 字段语义与约束

| 字段 | 必填 | 约束与语义 |
|------|------|------------|
| `key` | ✓ | `^[A-Za-z_][A-Za-z0-9_]*$` ≤64；与目录名一致；仓库内唯一 |
| `category` | ✓ | 引用 WebOS 分类 Key（同标识符规范）。**硬引用**：必须已声明、已存在、且启用，否则该技能同步失败并记错误日志（三态见下）。分类树在 WebOS 维护，仓库不维护 |
| `version` | — | 展示用；建议 semver；内容变更须递增（软约定，CI 可检查） |
| `author` | — | 展示用 |
| `license` | — | 展示用；与仓库级许可对应；是 frontmatter `license` 的投影源 |
| `repositoryUrl` | — | ≤512；指向该技能目录的 GitHub 地址 |
| `tags` | — | ≤16 项，去重；辅助检索 |
| `platforms` | — | 值域 `Windows/macOS/Linux`（`AiSkillPlatform` 枚举），≤16 去重 |
| `agents` | — | 自由字符串 ≤16 项（展示客户端能力，不校验值域） |
| `priority` | — | 整数，缺省 1000；仅作来源排序，运营可在 WebOS 覆盖 |
| `languages.en` | ✓ | 回退源；`en.summary` **必填**（作为 frontmatter `description` 的投影源） |
| `languages.zh-Hans` / `zh-Hant` | — | 缺失时由 `en` 生成回退内容（`IsFallback=true`）并记「待翻译」 |
| `languages.*.name` | ✓ | 非空 ≤128；官网展示名 |
| `languages.*.summary` | `en` 必填；其余可选 | ≤300；`en.summary` 兼任 frontmatter `description` |
| `languages.*.description` | — | Markdown ≤64KB；官网详情页主文 |

## `category` 引用三态（均为技能级失败）

分类是**硬引用**，不存在「归未分类」的降级路径。三种情形分别产出不同的 `CatalogRule`，让运营一眼看出该做什么：

| 情形 | `CatalogRule` | 报错要点 | 运营动作 |
|------|---------------|----------|----------|
| 未声明 `category` | `CategoryRequired` | 未声明分类 | 补 `skill.json.category` |
| 声明了但 WebOS 无此分类 | `CategoryUnknown` | 分类 `<key>` 不存在 | 在 WebOS 创建该分类 |
| 声明了但分类被禁用 | `CategoryDisabled` | 分类 `<key>` 已被禁用 | 在 WebOS 启用该分类 |

失败的技能不落库、计入 `FailedKeys`、逐条记错误日志，且**绝不判孤儿**。注意这是**状态校验**（需查 WebOS 分类表），不在纯函数校验器内，见[工程规范](./skillshub-engineering.md) §校验分层。

## 分类删除守卫

分类被删会让已落库技能失去分类，下次同步即触发 `CategoryRequired`/`CategoryUnknown` 失败。因此：

- **有任意技能引用（含已停用技能）的分类不可删除**，抛 `010004`（`Categories.HasSkills`）。
- 引用计及**生效值** `CategoryId = OverrideCategoryId ?? SourceCategoryId`，运营改过的引用同样受保护。
- 需要下线分类时应**禁用**而非删除；禁用会让引用它的技能同步失败并明确提示「分类已被禁用」，比静默归未分类更可控。

## 三语与回退规则

- 站点语言集固定 `zh-Hans` / `zh-Hant` / `en` 三语，与站点本地化体系一致。
- `en` 为回退源（必填）：任一语言缺失某文案时，用 `en` 内容生成回退行并标注 `IsFallback`，官网显示「待翻译」标记。
- `zh-Hans` 是站点第一语言，官网展示优先按访客语言取；不存在「SKILL.md 正文按语言切换」——正文语言策略见[技能研发规范](./skill-authoring.md)。

## frontmatter 投影关系

`skill.json` 是权威，SKILL.md frontmatter 是投影（校验强制一致，见工程规范 §校验规则总表）：

| frontmatter 字段 | 投影源 | 规则 |
|------------------|--------|------|
| `name` | `key` | Key 的 kebab-lowercase 投影（见[技能结构与元信息描述](./skill-structure.md)） |
| `description` | `languages.en.summary` | 逐字一致 |
| `license` | `license` | 两侧同填或同空 |

投影方向单向（`skill.json` → frontmatter），禁止反向手改 frontmatter。

## 与运营覆盖的关系

`skill.json` 落库到 `Skill.Source*` / `SkillContent`（来源字段）；WebOS 运营覆盖（`Override*`：启用/排序/分类/封面）不写回仓库，生效值 = 覆盖 ?? 来源——仓库权威与运营灵活性分层，见 [overview](../overview.md) §核心概念。

## 实施状态与校验

字段级校验（长度、值域、去重、`en.summary` 必填）在 `SkillCatalogValidator`（纯函数，结构校验）；`en.summary` 必填为本标准新增收紧（待实施，见工程规范 §实施状态）。`category` 三态校验属**状态校验**，在同步执行器内完成（见工程规范 §校验分层），已实施。删除守卫已收紧为「含停用技能一并保护」，并覆盖来源引用与运营覆盖两种非生效引用。其余基线校验已实施。

## 开放问题

- 详情页主文以 `languages.*.description` 为准、`SKILL.md` 折叠原文展示（当前设计，既有遗留）。
- frontmatter `description` 复用 `en.summary`：官网摘要与「给 Agent 的触发说明」当前合一；若未来需要分化（摘要偏营销、description 偏触发时机），再引入独立投影规则。
