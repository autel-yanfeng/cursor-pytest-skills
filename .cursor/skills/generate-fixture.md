# Skill: generate-fixture

## 触发方式
- "生成 fixture"
- "提取公共测试数据"
- "帮我写 conftest"
- "这些测试有重复数据，帮我重构"

## 描述
分析测试文件中的重复初始化逻辑和测试数据，提取为可复用的 pytest fixture，放入 `conftest.py`。

---

## Prompt 模板

请分析以下测试代码，提取重复逻辑并生成 `conftest.py` 中的 fixture：

```
{粘贴当前测试文件代码}
```

### 生成要求

**Fixture 设计原则：**

1. **Scope 选择策略：**
   | 使用频率 | 创建代价 | 推荐 scope |
   |---------|---------|-----------|
   | 每个测试独立 | 低 | `function`（默认） |
   | 同类测试共享 | 中 | `class` |
   | 整个模块共享 | 高 | `module` |
   | 全局单例（如 DB 连接） | 很高 | `session` |

2. **Fixture 类型：**
   - **数据 fixture**：返回测试数据对象
   - **工厂 fixture**：返回一个函数，允许自定义参数（适合需要变体的场景）
   - **Mock fixture**：封装常用的 mock 设置
   - **资源 fixture**：使用 `yield` 管理生命周期（文件、连接等）

3. **文档要求：**
   - 每个 fixture 必须有 docstring，说明：提供什么数据、适用场景、scope 原因

**输出格式：**
- 完整的 `conftest.py` 文件
- 注明哪些测试函数应该替换为使用 fixture
- 如需新增依赖，列出 `pip install` 命令

---

## 示例输出

**识别到的重复模式：**
```python
# 多个测试都在重复创建 User 对象
user = User(name="Alice", age=30, email="alice@example.com")
```

**生成的 conftest.py：**
```python
import pytest
from myapp.models import User, Database


@pytest.fixture
def basic_user():
    """
    提供一个标准测试用户（Alice）。
    
    适用：需要有效 User 对象但不关心具体属性的测试。
    Scope: function（默认）- 每个测试独立，避免状态污染。
    """
    return User(name="Alice", age=30, email="alice@example.com")


@pytest.fixture
def user_factory():
    """
    用户对象工厂，允许自定义属性。
    
    适用：需要多种变体用户的测试。
    
    用法：
        def test_something(user_factory):
            admin = user_factory(name="Bob", role="admin")
    """
    def _factory(name="Test User", age=25, email="test@example.com", **kwargs):
        return User(name=name, age=age, email=email, **kwargs)
    return _factory


@pytest.fixture(scope="session")
def db_connection():
    """
    数据库连接（整个测试会话共享）。
    
    适用：需要真实 DB 连接的集成测试。
    Scope: session - DB 连接开销大，整个会话复用同一连接。
    """
    db = Database.connect("sqlite:///:memory:")
    yield db
    db.close()  # 测试结束后自动清理


@pytest.fixture
def mock_email_service(mocker):
    """
    Mock 邮件发送服务，防止测试中真实发送邮件。
    
    适用：任何触发邮件发送的测试。
    """
    return mocker.patch("myapp.services.email.send_email", return_value=True)
```
