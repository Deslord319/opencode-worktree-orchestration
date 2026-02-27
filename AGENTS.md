# AI Trading System

> 基于机器学习的加密货币自动化交易系统

## 文档索引

| 类型    | 路径                                                 | 说明           |
| ------- | ---------------------------------------------------- | -------------- |
| Stories | docs/RFC/006-microservices-implementation-stories.md | 任务定义与依赖 |
| 规约    | forAgentOpenCode.md                                  | RFC-011 Gate   |
| 架构    | docs/RFC/                                            | RFC 文档       |
| 业务规则| docs/BRD/                                            | 业务约束       |

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

### RFC-011 Gate 三原则

1. **边界一次性标准化** - 外部输入在边界层清洗完成
2. **核心流程契约强制化** - 禁止 `Optional[Any]/dict` 直接穿透到交易核心路径
3. **移除隐式修复路径** - 禁止临时 ID、类型猜测、空值兜底后继续交易

### 禁止项

```
rg -n "TEMP_|getattr\(.*\.side.*value|\.get\('side'.*or.*'none'" backend/services
```

---

## 常用命令

```bash
# P1 单元测试
.venv_linux/bin/python -m pytest backend/tests/ -v --ignore=backend/tests/integration/

# P2 集成测试
.venv_linux/bin/python -m pytest backend/tests/integration/ -v

# Docker 部署
docker-compose up -d
docker-compose ps
docker-compose logs -f
```

---

## 项目结构

```
ai-trading-system/
├── backend/           # Python FastAPI 后端
│   ├── services/      # 微服务
│   ├── shared/        # 共享模块
│   └── tests/         # 测试
├── frontend/          # React TypeScript 前端
├── ml-training/       # ML 模型训练
├── docs/              # 文档
│   ├── RFC/           # 技术设计
│   ├── BRD/           # 业务规则
│   └── test-reports/  # 测试报告
└── scripts/           # 脚本
```
