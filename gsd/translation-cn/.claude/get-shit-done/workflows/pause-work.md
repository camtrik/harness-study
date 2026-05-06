<purpose>
创建结构化的 `.planning/HANDOFF.json` 和 `.continue-here.md` 交接文件，以在会话之间保留完整的工作状态。
</purpose>
<process>
<step name="detect">检测正在暂停的工作类型（阶段/spike/sketch/审议/研究/默认）并确定交接目标路径。</step>
<step name="gather">收集完整状态（当前位置、已完成工作、剩余工作、决策、阻塞项、待处理的人工操作、后台进程、修改的文件、阻塞约束——包括严重级别 blocking 或 advisory）。</step>
<step name="write_structured">将结构化交接写入 `.planning/HANDOFF.json`。</step>
<step name="write">将人类可读的交接写入检测到的目标路径（例如 `.planning/phases/XX-name/.continue-here.md`），包含 BLOCKING CONSTRAINTS、Critical Anti-Patterns、当前状态、已完成工作、剩余工作等部分。</step>
<step name="commit">作为 WIP 提交。</step>
</process>
