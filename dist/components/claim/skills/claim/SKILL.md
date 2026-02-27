# 任务认领规约

定义新 Worktree 实例如何认领任务。

## 角色说明

| 位置              | Agent        | 职责           |
| ----------------- | ------------ | -------------- |
| 主会话            | `orchestrator` | 分解任务、创建 worktree、Gate Review |
| Worktree 实例     | `build` (默认) | 执行具体开发任务，读取 .opencode/AGENTS.md 认领任务 |

Worktree 中的实例使用默认的 `build` agent，拥有完整的文件操作权限。

## 启动后必须执行

前提：当前实例必须由 `worktree_create` 创建，且在独立 worktree 目录中运行。

### Step 1: 读取任务规则文件

```bash
# AGENTS.md 位于 .opencode 目录
cat .opencode/AGENTS.md
```

找到「当前任务」部分。

### Step 2: 解析任务信息

从 `.opencode/AGENTS.md` 获取：

- `Issue`: #<number>
- `Story`: <story-id>
- `目标`: <description>
- `Allowed Files`: 文件列表
- `Exit Criteria`: 完成标准

### Step 3: 确认认领

输出以下内容：

```
我已认领任务:
- Issue: #<number>
- Story: <story-id>
- 目标: <description>
- Allowed Files: <count> 个文件
- Exit Criteria: <count> 项
```

### Step 4: 开始工作

按照任务规则文件中的规范和 Exit Criteria 开始工作。

## Commit Message 格式

```
<type>(<scope>): <description> (#<issue-number>)
```

类型：
- `feat` - 新功能
- `fix` - 修复
- `test` - 测试
- `refactor` - 重构
- `docs` - 文档

示例：
```
feat(redis): 实现 lpush 方法 (#101)
test(redis): 添加 lpush 单元测试 (#101)
```

## 文件修改约束

- **只能修改** Allowed Files 中的文件
- **禁止修改** 其他文件
- **禁止** `git add .`，必须显式指定文件

## 完成后

1. **检查 Exit Criteria**
   - 所有 checkbox 必须满足
   - 测试必须通过

2. **更新测试报告**（如有要求）

3. **推送 Commit**
   ```bash
   git add <allowed-files>
   git commit -m "<type>(<scope>): <desc> (#<issue>)"
   git push
   ```

4. **创建 PR**
   ```bash
   gh pr create --title "[Story-<id>] <title>" --body-file /tmp/pr.md
   ```

## PR Body 模板

```markdown
## Summary
<变更摘要>

## Changes
- <change-1>
- <change-2>

## Test Plan
<测试命令和结果>

## Checklist
- [ ] Exit Criteria 满足
- [ ] 测试通过
- [ ] 仅修改 Allowed Files

Closes #<issue-number>
```
