---
name: gsd:graphify
description: "构建、查询和检查 .planning/graphs/ 中的项目知识图谱"
argument-hint: "[build|query <term>|status|diff]"
allowed-tools:
  - Read
  - Bash
  - Task
---

**停止 — 不要读取此文件。你已经正在阅读它。此 prompt 已由 Claude Code 的命令系统注入到你的上下文中。使用 Read 工具读取此文件会浪费 token。立即从步骤 0 开始执行。**

**仅 CJS（graphify）：** `graphify` 子命令未注册在 `gsd-sdk query` 上。请使用 `node /Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs graphify …`，如此命令和 `docs/CLI-TOOLS.md` 中所述。其他工具在有 handler 的地方仍可使用 `gsd-sdk query`。

## 步骤 0 -- Banner

**在任何工具调用之前**，显示此 banner：

```
GSD > GRAPHIFY
```

然后继续步骤 1。

## 步骤 1 -- 配置关卡

通过直接使用 Read 工具读取 `.planning/config.json` 来检查 graphify 是否已启用。

**不要使用 gsd-tools config get-value 命令** — 它在缺少键时会硬退出。

1. 使用 Read 工具读取 `.planning/config.json`
2. 如果文件不存在：显示下方的禁用消息并 **停止**
3. 解析 JSON 内容。检查 `config.graphify && config.graphify.enabled === true`
4. 如果 `graphify.enabled` 不是显式的 `true`：显示下方的禁用消息并 **停止**
5. 如果 `graphify.enabled` 为 `true`：继续步骤 2

**禁用消息：**

```
GSD > GRAPHIFY

知识图谱已禁用。激活方式：

  node /Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs config-set graphify.enabled true

然后运行 /gsd-graphify build 创建初始图谱。
```

---

## 步骤 2 -- 解析参数

解析 `$ARGUMENTS` 以确定操作模式：

| 参数 | 操作 |
|----------|--------|
| `build` | 启动 graphify-builder agent（步骤 3） |
| `query <term>` | 运行内联查询（步骤 2a） |
| `status` | 运行内联状态检查（步骤 2b） |
| `diff` | 运行内联差异检查（步骤 2c） |
| 无参数或未知 | 显示用法消息 |

**用法消息**（无参数或无法识别的参数时显示）：

```
GSD > GRAPHIFY

用法：/gsd-graphify <mode>

模式：
  build           构建或重建知识图谱
  query <term>    在图谱中搜索一个词
  status          显示图谱的新鲜度和统计信息
  diff            显示自上次构建以来的变化
```

### 步骤 2a -- 查询

运行：

```bash
node /Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs graphify query <term>
```

解析 JSON 输出并显示结果：
- 如果输出包含 `"disabled": true`，则显示步骤 1 中的禁用消息并 **停止**
- 如果输出包含 `"error"` 字段，则显示错误消息并 **停止**
- 如果未找到节点，显示：`未找到 '<term>' 的图谱匹配项。尝试 /gsd-graphify build 创建或重建图谱。`
- 否则，按类型分组显示匹配的节点，附带边关系和置信度层级（EXTRACTED/INFERRED/AMBIGUOUS）

显示结果后 **停止**。不要启动 agent。

### 步骤 2b -- 状态

运行：

```bash
node /Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs graphify status
```

解析 JSON 输出并显示：
- 如果 `exists: false`，显示 message 字段
- 否则显示上次构建时间、节点/边/超边计数，以及 STALE 或 FRESH 指示器

显示状态后 **停止**。不要启动 agent。

### 步骤 2c -- 差异

运行：

```bash
node /Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs graphify diff
```

解析 JSON 输出并显示：
- 如果 `no_baseline: true`，显示 message 字段
- 否则显示节点和边变更计数（已添加/已删除/已变更）

如果不存在快照，建议运行 `build` 两次（第一次创建，第二次生成差异基线）。

显示差异后 **停止**。不要启动 agent。

---

## 步骤 3 -- 构建（Agent 启动）

首先运行预检：

```
PREFLIGHT=$(node "/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs" graphify build)
```

如果预检返回 `disabled: true` 或 `error`，显示消息并 **停止**。

如果预检返回 `action: "spawn_agent"`，显示：

```
GSD > 正在启动 graphify-builder agent……
```

启动 Task：

```
Task(
  description="构建或重建项目知识图谱",
  prompt="你是 graphify-builder agent。你的工作是利用 graphify CLI 构建或重建项目知识图谱。

项目根目录：${CWD}
gsd-tools 路径：/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs

## 说明

1. **调用 graphify：**
   从项目根目录运行：
   ```
   graphify update .
   ```
   这会使用 SHA256 增量缓存构建知识图谱。
   超时：最多 5 分钟（或按 graphify.build_timeout 的配置）。

2. **验证输出：**
   检查 graphify-out/graph.json 是否存在且是包含 nodes[] 和 edges[] 数组的有效 JSON。
   如果 graphify 以非零退出或 graph.json 不可解析，输出：
   ## 图谱构建失败
   包含 stderr 输出以便调试。不要删除 .planning/graphs/ — 先前的有效图谱仍然可用。

3. **复制制品到 .planning/graphs/：**
   ```
   cp graphify-out/graph.json .planning/graphs/graph.json
   cp graphify-out/graph.html .planning/graphs/graph.html
   cp graphify-out/GRAPH_REPORT.md .planning/graphs/GRAPH_REPORT.md
   ```
   这三个文件是 query、status 和 diff 命令使用的构建输出。

4. **写入差异快照：**
   ```
   node \"/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs\" graphify build snapshot
   ```
   这会创建 .planning/graphs/.last-build-snapshot.json 用于未来的差异比较。

5. **报告构建摘要：**
   ```
   node \"/Users/ebbi/Work/harness-study/gsd/.claude/get-shit-done/bin/gsd-tools.cjs\" graphify status
   ```
   显示状态输出中的节点数、边数和超边数。

完成后，输出：## 图谱构建完成，附带摘要统计。
如果任何步骤失败，输出：## 图谱构建失败，附带详细信息。"
)
```

等待 agent 完成。

---

## 反模式

1. 不要为 query/status/diff 操作启动 agent — 这些是内联 CLI 调用
2. 不要直接修改图谱文件 — 构建 agent 负责写入
3. 不要跳过配置关卡检查
4. 不要使用 gsd-tools config get-value 进行配置关卡检查 — 它在缺少键时会退出
