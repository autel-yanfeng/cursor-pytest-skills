# Cursor 完整配置教程：Rules、Skills、MCP

> 适用版本：Cursor 0.40+
> 最后更新：2026-02

---

## 目录

1. [整体架构概览](#一整体架构概览)
2. [Rules 配置](#二rules-配置)
3. [Skills（@Docs）配置](#三skillsdocs-配置)
4. [MCP 配置](#四mcp-配置)
5. [项目级统一配置](#五项目级统一配置)
6. [团队协作最佳实践](#六团队协作最佳实践)
7. [常见问题排查](#七常见问题排查)

---

## 一、整体架构概览

```
Cursor AI 能力增强
├── Rules        → 约束 AI 行为（怎么做）
│   ├── 全局 Rules    Cursor Settings > Rules for AI
│   └── 项目 Rules    .cursor/rules/*.mdc
│
├── Skills/Docs  → 提供参考知识（知道什么）
│   ├── 本地 Skills   .cursor/skills/*.md
│   └── 远程 Docs     Cursor Settings > Docs（URL 索引）
│
└── MCP          → 扩展 AI 工具能力（能做什么）
    ├── 全局 MCP      ~/.cursor/mcp.json
    └── 项目 MCP      .cursor/mcp.json
```

---

## 二、Rules 配置

### 2.1 全局 Rules（个人通用规范）

**位置：** `Cursor → Settings → General → Rules for AI`

**适合写什么：**
- 回复语言偏好
- 代码风格底线
- 个人习惯

**示例：**
```
- 始终用中文回复
- 代码注释使用中文
- 不要生成多余的解释，直接给代码
- 变量命名使用小写下划线风格（snake_case）
```

---

### 2.2 项目 Rules（`.cursor/rules/`）

#### 文件格式

```
your-project/
└── .cursor/
    └── rules/
        ├── general.mdc
        ├── python.mdc
        ├── pytest.mdc
        └── api.mdc
```

每个 `.mdc` 文件结构：

```markdown
---
description: 这条 rule 的用途说明（AI 据此判断是否加载）
globs: ["**/*.py", "tests/**"]
alwaysApply: false
---

# 规范正文（Markdown 格式）

## 章节一
- 规则 1
- 规则 2
```

#### 四种 Rule 类型对比

| 类型 | alwaysApply | globs | 触发方式 | 适用场景 |
|------|------------|-------|---------|---------|
| Always | true | 无 | 每次对话自动注入 | 团队强制规范 |
| Auto-attached | false | 有 | 打开匹配文件时注入 | 语言/框架规范 |
| Agent-requested | false | 无，有 description | AI 自判断 | 补充说明文档 |
| Manual | false | 无，无 description | 手动 @ 引用 | 临时参考 |

#### 完整示例：`pytest.mdc`

```markdown
---
description: pytest 测试规范，生成或修改测试文件时适用
globs: ["test_*.py", "*_test.py", "conftest.py"]
alwaysApply: false
---

# pytest 测试规范

## 结构
- 使用 AAA（Arrange-Act-Assert）结构，注释标注
- 每个测试只验证一个行为
- 相关测试用 class 分组

## Fixtures
- 公共 fixture 放 conftest.py
- scope 必须显式声明

## Mock
- 使用 pytest-mock 的 mocker，禁止直接 import unittest.mock

## 断言
- 使用原生 assert，不用 assertEqual
- 异常用 pytest.raises(Type, match="pattern")
- 浮点用 pytest.approx

## 命名
- test_<function>_<scenario>_<expected>
```

#### 完整示例：`python.mdc`

```markdown
---
description: Python 通用编码规范
globs: ["**/*.py"]
alwaysApply: false
---

# Python 编码规范

## 格式
- Black 格式化，行长 88
- 类型注解完整（参数 + 返回值）
- Docstring 使用 Google 风格

## 禁止
- 禁止裸 except
- 禁止可变默认参数
- 禁止直接 print（用 logging）

## 导入顺序
1. 标准库
2. 第三方库
3. 本地模块
（每组之间空一行）
```

---

### 2.3 Rules 优先级

```
项目 .cursor/rules/  >  全局 Settings Rules  >  默认行为
```

**调试技巧：** 在 Cursor Chat 输入：
```
列出当前对话中生效的所有 rules
```

---

## 三、Skills（@Docs）配置

### 3.1 本地 Skills（项目内）

#### 文件位置

```
your-project/
└── .cursor/
    └── skills/
        ├── generate-unit-test.md
        ├── generate-fixture.md
        └── review-code.md
```

#### 使用方式

在 Cursor Chat 中输入 `@` 触发引用：
```
@generate-unit-test 为这个函数生成测试
```

#### Skill 文件编写规范

```markdown
# Skill: skill-name

## 触发方式
- "触发词一"
- "触发词二"

## 描述
这个 skill 的功能说明

## Prompt 模板

请按照以下要求处理：{粘贴代码}

### 要求
1. 要求一
2. 要求二

### 输出格式
- 格式说明

## 示例
（可选：输入/输出示例）
```

---

### 3.2 远程 Docs（全局注册）

**位置：** `Cursor → Settings → Features → Docs → + Add new doc`

**用途：** 将官方文档、GitHub 仓库、内部 Wiki 索引给 AI

**配置步骤：**

1. 点击 `Add new doc`
2. 输入 URL（支持官方文档站、GitHub README 等）
3. 等待 Cursor 爬取并建立索引
4. 在对话中用 `@文档名` 引用

**常用示例：**

```
名称: FastAPI Docs       URL: https://fastapi.tiangolo.com
名称: pytest Docs        URL: https://docs.pytest.org
名称: 团队规范            URL: https://github.com/your-org/standards
名称: cursor-pytest      URL: https://github.com/autel-yanfeng/cursor-pytest-skills
```

**使用方式：**
```
@FastAPI Docs 帮我写一个带认证的路由
@cursor-pytest 为这个函数生成测试
```

---

### 3.3 Skills vs Rules 选择原则

```
用 Rules：
  ✅ 每次都要遵守的约束（命名规范、禁止事项）
  ✅ 希望 AI 自动感知，无需手动触发

用 Skills：
  ✅ 特定任务的操作模板（生成测试、review 代码）
  ✅ 需要较长 prompt 才能描述清楚的任务
  ✅ 希望复用的对话模式
```

---

## 四、MCP 配置

### 4.1 什么是 MCP

MCP（Model Context Protocol）是一种标准协议，让 Cursor AI 可以调用外部工具和服务，例如：
- 查询数据库
- 读写文件系统
- 调用 API
- 执行代码
- 访问浏览器

### 4.2 MCP 配置文件位置

```
~/.cursor/mcp.json          # 全局 MCP（所有项目可用）
.cursor/mcp.json            # 项目级 MCP（仅当前项目）
```

### 4.3 配置格式

```json
{
  "mcpServers": {
    "服务名称": {
      "command": "启动命令",
      "args": ["参数1", "参数2"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

### 4.4 常用 MCP 服务配置示例

#### 文件系统操作

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/yourname/projects"
      ]
    }
  }
}
```

#### 数据库查询（SQLite）

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "uvx",
      "args": [
        "mcp-server-sqlite",
        "--db-path",
        "/path/to/your/database.db"
      ]
    }
  }
}
```

#### GitHub 操作

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_your_token"
      }
    }
  }
}
```

#### Brave 搜索

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "your-brave-api-key"
      }
    }
  }
}
```

#### Puppeteer（浏览器自动化）

```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

#### 完整多服务配置示例

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/yourname/projects"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    },
    "sqlite": {
      "command": "uvx",
      "args": ["mcp-server-sqlite", "--db-path", "./data/app.db"]
    }
  }
}
```

### 4.5 启用 MCP 步骤

1. 编辑 `~/.cursor/mcp.json`（全局）或 `.cursor/mcp.json`（项目）
2. 重启 Cursor 或点击 MCP 设置页的刷新按钮
3. 在 Chat 中验证：输入 `可以使用哪些工具？`
4. AI 会列出可用的 MCP 工具

### 4.6 在 Cursor Settings 中查看 MCP 状态

```
Cursor → Settings → MCP
```

显示所有已配置的 MCP 服务及其状态（✅ 运行中 / ❌ 错误）

### 4.7 MCP 安全注意事项

```
⚠️  生产数据库只给只读权限
⚠️  API Token 不要提交到 Git（用环境变量）
⚠️  filesystem MCP 只授权必要目录
⚠️  项目级 mcp.json 加入 .gitignore（含敏感信息时）
```

---

## 五、项目级统一配置

### 完整目录结构

```
your-project/
├── .cursor/
│   ├── rules/
│   │   ├── python.mdc          # Python 规范（Auto-attached）
│   │   ├── pytest.mdc          # 测试规范（Auto-attached）
│   │   ├── api.mdc             # API 规范（Auto-attached）
│   │   └── security.mdc        # 安全规范（Always）
│   ├── skills/
│   │   ├── generate-unit-test.md
│   │   ├── generate-fixture.md
│   │   ├── review-test-quality.md
│   │   └── generate-parametrize.md
│   └── mcp.json                # 项目级 MCP（⚠️ 酌情加入 .gitignore）
├── .cursorignore
└── ...项目文件
```

### `.cursorignore` 示例

```
# 不让 AI 分析的目录
node_modules/
.venv/
__pycache__/
*.egg-info/
dist/
build/
htmlcov/
.coverage
*.lock
*.log
```

### `.gitignore` 建议

```gitignore
# .cursor/ 目录大部分应提交，团队共享
# 但以下文件含个人/敏感信息，不提交：
.cursor/mcp.json
```

---

## 六、团队协作最佳实践

### 分层管理策略

```
层级           位置                    管理者        内容
─────────────────────────────────────────────────────────
个人全局       Cursor Settings         个人          语言偏好、个人习惯
              ~/.cursor/mcp.json

团队共享       .cursor/rules/          团队 Git      编码规范、测试规范
              .cursor/skills/

项目私有       .cursor/mcp.json        个人/CI       数据库连接、API Key
              (加入 .gitignore)
```

### 推荐的 Rules 文件规划

```
rules/
├── always/
│   └── team-standards.mdc    # alwaysApply: true，团队铁律
└── auto/
    ├── python.mdc            # Python 文件自动触发
    ├── pytest.mdc            # 测试文件自动触发
    ├── api.mdc               # API 文件自动触发
    └── frontend.mdc          # 前端文件自动触发
```

### 新成员接入流程

```bash
# 1. 克隆项目（.cursor/ 已包含在仓库中）
git clone https://github.com/your-org/your-project.git

# 2. 配置个人 MCP（可选）
cp .cursor/mcp.example.json ~/.cursor/mcp.json
# 编辑填入个人 API Key

# 3. 在 Cursor Settings > Docs 添加团队文档（可选）
# URL: https://github.com/your-org/cursor-skills

# 4. 开始使用，Rules 自动生效
```

---

## 七、常见问题排查

### Q1：Rules 没有生效？

**排查步骤：**
1. 检查 `.mdc` 文件的 YAML frontmatter 格式是否正确
2. 检查 `globs` 路径是否匹配当前文件
3. 在 Chat 中问：`当前有哪些 rules 生效？`
4. 重启 Cursor

### Q2：MCP 服务启动失败？

**排查步骤：**
1. 打开 `Cursor → Settings → MCP` 查看错误信息
2. 确认依赖已安装：`npx` 需要 Node.js，`uvx` 需要 Python
3. 检查 JSON 格式是否合法（用 jsonlint 验证）
4. 手动运行命令测试：`npx -y @modelcontextprotocol/server-filesystem /tmp`

### Q3：@Docs 引用找不到？

**排查步骤：**
1. 确认 Docs 已在 Settings 中添加并完成索引（等待爬取完成）
2. 本地 Skills 文件确认放在 `.cursor/skills/` 目录
3. 文件名不含特殊字符

### Q4：Rules 消耗太多 context？

**优化方案：**
- 减少 `alwaysApply: true` 的 rule 数量
- 将长规范拆分为多个小文件，按需触发
- 用 Agent-requested 类型代替 Always

### Q5：如何验证 MCP 工具可用？

在 Cursor Chat 中输入：
```
你现在可以使用哪些 MCP 工具？列出来
```

---

## 附录：快速参考卡

### Rules frontmatter 速查

```yaml
---
description: "一句话说明用途"          # Agent-requested 类型必填
globs: ["**/*.py"]                    # 文件匹配模式
alwaysApply: true/false               # 是否每次对话注入
---
```

### MCP 配置速查

```json
{
  "mcpServers": {
    "<name>": {
      "command": "<cmd>",
      "args": ["<arg1>"],
      "env": { "KEY": "value" }
    }
  }
}
```

### 常用 MCP 包速查

| 功能 | 包名 |
|------|------|
| 文件系统 | `@modelcontextprotocol/server-filesystem` |
| GitHub | `@modelcontextprotocol/server-github` |
| SQLite | `mcp-server-sqlite`（uvx） |
| Brave 搜索 | `@modelcontextprotocol/server-brave-search` |
| 浏览器 | `@modelcontextprotocol/server-puppeteer` |
| PostgreSQL | `@modelcontextprotocol/server-postgres` |
| Slack | `@modelcontextprotocol/server-slack` |

---

## 相关资源

- Cursor 官方文档：https://docs.cursor.com
- MCP 协议官网：https://modelcontextprotocol.io
- MCP 服务市场：https://github.com/modelcontextprotocol/servers
- pytest Skills 仓库：https://github.com/autel-yanfeng/cursor-pytest-skills
