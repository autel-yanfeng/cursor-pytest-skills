# Skill: generate-fixture

**触发词：** `生成 fixture` / `提取公共测试数据` / `帮我写 conftest`

## 指令

分析当前测试文件，提取重复初始化逻辑，生成 `conftest.py` fixture：

1. scope 选择：function（默认）/ class / module / session（按使用频率和创建代价）
2. 有副作用的用 `yield` + 清理逻辑
3. 需要变体的用工厂模式（返回函数）
4. 每个 fixture 写一行 docstring 说明用途
