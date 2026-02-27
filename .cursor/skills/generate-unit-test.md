# Skill: generate-unit-test

## 触发方式
- "为 [函数名] 生成测试"
- "写测试用例"
- "生成单元测试"
- "帮我测试这个函数"

## 描述
为指定的 Python 函数或类生成完整、可运行的 pytest 单元测试文件。

---

## Prompt 模板

请为以下代码生成 pytest 单元测试：

```
{粘贴目标函数或类代码}
```

### 生成要求

**文件结构：**
- 文件名：`tests/test_<当前模块名>.py`
- 顶部包含所有必要的 import
- 相关测试用 `class Test<FunctionName>` 分组

**必须覆盖的场景（每类至少1个）：**

| 场景类型 | 说明 | 标记 |
|---------|------|------|
| ✅ 正常路径 | 典型有效输入，验证预期输出 | 无 |
| 🔲 边界值 | 空值、零、最大/最小值、空列表等 | 无 |
| ❌ 异常路径 | 无效类型、越界、业务异常 | 无 |
| 🔄 参数化 | 多组相似输入合并为一个测试 | `@pytest.mark.parametrize` |

**代码规范：**
- 使用 AAA 三段式，注释标注 `# Arrange` `# Act` `# Assert`
- 需要 mock 的地方使用 `mocker` fixture（来自 pytest-mock）
- 浮点比较使用 `pytest.approx`
- 异常断言使用 `pytest.raises(ErrorType, match="pattern")`

**命名格式：**
```
test_<function>_<scenario>_<expected>
# 例：test_divide_by_zero_raises_value_error
```

---

## 示例输出

**输入函数：**
```python
def divide(a: float, b: float) -> float:
    """两数相除，除数为零时抛出 ValueError"""
    if b == 0:
        raise ValueError("除数不能为零")
    return a / b
```

**期望生成：**
```python
import pytest
from mymodule import divide


class TestDivide:
    """divide 函数测试套件"""

    @pytest.mark.parametrize("a, b, expected", [
        (10, 2, 5.0),
        (-6, 3, -2.0),
        (0, 5, 0.0),
        (7, 2, 3.5),
    ])
    def test_divide_normal_returns_correct_result(self, a, b, expected):
        # Arrange - 参数已通过 parametrize 传入

        # Act
        result = divide(a, b)

        # Assert
        assert result == pytest.approx(expected)

    def test_divide_large_numbers_returns_correct_result(self):
        # Arrange
        a, b = 1e15, 1e10

        # Act
        result = divide(a, b)

        # Assert
        assert result == pytest.approx(1e5)

    def test_divide_by_zero_raises_value_error(self):
        # Arrange
        a, b = 10, 0

        # Act & Assert
        with pytest.raises(ValueError, match="除数不能为零"):
            divide(a, b)

    def test_divide_by_negative_zero_raises_value_error(self):
        # Arrange
        a, b = 5, -0

        # Act & Assert
        with pytest.raises(ValueError, match="除数不能为零"):
            divide(a, b)
```
