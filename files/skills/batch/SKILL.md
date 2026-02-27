# 批次执行规约

定义多 Story 批次执行的标准流程。

## 批次生成

### 1. 读取 Story 定义

从 `.opencode/AGENTS.md` 的「文档索引」找到 Stories 文件，解析：

- Story ID
- Story 名称
- 依赖关系
- 优先级

### 2. 拓扑排序

```
completed = []
batches = []
remaining = all_stories

while remaining:
    ready = [s for s in remaining if all_deps_satisfied(s, completed)]
    if not ready:
        error("循环依赖")
    batches.append(sort_by_priority(ready))
    completed.extend(ready)
    remaining -= ready
```

### 3. 批次队列

```
Batch-1: [S1, S5]  # 无依赖，可并行
Batch-2: [S2]      # 依赖 S1
Batch-3: [S3, S4]  # 依赖 S2, S5，可并行
...
```

## 批次执行流程

```
For each batch:
    1. 创建 Issues
       - 为 batch 中每个 story 创建 GitHub Issue
       - Issue body 包含 Allowed Files, Exit Criteria
    
    2. 创建 Worktrees (并行)
       For each story in batch:
           a. git checkout -b story-<id>-#<issue>
           b. 更新 `.opencode/AGENTS.md` 中的「当前任务」
           c. git commit && push
           d. git worktree add ../wt-<id>
           e. git checkout main
    
    3. 等待完成
       - 用户在各 worktree 终端启动 OpenCode
       - 新实例读取 `.opencode/AGENTS.md` 认领任务
       - 完成后创建 PR
    
    4. Gate Review
       - 调用 @reviewer 检查每个 PR
       - 记录 PASS/BLOCK 结果
    
    5. 汇报确认
       - 输出批次完成报告
       - 等待人类输入：继续 / 暂停 / 讨论
    
    6. 下一批次
```

## 人类确认点

每批次完成后**必须**等待人类确认：

```markdown
## Batch-N 完成

### 结果
| Story | Issue | PR   | Gate |
| ----- | ----- | ---- | ---- |
| S1    | #101  | #201 | PASS |

### 下一批次
Batch-N+1: [S3, S4]
依赖状态: ✅ 已满足

### 确认
- "继续" → 执行下一批次
- "等待" → 暂停
- 其他 → 讨论
```

## 异常处理

- **Gate BLOCK**: 不进入下一批次，等待修复
- **依赖失败**: 后续批次自动暂停
- **Issue 创建失败**: 重试 1 次，失败则暂停
- **Worktree 冲突**: 提示用户手动处理
