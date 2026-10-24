---
summary: "技能研发规范 v1.0：SKILL.md frontmatter 写作、正文结构模板、尺寸预算、references 渐进披露、语言策略、质量流程与安全红线"
read_when:
  - 为 AtomUI.Skills 仓库编写、评审或修订技能内容（SKILL.md 正文）时阅读。
title: "技能研发规范 正式标准"
---

# 技能研发规范 正式标准

> 状态：**标准 v1.0**（2026-10-24 定稿，首次发布）。SkillsHub 标准体系的内容层：[工程规范](./skillshub-engineering.md)管仓库、[结构与元信息描述](./skill-structure.md)管形态、[元信息描述](./skill-metadata.md)管 `skill.json`，本规范定义「技能内容怎么写、写多长、写到什么质量」。

## 业界基线

格式基线（Agent Skills 开放标准，四厂商收敛情况）见[技能结构与元信息描述](./skill-structure.md) §业界基线。本规范吸收的是各家的**编写方法论**：

- **Anthropic**：评测驱动编写（先跑代表性任务、针对失败点写规则）、渐进披露三档、只写模型不知道的上下文、互斥场景拆开。
- **OpenAI**：技能指令按用户级输入生效——正文措辞按「给执行者的任务说明」写，不按「系统设定」写。
- **OpenClaw**：发布时校验「声明的元数据 vs 实际行为」一致性——description 声明的触发场景必须与正文内容相符（我们无脚本，故不需要其 `requires` 体系）。

基线字段（`name` / `description` / `license` / `compatibility` / `metadata` / `allowed-tools`）语义全部照搬开放标准；我们未用到的扩展（`allowed-tools`、`compatibility`）按标准保留透传，不禁止。

## 设计原则

1. **开放标准为底**：任何本规范未规定的细节，以 [agentskills.io/specification](https://agentskills.io/specification) 为准；本规范只做收紧和本地化，不发明冲突格式。
2. **知识型技能定位**：AtomUI.Skills 的技能是「程序性知识」（约定、规范、工作流），不是「工具型技能」——仓库禁脚本与可执行文件（见工程规范安全条款），故不设 `scripts/`；`allowed-tools` / `compatibility` 通常无需填写。
3. **渐进式披露**（业界三档）：frontmatter（常驻上下文，~100 token）→ SKILL.md 正文（激活时加载）→ `references/`（按需加载，本规范引入、引擎待支持，实施前生效两档）。
4. **单一权威不破坏**：frontmatter 是 `skill.json` 的投影（规则见[技能元信息描述](./skill-metadata.md) §frontmatter 投影关系）；本规范管的是**怎么写好投影源和正文**。
5. **质量先于数量**：每个技能必须对应真实任务场景与失败案例（Anthropic 方法论：先评测后编写）。

## Frontmatter 写作规范

字段定义、必填性与投影校验见[技能结构与元信息描述](./skill-structure.md)；此处只约束**写法**：

| 字段 | 写作规则 | 示例 |
|------|----------|------|
| `name` | = Key 的 kebab 投影（机器生成，不手写）。命名从**任务视角**出发：动词或动名词开头（`review-avalonia-style`、`migrate-theme-tokens`），避免 `helper`/`utils`/`tool` 这类无信息词 | `avalonia-theme-tokens` |
| `description` | 英文、第三人称、一句话讲全**三要素**：①做什么（what）②何时用（when，触发场景/用户意图关键词）③边界关键词（帮助 Agent 检索命中）。≤300（我们的收紧，天然满足业界 1024） | `Applies AtomUI theme token conventions (palette, sizing, spacing, dark-mode rules) when writing or reviewing Avalonia XAML styles and ControlThemes. Use for theming, token lookups, or dark-mode questions.` |
| `license` | 与仓库级一致（`MIT`）；不逐技能改 | `MIT` |
| `compatibility` | 仅当有真实环境前提才填（如「Requires .NET 8 SDK」）；知识型技能通常不填 | — |
| `allowed-tools` | 不填（无脚本、无工具预授权诉求） | — |
| `metadata` | 不填；扩展元信息一律放 `skill.json`（三语权威），避免双源 | — |

description 反例（业界通病）：`Helps with theming.`——没有 when、没有关键词，Agent 无法据此决定触发。

## 正文结构规范

正文 = frontmatter 之后的 Markdown，写给 Agent 读（不是写给人读的文档）。**推荐章节模板**（按序；小技能可合并，但不允许缺「何时使用」与「示例」）：

```markdown
# <技能标题>

## 何时使用
触发场景清单（用户意图、任务类型）；一句话写清。

## 何时不使用
反例清单（相邻技能的边界，指名让给谁：『仅查 Lumi 组件尺寸时用 sizing-scale，不用本技能』）。

## 工作流程
1. 步骤化指令，祈使句，一步一件事。
2. 需要判断的写判断规则，不写「视情况而定」。

## 示例
- 输入 → 期望输出（至少 1 组；XAML/代码示例用围栏代码块标语言）。

## 边界情况
- 已知的坑、例外规则、失败模式。

## 参考
- 官方文档链接（限 atomui.net / AtomUI GitHub 域名）；长文拆到 references/ 的在此引用相对路径。
```

写作规则（违反即评审打回）：

1. **只写 Agent 不知道的**：模型已知通用知识（Markdown 语法、git 常规操作）不写；只写 AtomUI 特有约定、项目裁决、数值、反直觉规则。
2. **一技能一职责**：两个场景几乎不同时触发 → 拆成两个技能；互斥场景不得塞进同一正文（Anthropic：rarely co-used contexts 拆开省上下文）。
3. **祈使句、正例优先**：写「使用 X」而不是「你应该可能会考虑使用 X」；禁止角色扮演开头（「你是一名资深设计师…」）。
4. **确定性优先**：可枚举就枚举（数值、清单、表格），不写形容词（「适当的间距」「优雅的处理」）。
5. **代码/路径/命令一律围栏或反引号**，并标注语言。
6. **不引用仓库外文件**：正文只引用本技能目录内文件与官方域名链接。

## 尺寸预算

| 层 | 硬上限（引擎校验） | 软目标（评审标准） |
|----|--------------------|---------------------|
| frontmatter `description` | ≤300 字符 | 80–300，三要素齐 |
| SKILL.md 正文 | ≤256KB | **< 500 行 / 约 5000 token**（业界建议值） |
| `references/` 单文件 | —（待实施时定） | 聚焦单一主题，每个 < 500 行；引用深度一层（SKILL.md → references/x.md，不再嵌套） |

超软目标的处理顺序：先删冗余（规则 1）→ 再拆 `references/`（等引擎支持，见工程规范 §实施状态）→ 最后才考虑拆技能。

## `references/` 目录（本规范引入，引擎待支持）

用途：承接超长参考内容（完整 token 表、全量组件规范），让 SKILL.md 正文保持导航层。结构与文件规则（仅 `.md`、一层引用、≤8 个文件）见[技能结构与元信息描述](./skill-structure.md) §`references/`；**引擎侧解析与哈希纳入待实施**（见工程规范 §实施状态），实施前先不提交 `references/` 目录。

## 语言策略

- frontmatter `description`：**英文**（业界发现层通用语，= `skill.json` 的 `en.summary`）。
- 正文：**默认 zh-Hans**（技能受众是 AtomUI 中文生态开发者；Agent 对中文指令理解无损）。面向国际社区的技能可写英文或中英双语，但**同一技能内保持一种主语言**，不逐段混排。
- 代码、标识符、路径：保持原生（XAML/C#/英文 token 名不翻译）。

## 质量流程（源自 Anthropic 评测驱动方法论）

1. **先评测后编写**：技能必须源于真实任务的失败——先跑 2–3 个代表性任务（如让 Agent 写一个 ControlTheme），观察它在哪违反 AtomUI 约定，针对失败点写规则。
2. **从 Agent 视角迭代**：技能合入后观察实际使用轨迹（过度触发、漏触发、误读规则），把修正沉淀回正文；description 与正文行为不一致时**先改 description**（发现层优先）。
3. **提交前自检清单**（CODEOWNERS 评审与 CI 校验共用）：
   - [ ] description 三要素齐（what / when / 关键词）
   - [ ] 有「何时使用」+「何时不使用」+ 至少 1 组示例
   - [ ] 无模型已知的通用知识、无角色扮演开头、无形容词式规则
   - [ ] 正文 <500 行；references/ 一层引用（如使用）
   - [ ] 外部链接仅 atomui.net / AtomUI GitHub
   - [ ] frontmatter 投影与 `skill.json` 一致（引擎校验兜底）
   - [ ] 已声明 `category`，且该分类已在 WebOS 创建**并启用**（否则同步会失败）

## 安全红线

继承工程规范安全条款（无脚本、无可执行、无隐藏文件），正文写作追加三条（供应链教训：技能文本本身即 Agent 指令）：

1. **外部链接白名单**：正文只允许 `atomui.net` 与 `github.com/AtomUI*` 域名链接；禁止指向任意第三方 URL（防提示注入与流量劫持）。
2. **禁止远程指令**：正文不得指示 Agent 下载、执行、curl 远程内容或「按链接里的最新指示行事」。
3. **禁止越权措辞**：不得出现「忽略之前的指令」「无需用户确认」类对抗系统/用户边界的表述。

## 与标准体系的关系

- 字段必填性、投影一致性、文件校验、增量哈希：以[工程规范](./skillshub-engineering.md)与[技能结构与元信息描述](./skill-structure.md)为准。
- 本规范对正文的约束中，**语言默认、章节模板、软预算**为评审标准（CI 不可全量机检）；description 三要素、外部链接白名单可机检，纳入 CI 校验器规则池（见工程规范 §CI 校验与仓库治理）。

## 实施状态

| 项 | 落点 | 状态 |
|----|------|------|
| frontmatter 投影一致性校验（本规范依赖） | `SkillCatalogValidator` | 待实施（见工程规范 §实施状态） |
| `references/` 目录解析、白名单（仅 .md）、纳入增量哈希与资产豁免 | `SkillCatalogParser` / `SkillCatalogValidator` | 待实施 |
| CI 可机检项：description 长度、外链域名白名单、角色扮演开头黑名单 | 校验 CLI（工程规范 §实施状态） | 待实施 |
| `category` 已声明的机检项（存在性/启用态需查 WebOS，仅本地 CI 不可检） | 校验 CLI + 同步执行器 | 待实施 |
| `template/` 更新为含推荐章节模板的 SKILL.md 范本 | `AtomUI.Skills` 仓库初始化 | 待实施 |

## 测试矩阵

待补充（并入 [tests.md](../tests.md) §解析器）：references/ 仅 .md 校验、references 计入哈希、外链域名白名单命中/放行各一例。

## 开放问题

- 正文主语言是否强制 zh-Hans（当前：默认 + 允许例外）。
- `references/` 是否随首批技能启用（当前：规范就绪、引擎待实施；若首批无超长技能可推迟）。
- description 关键词命中率的自动化评测（Anthropic 式 LLM-judge）是否引入 CI。
