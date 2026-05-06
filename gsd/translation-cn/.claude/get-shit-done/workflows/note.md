<purpose>
零摩擦想法捕获。一次 Write 调用，一条确认行。无问题，无提示。
</purpose>
<process>
<step name="storage_format">笔记以 markdown 文件存储（项目范围：`.planning/notes/`，全局范围：全局 notes/ 目录）。</step>
<step name="parse_subcommand">解析子命令（append/list/promote）。</step>
<step name="append">创建带时间戳的笔记文件。不修改文本、不提问。</step>
<step name="list">显示两个范围的笔记。排除已提升的活跃计数。</step>
<step name="promote">将笔记转换为待办事项。需要 .planning/ 目录。</step>
</process>
