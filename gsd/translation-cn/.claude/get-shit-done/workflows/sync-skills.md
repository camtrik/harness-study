# sync-skills — 跨运行时 GSD 技能同步

将受管理的 `gsd-*` 技能目录从一个规范运行时的技能根同步到一个或多个目标运行时技能根。在一个运行时上进行 gsd-update 后保持多运行时安装对齐。

参数：`--from <runtime>`（必填）、`--to <runtime|all>`（必填）、`--dry-run`（默认开启）、`--apply`。

流程：解析参数→解析技能根→计算每个目标的差异（CREATE/UPDATE/REMOVE/SKIP）→打印差异报告→如果 --apply 则执行同步。

安全规则：仅操作 `gsd-*` 目录、dry-run 为默认、源根必须存在、不进行跨运行时内容转换。
