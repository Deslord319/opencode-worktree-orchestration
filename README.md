# OpenCode Worktree Orchestration

Plan Agent + Worktree + Issue + Gate 编排配置。

## 安装

### 方式 1: 手动安装

```bash
# 复制配置到项目
cp -r .opencode /path/to/your/project/

# 复制 AGENTS.md 模板
cp AGENTS.md /path/to/your/project/
```

### 方式 2: OCX 安装

```bash
ocx add <registry>/opencode-worktree-orchestration
```

## 配置说明

### 文件结构

```
.opencode/
├── agents/
│   ├── plan.md       # Plan Agent 定义
│   └── reviewer.md   # Reviewer Subagent 定义
├── rules/
│   ├── batch.md      # 批次执行规约
│   └── claim.md      # 任务认领规约
└── opencode.json     # 配置入口
```

### AGENTS.md

项目根目录的 `AGENTS.md` 定义：

1. **文档索引** - 指向 Stories、规约、架构文档
2. **当前任务** - Plan Agent 动态更新
3. **开发规范** - 项目特定规则
4. **常用命令** - 测试、Lint 等

## 使用

### 1. 切换到 Plan Agent

```
Tab 键 → 选择 plan
```

### 2. 输入需求

```
实现用户认证功能
```

### 3. Plan Agent 自动

1. 读取文档索引，找到相关 Stories
2. 分析依赖，生成批次
3. 创建 GitHub Issues
4. 创建 Worktrees
5. 监视 PRs，执行 Gate Review
6. 每批次完成后等待确认

### 4. Worktree 实例

新实例启动后自动：

1. 读取 AGENTS.md 认领任务
2. 按 Allowed Files 修改
3. 满足 Exit Criteria
4. 创建 PR

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
### 禁止项

```bash
rg -n "TEMP_|getattr\(.*\.side" backend/services
```
```

### 修改测试命令

编辑 `AGENTS.md`：

```markdown
## 常用命令

```bash
# 测试
pytest backend/tests/ -v

# Lint
ruff check backend/
```
```

## 工作流示例

```
用户: 按照 RFC-006 实现 Phase 1

Plan Agent:
1. 读取 docs/RFC/006-stories.md
2. 识别 Phase 1: [S1, S5]
3. 创建 Issues: #101, #102
4. 创建 Worktrees: wt-s1, wt-s5
5. 等待 PRs...

[用户在 wt-s1 终端]
新实例: 我已认领任务 Issue #101, Story S1
... 工作中 ...
新实例: 创建 PR #201

[主会话]
Plan Agent:
  Batch-1 完成:
  | Story | Issue | PR   | Gate |
  | S1    | #101  | #201 | PASS |
  | S5    | #102  | #202 | PASS |
  
  继续执行 Batch-2?

用户: 继续
Plan Agent: 开始执行 Batch-2...
```

## License

MIT
