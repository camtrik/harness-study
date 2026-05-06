<purpose>根据项目贡献模板审查所有开放的 GitHub Issues 和 PR。生成显示每个项目合规状态的结构化报告，标记缺失的必填字段，识别标签差距，并可选择采取行动（标签、评论、关闭）。</purpose>
<process>预检（gh CLI、仓库检测、标志解析）→获取 Issues→审查 Issues（根据模板检查清单）→获取 PR→审查 PR（根据模板检查清单）→检查门（issue-first规则）→生成报告→自动操作（--label/--close-incomplete）。</process>
