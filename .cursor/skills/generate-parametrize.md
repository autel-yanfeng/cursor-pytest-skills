# Skill: generate-parametrize

**触发词：** `参数化重构` / `合并相似测试` / `用 parametrize 重构`

## 指令

将选中的重复测试函数重构为 `@pytest.mark.parametrize`：

1. 提取变化部分为参数，参数名有语义
2. 多组参数加 `ids=` 提升可读性
3. 输出：重构前后对比 + 节省行数说明
