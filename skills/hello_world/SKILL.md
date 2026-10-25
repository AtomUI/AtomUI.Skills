---
name: hello-world
description: Demonstrates the AtomUI Skills authoring format with one minimal working skill (SKILL.md + skill.json). Use when learning the repository layout, the key-to-name projection, or creating a first skill in AtomUI.Skills. Covers scaffolding, metadata conventions and validation.
license: MIT
---

# Hello World — 技能编写样板

本技能自身就是一个能通过全部校验的最小技能。照着它的形状写新技能。

## 何时使用

- 要在 `AtomUI.Skills` 里新建一个技能，先看目录该长什么样。
- 不确定 `skill.json.key` 与 frontmatter `name` 该怎么对应。
- 报了校验错误（key 不匹配、`en.summary` 缺失、分类不存在），要对照正确写法。
- 要确认哪些约束是机器能查的、哪些只能靠评审。

## 何时不使用

- 写 AtomUI 组件用法、设计令牌、协作流程等具体技能时，用对应的领域技能，不用本技能。
- 查询 `skill.json` 每个字段的完整语义时，查仓库内 `docs/standards/skill-metadata.md`。
- 查校验规则总表时，查仓库内 `docs/standards/skillshub-engineering.md`。

## 工作流程

1. 从 `template/` 复制一份，把目录名改成技能 `Key`。
2. 定 `Key`：`^[A-Za-z_][A-Za-z0-9_]*$`，≤64 字符，**用下划线不用短横线**（AtomIdea 标识符规范）。
   例：`avalonia_theme_tokens`，不是 `avalonia-theme-tokens`。
3. 定 `name`：由 `Key` 机器推导，**不要手写**。规则是 `ToLowerInvariant()` 后把 `_` 换成 `-`。
   `avalonia_theme_tokens` → `avalonia-theme-tokens`。
4. 填 `skill.json.category`：必须是**已在 WebOS 创建并启用**的分类 Key。留空、写错、或分类被禁用，该技能同步一律失败且报错不同。
5. 填 `languages.en.summary`：≤300 字符，讲全三要素——做什么、何时用、边界关键词。
6. 把 `en.summary` **原样**抄进 frontmatter 的 `description`。两者必须逐字一致。
7. 写正文：按下节模板填，祈使句，可枚举的写成清单或表格。
8. 自检：`key` ≡ 目录名；`name` ≡ Key 的 kebab 投影；`description` ≡ `en.summary`；`license` 两侧同填或同空。

## 示例

目录：

```text
skills/avalonia_theme_tokens/
├── SKILL.md
└── skill.json
```

`skill.json` 关键字段：

```json
{
  "key": "avalonia_theme_tokens",
  "category": "AtomUIOSS",
  "version": "1.0.0",
  "license": "MIT",
  "languages": {
    "en": {
      "name": "Avalonia Theme Tokens",
      "summary": "Applies AtomUI theme token conventions (palette, sizing, spacing, dark-mode rules) when writing or reviewing Avalonia XAML styles and ControlThemes."
    }
  }
}
```

对应 frontmatter：

```yaml
---
name: avalonia-theme-tokens
description: Applies AtomUI theme token conventions (palette, sizing, spacing, dark-mode rules) when writing or reviewing Avalonia XAML styles and ControlThemes.
license: MIT
---
```

正文章节模板（按序；小技能可合并，但「何时使用」与「示例」不得缺）：

```markdown
# <技能标题>

## 何时使用
## 何时不使用
## 工作流程
## 示例
## 边界情况
## 参考
```

## 边界情况

- **大小写不同的 `Key` 会撞同一个 `name`**：`My_Skill` 与 `my_skill` 都投影成 `my-skill`。仓库内 `name` 必须唯一，起名前先全局搜一遍。
- **`category` 是硬引用**：没有「归未分类」的降级路径。未声明、分类不存在、分类被禁用，三种情形分别报不同错误，运营要做的动作也不同。
- **`references/` 目前不要提交**：格式规范已定，但引擎侧解析与哈希纳入尚未实施。
- **正文只放 Agent 不知道的**：Markdown 语法、git 常规操作这类通用知识不要写，只写 AtomUI 特有约定、项目裁决、数值、反直觉规则。
- **超长内容先删冗余**，再考虑拆 `references/`，最后才拆技能。正文软目标 < 500 行。
- **`description` 三要素缺一不可**：只写 `Helps with theming.` 这种没有 when、没有关键词的描述，Agent 无法据此决定触发。

## 参考

正文外链仅限 `atomui.net` 与 `github.com/AtomUI*`，规范全文见本仓库 `docs/standards/`：

- [SkillsHub 工程规范](https://github.com/AtomUI/AtomUI.Skills/blob/main/docs/standards/skillshub-engineering.md)
- [技能结构与元信息](https://github.com/AtomUI/AtomUI.Skills/blob/main/docs/standards/skill-structure.md)
- [技能元信息](https://github.com/AtomUI/AtomUI.Skills/blob/main/docs/standards/skill-metadata.md)
- [技能研发规范](https://github.com/AtomUI/AtomUI.Skills/blob/main/docs/standards/skill-authoring.md)
