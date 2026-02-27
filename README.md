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
# 1. 添加 Registry
ocx registry add https://github.com/Deslord319/opencode-worktree-orchestration --name wto

# 2. 安装完整配置
ocx add wto/full

# 或单独安装组件
ocx add wto/orchestrator  # 只安装 orchestrator agent
ocx add wto/reviewer      # 只安装 reviewer subagent
ocx add wto/batch         # 只安装批次执行规约
ocx add wto/claim         # 只安装任务认领规约
```

### 方式 2: 手动安装

```bash
# 克隆仓库
git clone https://github.com/Deslord319/opencode-worktree-orchestration.git

# 复制组件到项目
cp -r files/agent/* /path/to/your/project/.opencode/agent/
cp -r files/skills/* /path/to/your/project/.opencode/skills/

# 复制 AGENTS.md 模板
cp AGENTS.md /path/to/your/project/
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

项目根目录的 `AGENTS.md` 定义：

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

1. 读取 AGENTS.md 认领任务
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

编辑 `AGENTS.md`：

```markdown
## 文档索引

| 类型    | 路径                    | 说明       |
| ------- | ----------------------- | ---------- |
| Stories | docs/RFC/006-stories.md | 任务定义   |
| 规约    | forAgentOpenCode.md     | RFC-011    |
| 架构    | docs/RFC/               | RFC 文档   |
```

### 修改禁止项

编辑 `AGENTS.md`：

```markdown
## 禁止项

rg -n "TEMP_|getattr\(.*\.side" backend/services
```

### 修改测试命令

编辑 `AGENTS.md`：

```markdown
## 常用命令

# 测试
pytest backend/tests/ -v

# Lint
ruff check backend/
```

## 开发 Registry

```bash
# 克隆仓库
git clone https://github.com/Deslord319/opencode-worktree-orchestration.git
cd opencode-worktree-orchestration

# 验证 Registry
ocx build . --out dist

# 本地测试
cd /path/to/test-project
ocx registry add /path/to/opencode-worktree-orchestration --name wto-local
ocx add wto-local/full
```

## License

MIT
