# Skill: generate-unit-test

**触发词：** `生成单元测试` / `为 [函数] 生成测试` / `写测试用例`

## 指令

为选中代码生成 pytest 单元测试：

1. 文件：`tests/test_<module>.py`
2. 分组：`class Test<FunctionName>`
3. 覆盖：正常路径 + 边界值 + 异常路径
4. 相似用例用 `@pytest.mark.parametrize`
5. 外部依赖用 `mocker` fixture mock
6. 每个函数头部注明测试场景
