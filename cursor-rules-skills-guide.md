# Cursor 项目级 Rules & Skills 使用配置指南

---

## 一、核心概念区分

| 概念 | 本质 | 作用范围 | 触发方式 |
|------|------|---------|---------|
| **Rules** | 告诉 AI "怎么做" 的约束 | 自动生效 | 被动，符合 glob 就触发 |
| **Skills** (旧称 Docs) | 告诉 AI "做什么" 的模板 | 按需调用 | 主动，用 `@` 引用或描述触发 |

---

## 二、Rules 配置详解

### 文件位置

```
your-project/
└── .cursor/
    └── rules/
        ├── general.mdc       # 通用规范
        ├── pytest.mdc        # 测试规范
        └── api-design.mdc    # API 设计规范
```

### `.mdc` 文件格式

每个 rule 文件由 **YAML frontmatter + Markdown 正文** 组成：

```markdown
---
description: 简短描述这条 rule 的作用（AI 用来判断是否应用）
globs: ["src/**/*.py", "tests/**/*.py"]   # 适用的文件匹配
alwaysApply: false   # true = 每次对话都注入，false = 按需
---

# Rule 正文
在这里写对 AI 的具体约束和规范...
```

### `alwaysApply` 的选择策略

```
alwaysApply: true   → 全局规范（代码风格、安全要求）
alwaysApply: false  → 场景规范（测试、API、数据库）
                      配合 globs 使用，只在相关文件时生效
```

### 四种 Rule 类型

```
1. Always Rule      alwaysApply: true，无 globs
   → 每次对话必然注入，适合团队通用规范

2. Auto-attached    alwaysApply: false，有 globs
   → 打开匹配文件时自动注入，适合场景规范

3. Agent-requested  只有 description，无 globs
   → AI 自行判断是否需要，适合补充性说明

4. Manual           无 description、无 globs
   → 只能手动 @引用，适合临时参考文档
```

### 实际示例

```markdown
---
description: Python 代码规范，适用于所有 Python 源文件
globs: ["**/*.py"]
alwaysApply: false
---

# Python 编码规范

## 格式
- 使用 Black 格式化，行长度 88 字符
- 类型注解必须完整（函数参数 + 返回值）
- Docstring 使用 Google 风格

## 禁止事项
- 禁止裸 except（必须指定异常类型）
- 禁止可变默认参数（`def f(x=[])` ❌）
- 禁止直接 print 调试（使用 logging）
```

---

## 三、Skills（@Docs）配置详解

> Cursor 中 Skills 本质是通过 **@Docs** 功能引入的参考文档，让 AI 在对话中调用。

### 两种使用方式

#### 方式 A：放在 `.cursor/skills/` 目录（本地项目）

```
your-project/
└── .cursor/
    └── skills/
        ├── generate-unit-test.md
        └── generate-fixture.md
```

在对话中手动 `@` 引用：
```
@generate-unit-test 为这个函数生成测试
```

#### 方式 B：通过 Cursor Settings 注册（全局/团队）

```
Cursor → Settings → Features → Docs → Add new doc
```

填入：
- **Name**：`pytest-skills`
- **URL**：`https://github.com/autel-yanfeng/cursor-pytest-skills`

注册后可在任何项目的对话中使用 `@pytest-skills`。

---

## 四、项目完整目录结构

```
your-project/
├── src/
├── tests/
├── .cursor/
│   ├── rules/
│   │   ├── python.mdc          # Python 通用规范
│   │   ├── pytest.mdc          # 测试规范（alwaysApply: false）
│   │   ├── api.mdc             # API 设计规范
│   │   └── security.mdc        # 安全规范（alwaysApply: true）
│   └── skills/
│       ├── generate-unit-test.md
│       ├── generate-fixture.md
│       ├── review-test-quality.md
│       └── generate-parametrize.md
├── .cursorignore               # 让 AI 忽略的文件
└── pyproject.toml
```

---

## 五、`.cursorignore` — 排除不需要分析的文件

```
# .cursorignore
node_modules/
.venv/
__pycache__/
*.egg-info/
dist/
htmlcov/
.coverage
*.lock
```

---

## 六、团队协作配置流程

```
1. 项目根目录创建 .cursor/ 目录
2. 编写 rules（规范） + skills（模板）
3. 提交到 Git（.cursor/ 不要加入 .gitignore）
4. 团队成员 clone 后自动生效
```

### 推荐的 `.gitignore` 设置

```gitignore
# 不要忽略 .cursor/（需要提交给团队）
# .cursor/   ← 注释掉或不加这行

# 可以忽略个人设置
.cursor/mcp.json    # 个人 MCP 配置
```

---

## 七、Rules 优先级与调试

### 优先级顺序

```
项目 Rules (.cursor/rules/)
  > 全局 Rules (Cursor Settings > Rules for AI)
    > 默认行为
```

### 查看哪些 Rules 生效

在 Cursor Chat 中输入：
```
当前对话中有哪些 rules 正在生效？
```

或打开 Cursor → **Settings → Rules** 查看已加载列表。

---

## 八、最佳实践建议

```
✅ Rule 要具体可验证，避免模糊说法
   好：使用 @pytest.mark.parametrize 合并相似测试
   差：写好的测试代码

✅ 一个 .mdc 文件只管一个关注点
   pytest.mdc 只写测试规范
   api.mdc 只写接口设计规范

✅ alwaysApply: true 要谨慎
   太多 always rule 会消耗 context 窗口

✅ Skills 写清楚触发词和输出格式
   AI 才能准确理解何时、如何使用

✅ 定期 review rules
   规范过时比没有规范更危险
```

---

## 附：pytest Rules & Skills 仓库

GitHub：https://github.com/autel-yanfeng/cursor-pytest-skills

快速安装到项目：
```bash
git clone https://github.com/autel-yanfeng/cursor-pytest-skills.git /tmp/cps
cp -r /tmp/cps/.cursor ./
```
