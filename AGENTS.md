# 项目名称

> 一句话描述项目

## 文档索引

| 类型    | 路径                | 说明           |
| ------- | ------------------- | -------------- |
| Stories | docs/stories.md     | 任务定义与依赖 |
| 规约    | forAgentOpenCode.md | 开发规范       |
| 架构    | docs/RFC/           | 技术设计       |

<!-- Plan Agent 会读取此索引找到相关文档 -->

---

## 当前任务

<!-- 由 Plan Agent 动态更新，请勿手动修改 -->

**Issue**: #<number>
**Story**: <story-id>
**目标**: <description>

### Allowed Files
- <file-1>
- <file-2>

### Exit Criteria
- [ ] <criteria-1>
- [ ] <criteria-2>

---

## 开发规范

<!-- 引用项目规约文件的关键内容，或直接指向文件 -->

参见：`forAgentOpenCode.md`

### 核心规则

1. <规则-1>
2. <规则-2>

### 禁止项

```
rg -n "<forbidden-patterns>" <scope>
```

---

## 常用命令

```bash
# 测试
<test-command>

# Lint
<lint-command>

# 类型检查
<typecheck-command>
```

---

## 项目结构

```
project/
├── src/          # 源代码
├── tests/        # 测试
├── docs/         # 文档
└── scripts/      # 脚本
```
