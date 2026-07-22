# Codex Tasks Workspace

一个轻量、Spec 驱动的 Codex 工作目录。

它不引入额外的 Agent 框架、数据库或 Memory 系统，只组合 Codex 原生的根对话、内置 Worker、`/goal`、subagent、worktree 和 code review。

## 结构

```text
workspace/
├── AGENTS.md
├── README.md
├── .codex/
│   ├── config.toml
│   └── agents/
│       └── reviewer.toml
└── tasks/
    └── <task-name>/
        ├── spec.md
        ├── plan.md
        ├── progress.md
        └── artifacts/
            └── review.md
```

代码保留在真实仓库中，不复制到 `tasks/`。每个任务的 `spec.md` 记录代码仓库和分支位置。

## 工作流程

```text
需求讨论
  ↓
Superpowers brainstorming → spec.md
  ↓
Superpowers writing-plans → plan.md
  ↓
用户确认 Spec 和 Plan
  ↓
Codex 内置 Worker + 原生 /goal
  ↓
必要时使用 worktree + branch 并行
  ↓
Worker 集成、测试并更新 progress.md
  ↓
自定义 Reviewer 审查
  ↓
SPEC_INCOMPLETE / P0_BUG → Worker 继续 Goal
PASS                       → 根对话汇报完成
```

## 角色

### 根对话

你唯一需要长期交流的入口，负责：

- 讨论和澄清不同任务
- 创建、更新 Spec 和 Plan
- 启动和协调 Worker
- 启动 Reviewer
- 汇总进度、阻塞和最终结果

### Worker

直接使用 Codex 内置 `worker`，不自定义和覆盖它。

每个 Worker：

- 只负责一个任务
- 读取对应的 Spec 和 Plan
- 在自己的线程使用原生 `/goal`
- 持续实施、测试并更新 `progress.md`
- 必要时派生下一层 subagent
- 负责集成所有并行分支并验证最终结果

### Reviewer

Reviewer 是唯一保留的自定义 Agent，负责：

- 检查 Plan 是否覆盖完整 Spec
- 检查每条验收标准是否有实现和证据
- 审查集成后的代码 Diff 和测试结果
- 判断是否存在 P0 Bug

Reviewer 只返回三种结果：

- `PASS`：完整满足 Spec，且没有 P0 Bug
- `SPEC_INCOMPLETE`：需求或验收标准没有完成
- `P0_BUG`：存在阻断发布的严重问题

非 P0 问题记录在 `artifacts/review.md`，不阻止任务完成。

## Task 文件

### `spec.md`

由 Superpowers brainstorming 生成，至少包含：

```yaml
repository: /absolute/local/repository/path
remote: optional-git-remote
base_branch: main
task_branch: task/<task-name>
```

正文记录目标、范围、约束、需求和可验证的验收标准。

### `plan.md`

由 Superpowers writing-plans 原样生成，作为 Worker 的完整实施计划。

### `progress.md`

记录当前状态、已完成内容、验证证据、阻塞和下一步，确保根对话和后续会话可以恢复任务上下文。

### `artifacts/review.md`

保存 Reviewer 的最终结论、缺失需求、P0 问题、验证结果和非阻断建议。

## 并行工作

不同任务可以运行不同的 Worker。

同一仓库需要多个 Agent 并行写代码时，每个 Agent 必须使用独立 worktree 和 branch：

```text
task branch
├── worktree A / branch A
├── worktree B / branch B
└── worktree C / branch C
```

Worker 负责等待、检查、集成和统一验证。多个 Agent 可以同时修改同一仓库，但不能同时写同一个物理 checkout。

## 使用方式

在 Codex Desktop 中打开本目录并新建一个根对话。

创建任务：

```text
新建任务 payment-api。代码在 /path/to/repository，先和我澄清需求并生成 Spec 和 Plan，不要实施。
```

开始实施：

```text
payment-api 的 Spec 和 Plan 已确认，开始任务。
```

查看和控制：

```text
payment-api 现在进度怎么样？
暂停 payment-api。
继续 payment-api。
打开 payment-api 的 Worker。
修改 payment-api 的需求：……
```

你也可以在 Codex Desktop 中直接打开 Worker 或 Reviewer 线程进行补充和纠正。

## 当前边界

这是一套交互式 Agent Loop，不是额外的常驻调度服务：

- 不提供独立数据库或 Memory
- 不定时扫描其他无关 Session
- 不在 Codex 关闭后自行运行
- 不自动让多个 Agent 共享同一个 checkout

长期任务由每个 Worker 自己的原生 Goal 持续推进，任务文件负责跨线程和跨会话保留可检查的状态。
