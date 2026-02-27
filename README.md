# OpenCode Worktree Orchestration

[![OCX Registry](https://img.shields.io/badge/OCX-Registry-blue)](https://github.com/Deslord319/opencode-worktree-orchestration)

Orchestrator Agent + Worktree + Issue + Gate 编排配置，支持多批次并行开发。

## 角色架构

| 位置              | Agent          | 职责           |
| ----------------- | -------------- | -------------- |
| 主会话            | `orchestrator` | 分解任务、创建 worktree、Gate Review |
| Worktree 实例     | `build` (默认) | 执行具体开发任务 |
| Subagent          | `reviewer`     | Gate Review 检查 |

## 安装

### 前置条件

- [OpenCode](https://opencode.ai) >= 1.0.0
- [OCX](https://github.com/kdcokenny/ocx) >= 1.0.16

```bash
# 安装 OCX
npm install -g ocx
# 或
curl -fsSL https://ocx.kdco.dev/install.sh | sh
```

### 方式 1: OCX 安装（推荐）

```bash
# 1. 初始化 OCX（首次）
ocx init

# 2. 添加 Registry
ocx registry add https://raw.githubusercontent.com/Deslord319/opencode-worktree-orchestration/main/dist --name wto

# 3. 安装完整配置
ocx add wto/full

# 4. 启动 OpenCode（请直接使用 opencode）
opencode .
```

> 说明：`ocx add` 会生成 `.opencode/opencode.jsonc`，OpenCode 可直接读取该配置。
> 当前不建议通过 `ocx opencode` 启动（已知在部分环境下 agent 解析为空）。

### 方式 2: 手动安装

```bash
# 克隆仓库
git clone https://github.com/Deslord319/opencode-worktree-orchestration.git

# 复制组件到项目
cp -r files/agent/* /path/to/your/project/.opencode/agent/
cp -r files/skills/* /path/to/your/project/.opencode/skills/

# 复制 AGENTS.md 模板到 .opencode（推荐）
mkdir -p /path/to/your/project/.opencode
cp AGENTS.md /path/to/your/project/.opencode/AGENTS.md
```

## 组件说明

### 安装后的目录结构

```
.opencode/
├── agent/
│   ├── orchestrator.md  # 编排 Agent (primary)
│   └── reviewer.md      # Gate Review Agent (subagent)
└── skills/
    ├── batch/
    │   └── SKILL.md     # 批次执行规约
    └── claim/
        └── SKILL.md     # 任务认领规约
```

### 组件列表

| 组件            | 类型         | 说明                            |
| --------------- | ------------ | ------------------------------- |
| `wto/orchestrator` | `ocx:agent`  | 任务分解与批次编排 Agent        |
| `wto/reviewer`     | `ocx:agent`  | Gate Review Agent (subagent)    |
| `wto/batch`        | `ocx:skill`  | 批次执行规约                    |
| `wto/claim`        | `ocx:skill`  | 任务认领规约                    |
| `wto/full`         | `ocx:bundle` | 完整配置（包含以上所有组件）    |

### AGENTS.md

推荐使用 `.opencode/AGENTS.md`（兼容 `AGENTS.md`）定义：

1. **文档索引** - 指向 Stories、规约、架构文档
2. **当前任务** - Orchestrator Agent 动态更新
3. **开发规范** - 项目特定规则
4. **常用命令** - 测试、Lint 等

## 使用

### 1. 配置 AGENTS.md

复制模板并编辑：

```markdown
## 文档索引

| 类型    | 路径                    | 说明       |
| ------- | ----------------------- | ---------- |
| Stories | docs/RFC/006-stories.md | 任务定义   |
| 规约    | forAgentOpenCode.md     | RFC-011    |
| 架构    | docs/RFC/               | RFC 文档   |

## 常用命令

# 测试
pytest backend/tests/ -v

# Lint
ruff check backend/

## 禁止项

rg -n "TEMP_|getattr\(.*\.side" backend/services
```

### 2. 切换到 Orchestrator Agent

```
Tab 键 → 选择 orchestrator
```

### 3. 输入需求

```
按照 RFC-006 实现 Phase 1
```

### 4. Orchestrator Agent 自动

1. 读取文档索引，找到相关 Stories
2. 分析依赖，生成批次
3. 创建 GitHub Issues
4. 创建 Worktrees
5. 监视 PRs，执行 Gate Review
6. 每批次完成后等待确认

### 5. Worktree 实例

新实例启动后自动：

1. 读取 `.opencode/AGENTS.md`（或 `AGENTS.md`）认领任务
2. 按 Allowed Files 修改
3. 满足 Exit Criteria
4. 创建 PR

## 工作流示例

```
用户: 按照 RFC-006 实现 Phase 1

Orchestrator Agent:
1. 读取 docs/RFC/006-stories.md
2. 识别 Phase 1: [S1, S5]
3. 创建 Issues: #101, #102
4. 创建 Worktrees: wt-s1, wt-s5
5. 等待 PRs...

[用户在 wt-s1 终端]
Build Agent (worktree): 我已认领任务 Issue #101, Story S1
... 工作中 ...
Build Agent: 创建 PR #201

[主会话]
Orchestrator Agent:
  Batch-1 完成:
  | Story | Issue | PR   | Gate |
  | S1    | #101  | #201 | PASS |
  | S5    | #102  | #202 | PASS |
  
  继续执行 Batch-2?

用户: 继续
Orchestrator Agent: 开始执行 Batch-2...
```

## 自定义

### 修改文档索引

编辑 `.opencode/AGENTS.md`（或 `AGENTS.md`）：

```markdown
## 文档索引

| 类型    | 路径                    | 说明       |
| ------- | ----------------------- | ---------- |
| Stories | docs/RFC/006-stories.md | 任务定义   |
| 规约    | forAgentOpenCode.md     | RFC-011    |
| 架构    | docs/RFC/               | RFC 文档   |
```

### 修改禁止项

编辑 `.opencode/AGENTS.md`（或 `AGENTS.md`）：

```markdown
## 禁止项

rg -n "TEMP_|getattr\(.*\.side" backend/services
```

### 修改测试命令

编辑 `.opencode/AGENTS.md`（或 `AGENTS.md`）：

```markdown
## 常用命令

# 测试
pytest backend/tests/ -v

# Lint
ruff check backend/
```

## OCX 兼容性说明

当前推荐链路：

```bash
ocx add wto/full
opencode .
```

### 验证命令

```bash
# 1) 组件安装状态
ocx list -i --json

# 2) 组件与 registry 一致性（应为 hasChanges=false）
ocx diff wto/orchestrator --json
ocx diff wto/reviewer --json
ocx diff wto/batch --json
ocx diff wto/claim --json

# 3) OpenCode 实际解析到的 agent（应包含 orchestrator/reviewer）
opencode debug config | rg "orchestrator|reviewer|\\.opencode/skills"
```

### 常见问题

```bash
# 现象：ocx opencode debug config 中 agent 是 {}
# 处理：改用 opencode 直接启动
opencode .
```

```bash
# 克隆仓库
git clone https://github.com/Deslord319/opencode-worktree-orchestration.git
cd opencode-worktree-orchestration

# 验证并构建
ocx build . --out dist

# 修改后推送，dist/ 会自动更新
git add -A && git commit -m "update" && git push
```

## License

MIT
