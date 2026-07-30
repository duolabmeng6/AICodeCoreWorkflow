# AI Code Core Workflow

一组同时兼容 Claude Code 与 Codex 的项目开发 Skill 与 Codex 自定义代理定义：

- `workflow`：显式调用的编程核心工作流。
- `core-memory`：可独立或由 `workflow` 调用的项目长期记忆助手。
- `review-agent`：只读、缺陷优先的评审 Skill 回退包。
- `terra-explorer`、`luna-worker`、`terra-reviewer`：供 Codex 编排时生成子代理的自定义代理。
- `agents/codex/AGENTS.md`：Codex 全局通用子代理调度规则的中文模板，不替代 `workflow` 的分级流程。

## 安装

把下面这句话发给 Claude Code、Codex 或其他支持 Agent Skills 的编程 Agent：

```text
帮我安装技能 https://github.com/duolabmeng6/AICodeCoreWorkflow
```

执行这条自然语言指令的 Agent 应遵守以下安装协议：

1. 把仓库下载或克隆到临时目录；不要把仓库根目录当成单个 Skill，也不要假设 Skill 安装器会处理 `agents/`。
2. 安装 `skills/workflow` 与 `skills/core-memory` 到宿主的个人 Skill 根目录，各自保留独立目录名。
3. 如果宿主是 Codex，先把 `${CODEX_HOME:-$HOME/.codex}` 解析为实际的 `$CODEX_HOME` 路径；把 `agents/codex/AGENTS.md` 安装到 `$CODEX_HOME/AGENTS.md`，并且只把 `agents/codex/*.toml` 安装到 `$CODEX_HOME/agents/`。
4. 每个目标文件或目录都先检查冲突：目标不存在则安装；内容相同则复用；目标存在但内容不同则保留并报告差异，只有获得明确同意后才能替换。绝不静默覆盖已有的全局 `$CODEX_HOME/AGENTS.md`。
5. 如果 Codex 已提供内置或 system `review-agent`，直接复用且不得覆盖；只在缺失时安装仓库中的 `skills/review-agent`。兼容 Agent Skills 的非 Codex 宿主也只在没有等价评审 Skill 时安装该回退。
6. 保持 `workflow` 与 `review-agent` 仅显式调用，并允许 `core-memory` 根据项目任务自动匹配。
7. 验证 Skill 可被发现，并在 Codex 中验证 `terra-explorer`、`luna-worker`、`terra-reviewer` 三个角色可用；全局指导需新建 Codex 任务或会话后验证已加载。最后报告新增、复用、冲突、回退和显式调用方式。

如果希望说得更明确，可以使用：

```text
帮我从 https://github.com/duolabmeng6/AICodeCoreWorkflow 安装 workflow 和 core-memory；Codex 还要把中文全局 AGENTS 模板安装到解析后的 $CODEX_HOME/AGENTS.md，并只把 agents/codex/*.toml 安装到 $CODEX_HOME/agents/，全程保留并报告内容不同的已有文件，未经明确同意不得覆盖；优先复用已有 review-agent、不覆盖 system Skill，其他兼容宿主缺少等价能力时可安装仓库回退。请新建 Codex 任务或会话验证全局指导和自定义代理已加载。
```

## 使用

显式调用核心工作流：

```text
Claude Code: /workflow 实现用户登录功能
Codex:       $workflow 实现用户登录功能
```

工作流会先调用 `core-memory` 读取相关项目记忆，再按复杂度分级推进：

```text
读取项目记忆 → 研究 → 构思 → 计划
                            ├─ 普通实现请求 → 自动执行 → 评审
                            └─ 讨论或确认计划 → 等待确认 → 执行 → 评审
评审完成 → 维护项目记忆 → 询问是否归档
```

编排屏障：根代理负责分类、最终计划、整合与验收；standard 需只读 explorer→单写 worker→独立 reviewer，complex/high-risk 还需计划 reviewer 与按风险轴评审。explorer 结果齐备后才能写入，worker 完成后才能评审，实质修复必须聚焦复评。执行期间根代理不修改 worker 所有文件；禁止子代理委派。能力缺失经一次有界重试后记录回退，simple/standard 可由根代理建立等价证据后继续，complex 无等价证据则阻塞。内部计划评审不构成用户批准门。

`core-memory` 也可以单独显式调用；在普通项目开发、设计、调试和评审对话中，Agent 可以根据 Skill description 自动采用它：

```text
Claude Code: /core-memory 记住后台主标题统一使用 20px
Codex:       $core-memory 记住后台主标题统一使用 20px
```

普通实现、修改和修复请求默认授权 Agent 在计划后自动执行，不再例行询问“是否按此计划执行”。只有用户主要想讨论、评审、比较或确认方案与计划，或明确要求实施前等待时，工作流才停在计划阶段。是否等待按整体语义判断，不以单个关键词决定。授权锁定目标、范围、外部行为和主要风险，不冻结范围内的实现细节；需要扩大范围、改变外部行为或取得新权限时仍会询问。

在向用户提问、比较方案、说明计划或解释程序逻辑时，Agent 会优先选择一目了然的 Markdown 表达：精确对比、映射、风险和检查项使用表格；流程、时序、状态、依赖和分支关系使用 Mermaid。Mermaid 流程图默认垂直展示，只有少于 5 个短节点且没有复杂分支时才考虑横向；简单内容不会为了可视化而可视化。

## Core Memory

`core-memory` 把长期有效的项目知识维护为 `.coreflow/memory/` 下的 Markdown Wiki。它会先读索引、再按需读取主题页，并在用户纠正、明确要求记住、Agent 在当前对话发现同类错误两次或任务完成后发现高价值知识时自动维护记忆。

```text
./.coreflow/memory/
├── index.md
├── {topic}/
│   └── {YYYY-MM}/
│       └── {page}.md
└── logs/
    └── {YYYY-MM}.md
```

主题由 Agent 根据项目自主组织；年月表示主题页首次建立的月份，后续原位维护。主题页保存当前有效结论，当月日志记录简短变更历史。记忆只属于当前项目，不建立全局或跨项目记忆。

用户纠正和明确要求记住的内容立即写入；疑似冲突、局部例外与长期规则无法区分或同等级证据冲突时先询问用户。记忆不保存敏感数据、完整对话、思维链、冗长日志或未验证猜测。

项目记忆的读取、维护、纠错、遗忘和检查不会调用任何 Git 命令，也不会修改 `.gitignore`、暂存区、提交、分支或远端。首次真正需要写入时才初始化目录，不创建空 Wiki。

## CoreFlow Ledger

实施获得授权后，Agent 会在当前项目根目录创建 CoreFlow Ledger 任务记录。授权既可以来自普通实现请求的默认自动执行，也可以来自讨论优先模式下的明确批准：

```text
./.coreflow/
└── tasks/
    ├── {YYYY-MM-DD-中文任务概览}/
    │   ├── task.json
    │   ├── prd.md
    │   ├── implement.md
    │   ├── design.md       # 按需
    │   ├── review.md       # 评审时创建
    │   └── research/       # 按需
    └── archive/
        └── {YYYY-MM}/
            └── {YYYY-MM-DD-中文任务概览}/
```

CoreFlow Ledger 保存已授权需求、实施计划、关键决策、验证结果和最终评审。`approvedAt` 保留为兼容字段，记录实施授权实际生效时间。实时执行进度仍由当前 Agent 的任务计划工具维护。

新任务目录使用 `{YYYY-MM-DD}-{中文任务概览}`，例如 `2026-07-19-应用工作室小屏布局优化`。中文概览通常为 6–20 个中文字符，以“对象 + 目标或动作”概括任务；允许保留必要技术名，但不生成纯英文 slug。`task.json.id` 与目录名保持一致，同日重名追加 `-2`、`-3`。已有英文任务目录保持有效，不自动迁移。

任务达到完成标准后不会自动归档。Agent 必须先询问用户；确认后才使用普通文件系统操作移动任务目录。CoreFlow Ledger 的创建、检查、更新和归档不会调用任何 Git 命令，也不会自动暂存、提交或推送内容。开发任务本身的 Git 行为仍由用户要求和运行环境规则决定。

## 自动调用

- `workflow` 仅在用户显式使用 `/workflow` 或 `$workflow` 时调用；其 `agents/openai.yaml` 设置为 `allow_implicit_invocation: false`。
- `core-memory` 通过 `SKILL.md` 的 description 匹配项目开发、UI 设计、调试、实现和评审任务，也支持用户显式调用。
- `workflow` 会明确组合调用 `core-memory`；如果未安装，只提示 `未安装 $core-memory` 并继续工作流。
- 纯聊天、临时目录和没有明确项目归属的任务不创建项目记忆。

## Review Skill 与自定义代理

`$review-agent` 是可显式调用的评审契约：它规定如何检查变更和输出缺陷，也可在宿主没有内置评审 Skill 时作为回退。它本身不是可生成的自定义代理，不能把 Skill 名称当作子代理角色传给生成代理的工具。

Codex 中可生成的角色来自 `agents/codex/*.toml`：`terra-explorer` 负责只读探索，`luna-worker` 负责有边界的写入实现，`terra-reviewer` 负责独立只读评审。其他宿主可以使用语义和边界等价的 explorer、worker、reviewer 角色。

`agents/codex/AGENTS.md` 提供宿主级通用所有权、路由、隔离与验收规则，不复制 `workflow` 的 simple/standard/complex 分级、能力回退和复评闭环，也不会让普通请求自动进入工作流；用户显式调用 `workflow` 后，再叠加 Skill 中更具体的阶段与屏障。安装到解析后的 `$CODEX_HOME/AGENTS.md` 后，需新建 Codex 任务或会话验证指导已加载；若目标已有不同内容，必须保留、报告并取得明确同意，绝不静默覆盖。

## 仓库结构

```text
agents/
└── codex/
    ├── AGENTS.md
    ├── luna-worker.toml
    ├── terra-explorer.toml
    └── terra-reviewer.toml
skills/
├── core-memory/
│   └── SKILL.md
├── review-agent/
│   ├── SKILL.md
│   └── agents/
│       └── openai.yaml
└── workflow/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── references/
        ├── coreflow-ledger.md
        └── agent-orchestration.md
```

## 手动安装回退

仅当 Agent 无法自动安装时，才需要手动复制或链接：

| Agent | 必装 Skill | Review Skill | 自定义代理 | 显式调用 |
| --- | --- | --- | --- | --- |
| Claude Code | 把 `workflow`、`core-memory` 放到 `~/.claude/skills/` 或 `.claude/skills/` | 无等价 Skill 时可安装 `review-agent` | 使用宿主提供的等价角色 | `/workflow`、`/core-memory`、`/review-agent` |
| Codex | 把 `workflow`、`core-memory` 放到 `~/.agents/skills/`、`$CODEX_HOME/skills/` 或项目 `.agents/skills/` | 优先复用内置/system `review-agent`，不得覆盖；缺失时才安装仓库回退 | 解析 `${CODEX_HOME:-$HOME/.codex}`；把 `AGENTS.md` 放到 `$CODEX_HOME/AGENTS.md`，只把 `*.toml` 放到 `$CODEX_HOME/agents/` | `$workflow`、`$core-memory`、`$review-agent` |

使用软链接安装时，后续只需更新本仓库；复制安装时，需要重新复制相应目录。手动安装也必须逐项检查同名目标：不存在则安装，相同则复用，不同则保留并报告，获得明确同意后才能替换；尤其不得静默覆盖全局 `AGENTS.md`、自定义代理或 Codex 已有的内置/system `review-agent`。安装全局模板后需新建 Codex 任务或会话验证指导已加载。


# linux.do 社区
https://linux.do/ 
id: duolabmeng6
