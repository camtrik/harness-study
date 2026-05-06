<purpose>安全的 git revert 工作流。使用阶段清单回滚 GSD 阶段或计划提交，并附依赖检查和确认门。使用 git revert --no-commit（绝不使用 git reset）以保留历史。</purpose>
<process>解析参数（--last N / --phase NN / --plan NN-MM）→收集提交→依赖检查→确认回滚→执行回滚（git revert --no-commit，脏树守卫）→摘要。</process>
