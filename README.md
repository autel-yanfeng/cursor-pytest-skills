# 🧪 cursor-pytest-skills

> Cursor IDE 自动生成 pytest 测试用例的 **Rules + Skills** 配置集。
> 设计原则：**精简注入，按需加载，最小化 token 消耗。**

---

## 📦 目录结构

```
.cursor/
├── rules/
│   ├── must.mdc              # 团队铁律（alwaysApply: true，极简）
│   ├── python.mdc            # Python 规范（globs: **/*.py）
│   └── pytest.mdc            # 测试规范（globs: test_*.py）
└── skills/
    ├── generate-unit-test.md  # 生成单元测试
    ├── generate-fixture.md    # 生成/提取 fixture
    ├── generate-parametrize.md# 参数化重构
    └── review-test-quality.md # 测试质量审查
```

---

## 🚀 快速安装

```bash
git clone https://github.com/autel-yanfeng/cursor-pytest-skills.git /tmp/cps
cp -r /tmp/cps/.cursor ./
```

---

## 📐 设计原则：高效利用 Token

Rules 和 Skills 内容越多，每次对话消耗的 token 越多，反而压缩了真正用于代码的 context 空间。本仓库遵循以下原则：

### Rules 分层策略

| 文件 | alwaysApply | globs | token 消耗 | 用途 |
|------|------------|-------|-----------|------|
| `must.mdc` | ✅ true | 无 | ~50 | 团队铁律，5条以内 |
| `python.mdc` | ❌ false | `**/*.py` | ~80 | 打开 Python 文件时触发 |
| `pytest.mdc` | ❌ false | `test_*.py` | ~120 | 打开测试文件时触发 |

**核心思路：**
```
alwaysApply: true  → 只放不可妥协的铁律（<200 token）
globs 匹配触发    → 场景规范，按需注入
Rule 内容格式     → 指令式列表，非散文，节省 60% token
```

### Skills 零消耗设计

Skills 文件**不会自动注入**，只有主动 `@引用` 时才消耗 token：

```
不引用时：0 token 消耗
@generate-unit-test 触发时：~100 token（精简后）
```

**对比：**
```
❌ 把 skill 内容写进 Rule → 每次都注入，800 token 浪费
✅ 存为 skill，用时 @引用 → 按需加载，完全可控
```

### Token 消耗对比

| 方式 | 每次消耗 | 说明 |
|------|---------|------|
| 全部 alwaysApply | ~2000 token | 大量规范与当前任务无关 |
| globs 按需触发 | ~200 token | 只注入相关规范 |
| Skill @引用 | 0~100 token | 不引用时零消耗 |

---

## 📖 使用说明

### Rules — 自动生效，无需操作

打开 `test_xxx.py` 时，`pytest.mdc` 自动注入；打开任意 `.py` 时，`python.mdc` 自动注入。

### Skills — 在 Cursor Chat 中 @ 触发

#### 生成单元测试

选中目标函数，在 Chat 输入：
```
@generate-unit-test
```
生成包含正常路径、边界值、异常路径的完整测试文件。

#### 生成 Fixture

打开测试文件，在 Chat 输入：
```
@generate-fixture
```
自动识别重复初始化逻辑，生成 `conftest.py`。

#### 参数化重构

选中多个相似测试函数，输入：
```
@generate-parametrize
```
合并为 `@pytest.mark.parametrize` 参数化版本。

#### 测试质量审查

打开测试文件，输入：
```
@review-test-quality
```
输出：问题列表（严重程度/位置/建议）+ 重构示例。

---

## 🛠 自定义 Rules

### 修改铁律（`must.mdc`）

保持 **5条以内**，每条一行，直接指令：

```markdown
---
alwaysApply: true
---
- 中文回复
- 类型注解完整
- 禁止裸 except
```

### 添加新场景 Rule

```markdown
---
description: FastAPI 路由规范
globs: ["*router*.py", "*api*.py"]
alwaysApply: false
---
- 路由函数必须有类型注解和 response_model
- 统一用 HTTPException 抛出错误
- 路径参数用下划线命名
```

### 添加新 Skill

在 `.cursor/skills/` 新建 `.md` 文件：

```markdown
# Skill: your-skill

**触发词：** `你的触发词`

## 指令
1. 要求一
2. 要求二
（保持简短，去掉示例代码，节省 token）
```

---

## 📋 依赖

```bash
pip install pytest pytest-mock pytest-cov
```

推荐 `pytest.ini`：
```ini
[pytest]
testpaths = tests
addopts = -v --tb=short
```

---

## 相关文档

- [Cursor Rules & Skills 完整配置教程](./cursor-rules-skills-guide.md)
- [Cursor 完整配置指南（Rules/Skills/MCP）](./cursor-complete-config-guide.md)
- Cursor 官方文档：https://docs.cursor.com
- MCP 协议：https://modelcontextprotocol.io

---

## 📄 License

MIT
