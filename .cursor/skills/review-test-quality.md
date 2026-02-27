# Skill: review-test-quality

**触发词：** `review 测试` / `检查测试质量` / `测试 code review`

## 指令

审查选中测试文件，按以下维度输出问题列表 + 改进建议：

- [ ] AAA 结构是否清晰
- [ ] 命名是否表达意图
- [ ] 是否有重复（可 fixture/parametrize）
- [ ] 断言是否精确（避免 `assert result is not None`）
- [ ] 是否覆盖异常路径
- [ ] Mock 是否过度（不应 mock 内部逻辑）
- [ ] 测试是否互相独立

输出格式：`问题表格（严重程度/位置/建议）+ 1-2个重构示例`
