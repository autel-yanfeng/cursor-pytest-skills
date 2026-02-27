# Skill: generate-parametrize

## 触发方式
- "帮我参数化这些测试"
- "这几个测试能合并吗"
- "生成参数化测试"
- "用 parametrize 重构"

## 描述
将多个结构相似的测试函数合并为 `@pytest.mark.parametrize` 参数化测试，减少重复代码。

---

## Prompt 模板

请将以下重复的测试函数重构为参数化测试：

```
{粘贴多个相似测试函数}
```

### 重构要求

1. **识别相似模式**：提取变化的部分作为参数
2. **参数命名**：参数名清晰描述含义，不用 `a, b, c`
3. **添加 ids**：复杂参数组添加 `ids=` 参数提升可读性
4. **保留语义**：参数化后的测试名仍能表达测试意图
5. **嵌套参数化**：多维度变化时考虑 `@pytest.mark.parametrize` 叠加

### 输出格式
- 重构前：原始代码（注释说明问题）
- 重构后：参数化版本
- 改进说明：减少了多少行代码，提升了什么

---

## 示例

**重构前（重复代码）：**
```python
def test_is_adult_age_18_returns_true(self):
    assert is_adult(18) == True

def test_is_adult_age_30_returns_true(self):
    assert is_adult(30) == True

def test_is_adult_age_17_returns_false(self):
    assert is_adult(17) == False

def test_is_adult_age_0_returns_false(self):
    assert is_adult(0) == False
```

**重构后：**
```python
@pytest.mark.parametrize("age, expected", [
    (18, True),   # 恰好成年
    (30, True),   # 典型成年
    (17, False),  # 未成年
    (0, False),   # 边界：零岁
], ids=["exact_adult", "typical_adult", "minor", "zero_age"])
def test_is_adult_returns_correct_result(self, age, expected):
    # Act
    result = is_adult(age)
    # Assert
    assert result == expected
```
