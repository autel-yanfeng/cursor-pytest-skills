# 🧪 Cursor pytest Skills & Rules

> 在 Cursor IDE 中自动生成高质量 pytest 测试用例的 Rules + Skills 配置集。

## 📦 目录结构

```
.cursor/
├── rules/
│   └── pytest.mdc              # pytest 测试规范（全局约束）
└── skills/
    ├── generate-unit-test.md   # 生成单元测试
    ├── generate-fixture.md     # 生成/提取 fixture
    ├── generate-parametrize.md # 参数化重构
    └── review-test-quality.md  # 测试质量审查
```

---

## 🚀 快速开始

### 1. 安装

**方式一：克隆到现有项目**

```bash
# 在你的项目根目录下
git clone https://github.com/your-org/cursor-pytest-skills.git /tmp/cursor-pytest-skills
cp -r /tmp/cursor-pytest-skills/.cursor ./
```

**方式二：作为 Git Submodule**

```bash
git submodule add https://github.com/your-org/cursor-pytest-skills.git .cursor-pytest
cp -r .cursor-pytest/.cursor ./
```

**方式三：手动复制**

将 `.cursor/` 目录整体复制到你的 Python 项目根目录下。

### 2. 验证安装

打开 Cursor，进入任意 Python 文件，在 Chat 中输入：
```
为 divide 函数生成测试
```
如果 Cursor 使用了本配置的格式（AAA 结构、parametrize 等），说明安装成功。

---

## 📖 使用说明

### Rules — `pytest.mdc`

Rules 是**全局约束**，只要你在项目中涉及测试文件，Cursor 就会自动遵守这些规范。

**适用文件：** `test_*.py`、`*_test.py`、`conftest.py`

无需手动触发，Cursor 在以下场景自动应用：
- 修改或新建测试文件时
- 让 Cursor 帮你补全测试代码时

---

### Skills — 手动触发

Skills 需要你在 Cursor Chat 中明确描述需求来触发。

#### 🔨 Skill 1：生成单元测试

**触发词：** `为 [函数名] 生成测试` / `写测试用例` / `生成单元测试`

**使用方法：**
1. 打开包含目标函数的文件
2. 选中函数代码（可选，但推荐）
3. 在 Cursor Chat 中输入：

```
为这个函数生成 pytest 单元测试
```

**会生成：**
- ✅ 正常路径测试（含参数化）
- 🔲 边界值测试
- ❌ 异常路径测试
- AAA 三段式结构 + 完整注释

---

#### 🏗️ Skill 2：生成 Fixture

**触发词：** `生成 fixture` / `提取公共测试数据` / `帮我写 conftest`

**使用方法：**
1. 打开测试文件
2. 在 Cursor Chat 中输入：

```
这些测试有重复的初始化代码，帮我提取为 conftest.py 的 fixture
```

**会生成：**
- 合适 scope 的 fixture
- 工厂模式 fixture（复杂场景）
- 资源管理 fixture（含 yield 清理）

---

#### 🔄 Skill 3：参数化重构

**触发词：** `帮我参数化这些测试` / `用 parametrize 重构`

**使用方法：**
1. 选中多个结构相似的测试函数
2. 在 Cursor Chat 中输入：

```
这几个测试结构很相似，帮我用 parametrize 合并
```

**会生成：**
- `@pytest.mark.parametrize` 参数化版本
- 清晰的 `ids=` 参数
- 减少重复代码说明

---

#### 🔍 Skill 4：测试质量审查

**触发词：** `检查测试质量` / `review 测试` / `测试 code review`

**使用方法：**
1. 打开测试文件（或选中部分代码）
2. 在 Cursor Chat 中输入：

```
帮我 review 这个测试文件的质量
```

**会输出：**
- 总体评分（X/10）
- 问题列表（严重程度 + 位置 + 建议）
- 关键问题的重构示例

---

## 🛠️ 自定义配置

### 修改 Rules

编辑 `.cursor/rules/pytest.mdc`，常见自定义：

```markdown
# 修改测试目录（默认 tests/）
- 所有测试文件放在 `src/tests/` 目录下

# 添加团队特定 Mock 规范
- 使用 `responses` 库 mock HTTP 请求，不使用 mocker.patch requests

# 修改命名规范
- 测试函数命名使用下划线分隔的动词短语：test_should_<expected>_when_<condition>
```

### 添加新 Skill

在 `.cursor/skills/` 下新建 Markdown 文件，参考现有 skill 格式：

```markdown
# Skill: your-skill-name

## 触发方式
- "你的触发词"

## 描述
这个 skill 做什么

## Prompt 模板
请按照以下要求...
```

---

## 📋 依赖

使用本配置前，确保项目已安装：

```bash
pip install pytest pytest-mock pytest-cov
```

推荐的 `pytest.ini` / `pyproject.toml` 配置：

```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
addopts = -v --tb=short --cov=src --cov-report=term-missing
markers =
    slow: 标记为耗时测试
    integration: 标记为集成测试
    unit: 标记为单元测试
```

---

## 🤝 贡献

欢迎提交 PR 或 Issue：
- 新增 skill（如：生成 mock 测试、生成 API 测试等）
- 改进现有规范
- 添加更多语言/框架的 rules

---

## 📄 License

MIT
