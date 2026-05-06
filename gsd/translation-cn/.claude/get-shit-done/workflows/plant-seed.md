<purpose>
将一个前瞻性想法捕获为带有触发条件的结构化种子文件。种子在新的里程碑范围匹配触发条件时自动浮现。
</purpose>
<process>
<step name="parse_idea">解析想法摘要。</step>
<step name="create_seed_dir">创建 seeds 目录。</step>
<step name="gather_context">通过聚焦的问题收集上下文（触发条件、原因、范围）。</step>
<step name="collect_breadcrumbs">在代码库中搜索相关引用。</step>
<step name="write_seed">写入 SEED-{NNN}-{slug}.md 文件，包含 frontmatter 和结构化内容。</step>
<step name="commit_seed">提交到 git。</step>
</process>
