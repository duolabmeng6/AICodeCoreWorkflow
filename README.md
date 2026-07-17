# AI Code Core Workflow

一个同时兼容 Claude Code 与 Codex 的编程核心工作流 Skill。

## 安装

把下面这句话发给 Claude Code、Codex 或其他支持 Agent Skills 的编程 Agent：

```text
帮我安装技能 https://github.com/duolabmeng6/AICodeCoreWorkflow
```

Agent 应当自动完成以下工作：

1. 获取仓库并识别 `skills/workflow`；
2. 安装到当前 Agent 的个人 Skill 目录；
3. 保持自动调用配置；
4. 验证 Skill 可被发现和调用；
5. 告知安装结果与显式调用方式。

如果希望说得更明确，可以使用：

```text
帮我从 https://github.com/duolabmeng6/AICodeCoreWorkflow 安装 skills/workflow，作为个人全局技能，保留自动调用配置，并验证安装成功。
```

## 使用

安装后，Agent 会在需要落地代码修改的开发任务中自动采用工作流：

```text
研究 → 构思 → 计划 → 等待批准 → 执行 → 评审
```

也可以显式调用：

```text
Claude Code: /workflow 实现用户登录功能
Codex:       $workflow 实现用户登录功能
```

计划是唯一强制批准门。批准锁定目标、方案、范围、外部行为和主要风险，不冻结具体实现细节。阶段内部由 Agent 自主选择调查方式、推理深度、工具和实现路径。

## 自动调用

- Claude Code 根据 `SKILL.md` 的 `description` 自动判断何时使用。
- Codex 通过 `agents/openai.yaml` 中的 `allow_implicit_invocation: true` 允许自动使用。
- 纯解释、简单问答和只读代码审查不会自动进入工作流，除非用户显式调用。

## 仓库结构

```text
skills/
└── workflow/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

## 手动安装回退

仅当 Agent 无法自动安装时，才需要手动把 `skills/workflow` 复制或链接到对应目录：

| Agent | 个人目录 | 项目目录 | 显式调用 |
| --- | --- | --- | --- |
| Claude Code | `~/.claude/skills/workflow` | `.claude/skills/workflow` | `/workflow` |
| Codex | `~/.agents/skills/workflow` 或 `$CODEX_HOME/skills/workflow` | `.agents/skills/workflow` | `$workflow` |

使用软链接安装时，后续只需更新本仓库；复制安装时，需要重新复制 `skills/workflow`。


# linux.do 社区
https://linux.do/ 
id: duolabmeng6
