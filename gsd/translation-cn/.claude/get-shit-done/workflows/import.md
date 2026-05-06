# 导入工作流

外部计划导入，具有冲突检测和 agent 委托。

流程：显示横幅→解析参数（--from <path>）→路径A：加载项目上下文→读取导入文件→冲突检测（BLOCKER/WARNING/INFO）→转换为 GSD PLAN.md 格式→通过 gsd-plan-checker 验证→最终确定（更新 ROADMAP.md、STATE.md、提交）。
