<purpose>
用于失败任务验证的自主修复操作器。当任务不满足其完成标准时由 execute-plan 调用。在升级给用户之前提出并尝试结构化修复。
</purpose>
<inputs>
FAILED_TASK、ERROR、PLAN_CONTEXT 和 REPAIR_BUDGET（默认：2）。
</inputs>
<repair_directive>
选择一种修复策略：RETRY（方法正确但执行失败）、DECOMPOSE（任务太粗粒度，拆分为子步骤，最多 3 个）、PRUNE（当前约束下不可行，跳过并说明理由）、ESCALATE（修复预算耗尽或需要架构决策）。
</repair_directive>
<process>
<step name="diagnose">分析错误并选择策略。</step>
<step name="execute_retry">应用调整并重试。</step>
<step name="execute_decompose">拆分为子任务并依次执行。</step>
<step name="execute_prune">跳过并记录理由。</step>
<step name="execute_escalate">提交给用户并提供完整修复历史。</step>
</process>
