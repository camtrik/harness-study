<purpose>
自主审计到修复流水线。运行审计，解析发现项，将每项分类为可自动修复 vs 仅可手动修复，启动 executor agent 处理可修复项，每次修复后运行测试，并以原子方式提交，附上发现项 ID 以便追踪。
</purpose>

<available_agent_types>
- gsd-executor — 执行一个具体的、范围明确的代码更改
</available_agent_types>

<process>

<step name="parse-arguments">
从用户调用中提取标志：
- `--max N` — 最多修复的发现项数量（默认：**5**）
- `--severity high|medium|all` — 处理的最低严重级别（默认：**medium**）
- `--dry-run` — 仅分类发现项而不修复（只显示分类表）
- `--source <audit>` — 运行哪个审计（默认：**audit-uat**）

验证 `--source` 是受支持的审计。当前支持：
- `audit-uat`

如果 `--source` 不受支持，报错停止：
```
Error: Unsupported audit source "{source}". Supported sources: audit-uat
```
</step>

<step name="run-audit">
调用源审计命令并捕获输出。对于 `audit-uat` 源，读取现有的 UAT 和验证文件以提取发现项并将其解析为结构化记录（ID、描述、严重级别、文件引用）。
</step>

<step name="classify-findings">
将每个发现项分类为：
- **auto-fixable** — 明确的代码更改、引用了特定文件、可测试的修复
- **manual-only** — 需要设计决策、范围模糊、架构更改、需要用户输入
- **skip** — 严重级别低于 `--severity` 阈值

分类启发式方法（不确定时倾向于 manual-only）：
- Auto-fixable 信号：引用特定文件路径和行号、描述缺失的测试或断言、缺失的导出/错误的导入路径/标识符拼写错误、清晰的单文件更改且预期行为明确
- Manual-only 信号：使用"consider"、"evaluate"、"design"、"rethink"等词、需要新架构或 API 更改、范围模糊或有多种有效方案、需要用户输入或设计决策、影响多个子系统的横切关注点、没有明确修复方案的性能或可扩展性问题

**不确定时，始终分类为 manual-only。**
</step>

<step name="present-classification">
显示分类表（包含发现项编号、描述、严重级别、分类和原因）。如果指定了 `--dry-run`，在此停止并退出。
</step>

<step name="fix-loop">
对于每个 **auto-fixable** 发现项（最多 `--max` 项，按严重级别降序排列）：

a. 启动 executor agent 执行修复
b. 运行测试
c. 如果测试通过 — 原子提交（提交消息必须包含发现项 ID）
d. 如果测试失败 — 回滚更改，将发现项标记为 `fix-failed`，停止流水线。测试失败表明代码库可能处于意外状态，因此流水线必须停止以避免连锁问题。
</step>

<step name="report">
呈现最终摘要（包含总发现项数、可自动修复数、仅可手动修复数、已修复数、失败数以及详细的状态表）。
</step>

</process>

<success_criteria>
- Auto-fixable findings processed sequentially until --max reached or a test failure stops the pipeline
- Tests pass after each committed fix (no broken commits)
- Failed fixes are reverted cleanly (no partial changes left)
- Pipeline stops after the first test failure (no cascading fixes)
- Every commit message contains the finding ID
- Manual-only findings are surfaced for developer attention
- --dry-run produces a useful standalone classification table
</success_criteria>
