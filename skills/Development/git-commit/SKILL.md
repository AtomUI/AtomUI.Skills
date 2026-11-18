---
name: git-commit
description: Drafts Git commit messages, checks the staged changes, and creates and verifies local commits when explicitly authorized. Use for “msg”, “commit msg”, “写提交信息”, “提交前检查”, “提交”, or “创建 commit” in a Git context.
license: MIT
---

# Git 提交

按用户授权确定范围，从实际差异提炼提交意图。在创建提交前核对暂存内容并完成要求的检查，提交后核对实际结果。

所有模式都检查完整消息。标题、正文或尾注只要含 `Co-Authored-By` 就拒绝，匹配忽略大小写，用户提供的草稿和仓库模板同样适用。

## 何时使用

| 请求 | 交付结果 |
| --- | --- |
| `msg`、`commit msg`、写提交信息、只要提交标题 | 根据实际差异返回一条可粘贴的纯文本消息。用户只要标题时返回一行。 |
| 检查提交、提交前检查 | 报告待提交范围、检查结果和阻塞项。保持暂存区和历史原样。 |
| `git commit`、提交、创建提交、创建 commit | 用户明确提出执行请求时，检查并提交该授权范围，随后核验结果。 |

把这些短语放在 Git 上下文中解释。「准备好提交信息」「看看能不能提交」分别对应消息和检查模式。

## 何时不使用

已有提交的代码审查、发布说明、分支或 worktree 管理、推送及改写历史属于独立任务。

## 从实际差异取证

先确认用户指的是哪个仓库，从仓库根目录读取状态、暂存清单和实际差异。

```bash
git rev-parse --show-toplevel
git --no-optional-locks status --short --branch
git diff --cached --stat
git diff --cached --name-status --find-renames
git --no-pager diff --cached --no-ext-diff --no-textconv
```

- 默认只读暂存区。同一文件同时有暂存和未暂存改动时，以 `git diff --cached` 的内容为准。用户指定文件或提交区间时按对应范围取证。用户明确要求纳入工作区改动时，再读这些文件的未暂存差异和范围内的未跟踪内容。
- `--stat` 和文件名帮助导航。实际 diff 决定消息内容。差异太大时，按模块分批读完，核对主要行为、配套测试和文档、生成文件，以及删除或改名的公开接口。二进制和子模块至少核对可确认的资源类型、旧新版本或提交指针。看不到内部变化时，只描述已证实的更新。
- 用户提供 diff 时按该 diff 起草。仓库里没有暂存改动且用户未指定其他输入时，回答「当前没有已暂存的改动。」缺少决定标题或迁移说明的关键信息时，指出具体缺口。

## 判断仓库风格

1. 先读目标仓库适用的提交指南、`AGENTS.md`、提交模板和校验配置，确认类型、scope、语种、长度及尾注要求。仓库有固定格式时按其格式起草，并遵守本技能的消息禁令。
2. 用 `git --no-pager log --no-merges -10 --format=%s` 看近期提交。历史混用风格时，优先比较同模块、同类变更的重复模式。从历史提取类型、模块拼写、scope 大小写及可选性。单次超长标题、自动化提交或归因尾注都不作为风格依据。需要核对正文或尾注时，只读少数可比提交的完整消息。
3. 提交标题和正文默认都用英文。对话语言和历史提交语言不改变这个默认值。用户明确指定其他语言且仓库规则允许时，按用户指定写。仓库有明文语种要求时，遵循该要求。`type` 保留标准英文关键字，`scope` 保留仓库标识符的原拼写。
4. 没有明确的格式规则或稳定惯例时，采用下文的 11 个默认类型。

## 提炼提交意图

1. 从每组差异找出使用者能观察到的变化，或维护者要追溯的工程目的。功能修复连同对应测试、示例和说明采用同一主类型。测到性能提升时才写数值。此前行为、缺陷触发条件或根因尚未证实时，标题只写能从差异确认的变化。
2. 跨层变更如果共同实现同一个行为，并且分开后无法独立生效，用一个功能 scope。标题写最终效果，正文按需解释层间关系。能独立理解、验证和回滚的变更可以分别起草，仓库的原子提交规则优先。拆分或重新暂存要求用户授权。
3. 用户要求一条消息覆盖所有暂存内容时，标题优先概括影响最大的实际改动，正文补齐相互独立的实质改动。有共同目的时用共同标题。只要一行时，用可以从 diff 证实的概括，确认所有重要改动都被覆盖。避免沿文件名顺序串成清单。

## 提交前检查

| 顺序 | 核对内容 | 阻塞时的处理 |
| --- | --- | --- |
| 1. 提交范围 | 逐项核对暂存内容是否在授权内。对局部暂存的文件同时看暂存与未暂存差异，确认分界。 | 暂存区为空或混入未经授权的改动时，报告具体文件和范围。只有用户明确授权暂存指定内容时，才调整该范围。 |
| 2. 仓库状态 | 读 `git status` 确认分支及正在进行的操作。`git ls-files --unmerged` 有输出代表存在未解决的冲突。 | 有冲突时报告路径。发现合并、变基、拣选或回退时，只有用户明确授权继续该操作，才按其对应流程处理。 |
| 3. 差异完整性 | 运行 `git diff --cached --check`。对照实际 diff，检查误纳入的生成文件、调试改动，以及缺失的关联修改。 | 报告具体文件、行号或缺失项。变更暂存范围后重新核对。 |
| 4. 验证结果 | 按仓库约定和改动风险运行相关测试、lint 或构建。可复用与当前代码、依赖和配置一致的已有结果。 | 报告失败命令和原因。若相关文件有未暂存改动，工作区测试只证明工作区结果。仓库要求验证暂存版本时，使用仓库已有的暂存版本验证入口，或报告缺少该版本的证据并暂停提交。 |
| 5. 消息 | 有候选消息时核对完整标题、正文和尾注，检查格式与内容，再按仓库的钩子规则执行。 | 包含禁用字段或与实际改动不符时指出问题。检查模式输出范围、已通过项、未执行项和阻塞项。 |

## 创建并核验提交

1. 有明确的提交授权后，完成范围与冲突检查，记录当前分支、`HEAD`（首次提交记为无）和 `git write-tree` 得到的暂存树，再执行其余验证。提交前立即复核这三项。任一项改变时，重新确认授权范围并重跑受影响的检查。
2. 所需检查通过后，用 `git commit -F <消息文件>` 提交当前暂存树。将已校验的完整消息原样写入消息文件，保留仓库的作者身份、签名和钩子设置。用户授权特定的 Git 操作时沿用其正常入口与要求。
3. 根据最新 `HEAD` 和索引核对执行结果。即使命令返回非零，也先确认 `HEAD` 是否已经推进。若提交已经创建，核验它并报告执行异常。若分支、`HEAD` 和暂存树保持原样，可处理失败原因后复核再试。
4. 读取新提交的完整消息、文件范围和树，与已检查的结果比较。普通提交同时核对父提交是否为记录的原 `HEAD`。报告短 SHA、标题和实际验证结果。发现差异时如实说明，改写历史需要独立授权。

## 默认类型白名单

通用格式采用 Conventional Commits 1.0.0。它定义 `feat`、`fix` 及消息结构。此处从 commitlint 的常用预设取 11 个小写类型。

| 类型 | 英文含义 | 适用边界 |
| --- | --- | --- |
| `feat` | Feature，新功能 | 新增用户可用的能力、公开 API 或行为。恢复原有预期行为用 `fix`。 |
| `fix` | Bug fix，缺陷修复 | 修正错误结果、崩溃、回归或不符合预期的交互和显示。 |
| `docs` | Documentation，文档 | 仅修改文档、说明或注释。随功能一起更新文档时，使用该功能的主类型。 |
| `style` | Code formatting，代码格式 | 空格、缩进或换行等纯格式调整，代码含义保持不变。实际改变界面表现时，按新增能力或缺陷修复分类。 |
| `refactor` | Refactoring，重构 | 拆分、重命名或移动内部实现，对外行为保持不变。 |
| `perf` | Performance，性能 | 主要目的为降低延迟、资源占用或重复计算。涉及具体提升幅度时，以已提供的测量证据为准。 |
| `test` | Tests，测试 | 仅新增或调整测试、夹具及测试基础设施。随修复加入的回归测试归该修复。 |
| `build` | Build system，构建与依赖 | 修改编译、打包、构建工具或外部依赖。依赖版本升级默认使用 `build(deps)`。 |
| `ci` | Continuous Integration，持续集成 | 修改 CI 工作流、流水线环境或专用脚本。影响项目构建方式的改动归 `build`。 |
| `chore` | Maintenance，例行维护 | 清理仓库管理文件、编辑器设置等辅助配置。已有更具体类型时优先使用具体类型。 |
| `revert` | Revert，撤销提交 | 有意撤销已存在的提交，正文记录已核实的原提交哈希。 |

`deps` 默认作为 scope。`release`、`gallery` 等独立类型仅由目标仓库的明确约定启用。

### 容易混淆的类型

| 比较 | 从 diff 中确认什么 |
| --- | --- |
| `feat` / `fix` | 恢复现有契约用 `fix`，例如恢复快捷键在既有模式下的行为。扩展能力用 `feat`，例如允许用户配置新的快捷键。 |
| `fix` / `perf` | 同时修正错误结果和减少计算时，判断改动的主目标。纯性能优化可用 `perf`，即使没有基准结果也可以写明减少了哪种已确认的工作。 |
| `refactor` / `fix` | 提取代码、调整分层、重命名之后对外行为相同，使用 `refactor`。同时改变用户行为时，按该行为的目的归类。 |
| `style` / `fix` | 仅格式化样式文件属于 `style`。修正焦点环位置、层叠顺序或布局属于行为修复。 |
| `docs` / `test` | 只改示例文档或源码注释用 `docs`。改变测试执行、断言或夹具用 `test`。如果它们是产品改动的配套项，随主类型归类。 |
| `build` / `ci` / `chore` | 编译产物、锁文件、依赖解析或打包方式变更归 `build`。流水线触发、执行矩阵或 CI 专用脚本归 `ci`。仅用于维护代码库的工具配置归 `chore`。 |
| `revert` / `fix` | `revert` 对应明确撤销的已有提交，须核实原提交标识。恢复旧效果但采用新的修复机制时，按实际行为修正使用 `fix`。 |

同一改动的测试、接口、文档和生成文件可以分布在许多目录。以它们共同交付的行为定类型，文件数和扩展名只用于核对覆盖范围。

## Scope 怎么选

| 差异的归属 | scope 写法 |
| --- | --- |
| 一个控件、子系统或外部服务 | 使用读者能在仓库历史中找到的最窄名称，如 `Select` 或 `auth`。实现横跨 UI、服务与测试时，仍可用所属功能的名称。 |
| 多个消费者共享的行为或实现 | 使用共同模块或职责，如 `layout` 或 `cache`。并列 scope 仅在仓库有这种写法且范围确实有直接关联时使用。 |
| 横跨项目的约定或维护 | 用有区分力的现有 scope，如 `deps`、`packaging`。找不到明确归属时省略 scope。 |

常见归类：

- `Select` 的修复同时涉及基类、桌面控件与测试，采用历史中的控件名 `Select`。
- 多个控件都受同一底层弹层定位算法影响，使用历史中的 `overlay` 或 `Layout`。只列相关控件名会遗漏实际的共享范围。
- 修复查询接口和前端过滤项之间的契约，只要它们独立发布会产生错误，合在同一消息，以 `search` 或仓库已有的功能 scope 说明结果。
- 两组改动能单独构建、验证和回滚时，分别拟消息。用户要求保留一笔时，可用主改动写标题，用正文诚实列出第二项。

## 标题

```text
<type>[(<scope>)][!]: <subject>
```

- 标题只占一行。使用半角标点，冒号后留一个空格。scope 可省略，填写时采用仓库稳定的模块名或领域名。破坏性变更的 `!` 紧贴冒号。
- subject 用一个短语交代动作、对象和实际变化。英文用动词原形开头，词首通常小写，保留专名和标识符原有拼写。中文以动词开头。标题末尾省略句号。
- 默认整行含前缀以 50 字符为目标，最多 72 字符。保留最能区分这次改动的事实，把原因、影响和取舍移到正文。仓库明确规定其他长度时，采用该限制。
- 用关键词定位行为表面和触发条件。`fix(Select): keep the menu open after selecting a value` 比 `fix(Select): update selection logic` 更易检索。标题保持一个清晰意图。遇到多个条件、实现细节或指标时，优先保留最能区分本次改动的条件。

| 初稿 | 可直接使用的标题 | 修改理由 |
| --- | --- | --- |
| `fix(auth): fix bug` | `fix(auth): reject expired refresh tokens` | 写明被拒绝的输入。 |
| `refactor: update classes` | `refactor(storage): share record encoding` | 用职责描述跨文件变动。 |
| `feat(reports): CSV updates` | `feat(reports): export filtered results as CSV` | 用动词开头，写清新增能力。 |
| `fix(Select): 修了下拉框` | `fix(Select): 保持多选下拉框在选值后展开` | 用户明确要求中文时，写明触发条件与结果。 |

复核标题时，删除 `WIP`、`misc`、`update files` 等占位词。避免在同一行附带阶段编号、括号解释或测试统计，仓库明确要求的编号格式除外。英文动词用祈使式，例如 `add`、`prevent`、`preserve`。整行要能单独出现在 `git log --oneline` 中。

## 正文

- 标题已经表达完整意图时省略正文。需要补充时，在标题后空一行。使用短段落或最多 5 条 `- ` 项，普通文字默认每行最多 72 字符。
- 写动机、已知根因、影响范围、关键取舍或迁移背景。独立而实质的次要改动在正文中交代。文件清单留在 diff。测试结果、性能指标及根因只写已得到确认的事实。
- 复杂修复可按「触发条件与原行为 → 已验证的根因 → 修复后的影响」组织。把关键实现写进正文的前提是它能解释范围、兼容性或回滚影响。选取有区分力的一到两项，普通修复仍可只用标题。
- 撤销提交时用 `This reverts commit <hash>.` 记录核实过的哈希。缺少撤销目标时，先明确需要核实的原提交。

正文按最有用的信息选材：

| 情形 | 正文应记录 |
| --- | --- |
| 缺陷只在特定状态出现 | 触发条件和可验证的原行为。根因已确认时说明哪段假设失效。 |
| 跨层迁移或兼容修复 | 哪些层共用规则、哪些入口受影响、用户升级是否要额外操作。 |
| 性能调整 | 已减少的计算、分配或 I/O。写数值时同时写清测量范围和条件。 |
| 增加验证、测试或日志 | 只有这些改动影响复现、排查或保障范围时才写细节。 |
| 一笔提交含两组独立改动 | 标题写主改动，正文单独交代另一项的实际目的。 |

## 尾注

- 尾注与正文之间空一行。没有正文时，与标题之间空一行。每条使用 `Key: value`，同一组尾注连续排列，续行缩进一个空格。
- 公开 API、协议、配置或使用方式的真实不兼容变更默认同时在标题加 `!`，并添加大写 `BREAKING CHANGE:` 尾注。写清受影响的旧用法和可以执行的迁移步骤。内部重命名或重排保持兼容时沿用常规格式。
- 已核实的关联使用 `Refs: #123`，明确应关闭的事项使用 `Fixes: #123`。签署尾注遵循仓库规定和经确认的身份。禁止 `Co-Authored-By` 尾注，无论归因对象是谁。

同一编号在多个仓库出现时，确认事项所属仓库，并按项目约定填写完整引用。消息最后逐字检查整块尾注。

## 特殊提交

| 场景 | 处理 |
| --- | --- |
| 撤销 | 从 Git 或用户提供的可信记录确认被撤销的提交。单次撤销使用 `revert`，正文先写原哈希，已确认时补充撤销理由。多笔撤销逐笔核对。 |
| 发版 | 有 `release` 约定的目标仓库按其格式。通用格式可用 `chore(release): prepare v2.4.0` 表达版本元信息更新。发版动作与创建普通提交分别授权。 |
| 合并 | 只在用户授权相应操作时处理。项目接受 Git 默认合并消息时沿用，仍需通过完整消息检查。核验实际合并结果时以预期父提交集合为准。 |
| `fixup!` / `squash!` | 用户明确要求且仓库采用该流程时，先核实目标提交。普通提交用常规格式。后续历史改写须有独立授权。 |

## 交付前复核

- 消息模式检查语种、格式、长度、标题主旨、重要变更和尾注事实。通过时只返回可粘贴的消息文本，首行直接写标题。用户只要标题时，返回单行。
- 检查模式列出待提交范围、实际运行的检查、结果和阻塞项。没有候选消息时可按需起草，再校验计划提交的那一版。

## 示例

### 11 种类型的常见输入

| 输入差异 | 输出 |
| --- | --- |
| 新增报表按视图筛选后导出 CSV 的能力 | `feat(reports): export filtered results as CSV` |
| 服务拒绝超时输入时丢失结构化错误码，新增测试锁定原契约 | `fix(api): preserve error codes on timeout` |
| 只补充部署说明里的端口和配置变量 | `docs(deploy): document the health check port` |
| 只格式化样式源码，输出样式保持一致 | `style: normalize stylesheet indentation` |
| 提取存储层公共编码逻辑，调用和数据格式保持兼容 | `refactor(storage): share record encoding` |
| 用缓存避免同一布局约束下重复计算路径，未提供基准数据 | `perf(layout): cache paths for repeated constraints` |
| 实现没变，只补了分页在空响应时的断言 | `test(pagination): cover empty responses` |
| 修改安装包的目标架构或新增构建产物 | `build(packaging): include arm64 in the installer` |
| 依赖版本不变，只调整 CI 的运行矩阵 | `ci: run smoke tests on macOS` |
| 删除已废弃的编辑器设置，无运行时改动 | `chore(editor): remove obsolete workspace settings` |
| 已确认撤销「启用报表预览」的提交 | `revert(reports): disable report previews`，正文记原提交 |

### 仅按暂存区生成消息

用户在中文对话中发 `msg`，未指定消息语言。近期提交用中文，仓库没有明文语种要求。暂存区修正刷新令牌过期后的登录行为，并新增对应的回归测试。

```text
fix(auth): reject expired refresh tokens
```

暂存区只升级依赖，登录逻辑另有未暂存的改动。以暂存差异中的依赖名和版本生成消息。

```text
build(deps): bump vitest to 3.2.1
```

同一个配置文件的暂存差异只把 Linux 包压缩格式切换为 gzip，未暂存差异还增加了 macOS 打包支持。

```text
build(packaging): switch Linux archives to gzip
```

### 跨层相关改动与独立改动

项目历史用 PascalCase scope。数据层、组件和回归测试共同修复搜索框清空后残留旧请求结果。

```text
fix(SearchBox): discard stale results after clearing input
```

共享弹层定位修复覆盖两个控件。测试证实锚点下方不足一行菜单高度时弹层本应向上打开，定位逻辑此前使用了整个滚动区域的高度。

```text
fix(overlay): open menus upward near the viewport bottom

Measure available space inside the scroll viewport so dropdowns
can open above the anchor when there is too little space below.
```

服务层和 UI 必须同时调整配置迁移。另有一项独立的 Windows 打包修正。用户要一条消息覆盖所有差异。

```text
fix(settings): preserve filters during workspace migration

- Restore persisted filters before initializing the search view.
- Include the missing license file in the Windows installer.
```

用户改为要求分别拟两条消息。迁移本身合在同一条，打包修正单独拟一条：

```text
fix(settings): preserve filters during workspace migration
```

```text
build(packaging): include the Windows installer license
```

### 根因、性能事实和尾注

用户明确要求中文，仓库允许，并使用 PascalCase scope。已证实焦点丢失由重复注册滚动监听器造成，修正后切换标签保留焦点。

```text
fix(TabView): 切换标签后保留列表焦点

切换标签时重复注册滚动监听器，旧监听器会覆盖新标签的焦点位置。
复用当前标签的监听器，保留还在可见范围内的焦点项。
```

优化界面测量。diff 证明缓存避免重复读取布局约束，用户未提供基准测试数据。

```text
perf(layout): reuse constraints across repeated measurements
```

另一个差异消除空闲场景中的轮询。用户提供了同一机器和配置下的基准数据，十分钟空闲会话的请求数从 600 降到 0。标题写优化方式，正文记录测量条件。

```text
perf(sync): stop polling during idle sessions

On the same local machine and configuration, an idle 10-minute
session makes zero requests instead of 600.
```

暂存区修复空结果时丢失分页游标，已确认客户端依靠游标继续翻页，且关联事项应随本提交关闭。

```text
fix(pagination): preserve cursors for empty pages

Clients still need the next cursor when filters remove every item
from a page.

Fixes: #184
```

公共导出 API 移除同步端点并提供异步任务，调用方需要迁移。

```text
feat(api)!: replace synchronous exports with jobs

Move export processing to the job queue so large exports can
continue after the initial request returns.

BREAKING CHANGE: Replace GET /export with POST /exports and poll
 the returned job URL for completion.
```

已核实撤销的原提交为 `a12b34c56d7890e12f34a56b7890c12d34e56f78`。项目记录原哈希，且已确认报告页面在刷新时重复启动预览。

```text
revert(reports): disable report previews

This reverts commit a12b34c56d7890e12f34a56b7890c12d34e56f78.
Refreshing the report page started duplicate previews.
```

### 从 AtomUI 变更提炼提交消息

AtomUI 更新 `QRCodeTheme.axaml`，让不透明掩码跟随外框圆角裁剪，并给 Loading、Expired 和 Scanned 三种状态补了回归测试。这些测试和变更日志服务于同一项渲染修复。

```text
fix(Controls): preserve rounded corners in masked QR codes

Clip the frame contents so the opaque mask respects the corner
radius. Cover the loading, expired, and scanned states in tests.
```

AtomUI 为 Shadcn 的 16px CheckBox 指示器设置了专用圆角。原先沿用的按钮圆角会让部分主题的指示器变圆。同一笔改动还为 CheckBox 圆角和 Empty 插画的填充层次补了测试。

```text
fix(Themes): use dedicated radii for Shadcn CheckBox

Use each Shadcn style's checkbox radius for the 16px indicator.
The inherited button radius could make the indicator circular.

Cover the CheckBox radius and the Empty fill ramp in theme tests.
```
