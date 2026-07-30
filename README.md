# AI Code Core Workflow

一组同时兼容 Claude Code 与 Codex 的项目开发 Skill：

- `workflow`：显式调用的编程核心工作流。
- `core-memory`：可独立或由 `workflow` 调用的项目长期记忆助手。

## 安装

把下面这句话发给 Claude Code、Codex 或其他支持 Agent Skills 的编程 Agent：

```text
帮我安装技能 https://github.com/duolabmeng6/AICodeCoreWorkflow
```

Agent 应当自动完成以下工作：

1. 获取仓库并识别 `skills/workflow` 与 `skills/core-memory`；
2. 把两个 Skill 安装到当前 Agent 的个人 Skill 目录；
3. 保持 `workflow` 仅显式调用，并允许 `core-memory` 根据项目任务自动匹配；
4. 验证两个 Skill 均可被发现和调用；
5. 告知安装结果与显式调用方式。

如果希望说得更明确，可以使用：

```text
帮我从 https://github.com/duolabmeng6/AICodeCoreWorkflow 安装 skills/workflow 和 skills/core-memory，作为个人技能；workflow 仅显式调用，core-memory 允许在项目任务中自动匹配，并验证安装成功。
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

## 仓库结构

```text
skills/
├── core-memory/
│   └── SKILL.md
└── workflow/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── references/
        ├── coreflow-ledger.md
        └── agent-orchestration.md
```

## 手动安装回退

仅当 Agent 无法自动安装时，才需要手动把 `skills/workflow` 和 `skills/core-memory` 分别复制或链接到对应 Skill 根目录：

| Agent | 个人 Skill 根目录 | 项目 Skill 根目录 | 显式调用 |
| --- | --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` | `/workflow`、`/core-memory` |
| Codex | `~/.agents/skills/` 或 `$CODEX_HOME/skills/` | `.agents/skills/` | `$workflow`、`$core-memory` |

使用软链接安装时，后续只需更新本仓库；复制安装时，需要重新复制 `skills/workflow` 和 `skills/core-memory`。


# linux.do 社区
https://linux.do/ 
id: duolabmeng6
