# Skill 编写最佳实践

> 学习如何编写 Claude 能发现并成功使用的有效 Skill。

好的 Skill 简洁、结构良好，并经过真实使用测试。本指南提供实用的编写决策，帮助你编写 Claude 能够发现并有效使用的 Skill。

关于 Skill 工作原理的概念背景，请参见 [Skills 概述](/en/docs/agents-and-tools/agent-skills/overview)。

## 核心原则

### 简洁是关键

[上下文窗口](https://platform.claude.com/docs/en/build-with-claude/context-windows)是一种公共资源。你的 Skill 与 Claude 需要知道的所有其他内容共享上下文窗口，包括：

* 系统 prompt
* 对话历史
* 其他 Skill 的元数据
* 你的实际请求

并非 Skill 中的每个 token 都有即时成本。在启动时，只有所有 Skill 的元数据（名称和描述）会被预加载。Claude 仅在 Skill 变得相关时才读取 SKILL.md，并且仅在需要时才读取其他文件。然而，在 SKILL.md 中保持简洁仍然很重要：一旦 Claude 加载它，每个 token 都会与会话历史和其他上下文竞争。

**默认假设**：Claude 已经很聪明了

只添加 Claude 尚未拥有的上下文。对每条信息进行质疑：

* "Claude 真的需要这个解释吗？"
* "我可以假设 Claude 知道这个吗？"
* "这段文字的 token 成本值得吗？"

**好的例子：简洁**（大约 50 个 token）：

````markdown  theme={null}
## 提取 PDF 文本

使用 pdfplumber 进行文本提取：

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**坏的例子：过于冗长**（大约 150 个 token）：

```markdown  theme={null}
## 提取 PDF 文本

PDF（便携式文档格式）文件是一种常见的文件格式，包含
文本、图像和其他内容。要从 PDF 中提取文本，你需要
使用一个库。有许多库可用于 PDF 处理，但我们
推荐 pdfplumber，因为它易于使用并且能处理大多数常见情况。
首先，你需要使用 pip 安装它。然后你可以使用下面的代码...
```

简洁版本假设 Claude 知道 PDF 是什么以及库是如何工作的。

### 设置适当的自由度

将具体程度与任务的脆弱性和可变性匹配。

**高自由度**（基于文本的指令）：

适用于：

* 多种方法都有效的情况
* 决策取决于上下文的情况
* 启发式方法指导方案的情况

示例：

```markdown  theme={null}
## 代码审查过程

1. 分析代码结构和组织
2. 检查潜在的 bug 或边界情况
3. 提出可读性和可维护性方面的改进建议
4. 验证项目规范的遵守情况
```

**中等自由度**（带有参数的伪代码或脚本）：

适用于：

* 存在首选模式的情况
* 允许一些变化的情况
* 配置影响行为的情况

示例：

````markdown  theme={null}
## 生成报告

使用此模板并根据需要自定义：

```python
def generate_report(data, format="markdown", include_charts=True):
    # 处理数据
    # 以指定格式生成输出
    # 可选择包含可视化
```
````

**低自由度**（特定脚本，少量或无参数）：

适用于：

* 操作脆弱且容易出错的情况
* 一致性至关重要的场合
* 必须遵循特定顺序的情况

示例：

````markdown  theme={null}
## 数据库迁移

精确运行此脚本：

```bash
python scripts/migrate.py --verify --backup
```

不要修改命令或添加额外的参数。
````

**类比**：把 Claude 想象成一个在道路上探索的机器人：

* **两边是悬崖的狭窄桥梁**：只有一条安全的路径。提供具体的护栏和精确的指令（低自由度）。示例：必须按精确顺序执行的数据库迁移。
* **没有危险的旷野**：很多路径都能成功。给出大致方向，信任 Claude 找到最佳路线（高自由度）。示例：上下文决定最佳方法的代码审查。

### 使用你计划用的所有模型进行测试

Skill 作为模型的附加组件，因此有效性取决于底层模型。使用你计划使用的所有模型测试你的 Skill。

**各模型的测试考虑因素**：

* **Claude Haiku**（快速、经济）：Skill 是否提供了足够的指导？
* **Claude Sonnet**（平衡）：Skill 是否清晰且高效？
* **Claude Opus**（强大的推理能力）：Skill 是否避免了过度解释？

对 Opus 完美运作的内容，对 Haiku 可能需要更多细节。如果你计划跨多个模型使用你的 Skill，目标是让指令对所有这些模型都良好运作。

## Skill 结构

<Note>
  **YAML 前置元数据**：SKILL.md 前置元数据需要两个字段：

  * `name` - Skill 的人类可读名称（最多 64 个字符）
  * `description` - Skill 做什么以及何时使用它的单行描述（最多 1024 个字符）

  有关完整的 Skill 结构细节，请参见 [Skills 概述](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### 命名规范

使用一致的命名模式，使 Skill 更易于引用和讨论。我们推荐对 Skill 名称使用**动名词形式**（动词 + -ing），因为这清楚地描述了 Skill 提供的活动或能力。

**好的命名示例（动名词形式）**：

* "Processing PDFs"
* "Analyzing spreadsheets"
* "Managing databases"
* "Testing code"
* "Writing documentation"

**可接受的替代方案**：

* 名词短语："PDF Processing"、"Spreadsheet Analysis"
* 面向操作："Process PDFs"、"Analyze Spreadsheets"

**避免**：

* 模糊的名称："Helper"、"Utils"、"Tools"
* 过于通用："Documents"、"Data"、"Files"
* 在你的 skill 集合中使用不一致的模式

一致的命名使其更容易：

* 在文档和对话中引用 Skill
* 一眼就理解 Skill 做什么
* 组织和搜索多个 Skill
* 维护一个专业的、连贯的 skill 库

### 编写有效的描述

`description` 字段启用 Skill 发现功能，应包含 Skill 做什么以及何时使用它。

<Warning>
  **始终以第三人称书写**。描述被注入到系统 prompt 中，不一致的语态可能导致发现问题。

  * **好的：** "Processes Excel files and generates reports"
  * **避免：** "I can help you process Excel files"
  * **避免：** "You can use this to process Excel files"
</Warning>

**具体化并包含关键术语**。同时包含 Skill 做什么以及触发/使用场景的具体上下文。

每个 Skill 恰好有一个 description 字段。描述对于 skill 选择至关重要：Claude 用它从可能的 100+ 个可用 Skill 中选择正确的 Skill。你的描述必须提供足够的细节，让 Claude 知道何时选择此 Skill，而 SKILL.md 的其余部分提供实施细节。

有效示例：

**PDF 处理 skill：**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel 分析 skill：**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git 提交助手 skill：**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

避免此类模糊的描述：

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### 渐进式信息披露模式

SKILL.md 作为一个概述，根据需要将 Claude 指向详细材料，就像入门指南中的目录一样。关于渐进式信息披露如何工作的解释，请参见概述中的 [Skills 如何工作](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

**实用指导：**

* 将 SKILL.md 正文保持在 500 行以下以获得最佳性能
* 接近此限制时将内容拆分为单独的文件
* 使用下面的模式有效组织指令、代码和资源

#### 可视化概览：从简单到复杂

一个基本的 Skill 从一个只包含元数据和指令的 SKILL.md 文件开始：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="显示 YAML 前置元数据和 markdown 正文的简单 SKILL.md 文件" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

随着你的 Skill 增长，你可以捆绑额外的内容，Claude 仅在需要时加载：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="捆绑额外的参考文件如 reference.md 和 forms.md。" data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

完整的 Skill 目录结构可能如下所示：

```
pdf/
├── SKILL.md              # 主要指令（触发时加载）
├── FORMS.md              # 表单填写指南（按需加载）
├── reference.md          # API 参考（按需加载）
├── examples.md           # 使用示例（按需加载）
└── scripts/
    ├── analyze_form.py   # 工具脚本（执行，不加载）
    ├── fill_form.py      # 表单填写脚本
    └── validate.py       # 验证脚本
```

#### 模式 1：带引用的高级指南

````markdown  theme={null}
---
name: PDF Processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF 处理

## 快速开始

使用 pdfplumber 提取文本：
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## 高级功能

**表单填写**：参见 [FORMS.md](FORMS.md) 获取完整指南
**API 参考**：参见 [REFERENCE.md](REFERENCE.md) 获取所有方法
**示例**：参见 [EXAMPLES.md](EXAMPLES.md) 获取常见模式
````

Claude 仅在需要时加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

#### 模式 2：按领域组织

对于包含多个领域的 Skill，按领域组织内容以避免加载无关上下文。当用户询问销售指标时，Claude 只需要读取销售相关的 schema，而不需要财务或营销数据。这可以保持 token 使用量低且上下文聚焦。

```
bigquery-skill/
├── SKILL.md（概述和导航）
└── reference/
    ├── finance.md（收入、计费指标）
    ├── sales.md（商机、管道）
    ├── product.md（API 使用、功能）
    └── marketing.md（活动、归因）
```

````markdown SKILL.md theme={null}
# BigQuery 数据分析

## 可用数据集

**财务**：收入、ARR、计费 → 参见 [reference/finance.md](reference/finance.md)
**销售**：商机、管道、账户 → 参见 [reference/sales.md](reference/sales.md)
**产品**：API 使用、功能、采用 → 参见 [reference/product.md](reference/product.md)
**营销**：活动、归因、邮件 → 参见 [reference/marketing.md](reference/marketing.md)

## 快速搜索

使用 grep 查找特定指标：

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### 模式 3：条件细节

展示基本内容，链接到高级内容：

```markdown  theme={null}
# DOCX 处理

## 创建文档

使用 docx-js 进行新文档创建。参见 [DOCX-JS.md](DOCX-JS.md)。

## 编辑文档

对于简单编辑，直接修改 XML。

**对于修订跟踪**：参见 [REDLINING.md](REDLINING.md)
**对于 OOXML 细节**：参见 [OOXML.md](OOXML.md)
```

Claude 仅在用户需要这些功能时才读取 REDLINING.md 或 OOXML.md。

### 避免深层嵌套引用

当从其他引用文件中引用文件时，Claude 可能会部分读取文件。遇到嵌套引用时，Claude 可能使用 `head -100` 等命令预览内容，而不是读取整个文件，从而导致信息不完整。

**保持引用从 SKILL.md 起仅一层深度**。所有参考文件应该直接从 SKILL.md 链接，以确保 Claude 在需要时读取完整的文件。

**坏的例子：太深**：

```markdown  theme={null}
# SKILL.md
参见 [advanced.md](advanced.md)...

# advanced.md
参见 [details.md](details.md)...

# details.md
这里是实际信息...
```

**好的例子：一层深度**：

```markdown  theme={null}
# SKILL.md

**基本用法**：[SKILL.md 中的指令]
**高级功能**：参见 [advanced.md](advanced.md)
**API 参考**：参见 [reference.md](reference.md)
**示例**：参见 [examples.md](examples.md)
```

### 为较长的参考文件添加目录结构

对于超过 100 行的参考文件，在顶部包含一个目录。这确保 Claude 即使通过部分读取预览，也能看到可用信息的完整范围。

**示例**：

```markdown  theme={null}
# API 参考

## 目录
- 认证和设置
- 核心方法（创建、读取、更新、删除）
- 高级功能（批量操作、webhooks）
- 错误处理模式
- 代码示例

## 认证和设置
...

## 核心方法
...
```

然后 Claude 可以根据需要读取完整文件或跳转到特定部分。

有关这种基于文件系统的架构如何实现渐进式信息披露的详细信息，请参见下方高级部分中的[运行时环境](#runtime-environment)部分。

## 工作流和反馈循环

### 对复杂任务使用工作流

将复杂操作分解为清晰的顺序步骤。对于特别复杂的工作流，提供一个检查清单，Claude 可以复制到其响应中，并在推进过程中勾选。

**示例 1：研究综合工作流**（适用于没有代码的 Skill）：

````markdown  theme={null}
## 研究综合工作流

复制此检查清单并跟踪你的进度：

```
研究进度：
- [ ] 步骤 1：阅读所有源文档
- [ ] 步骤 2：识别关键主题
- [ ] 步骤 3：交叉引用声明
- [ ] 步骤 4：创建结构化摘要
- [ ] 步骤 5：验证引用
```

**步骤 1：阅读所有源文档**

审查 `sources/` 目录中的每个文档。记录主要论点和支持证据。

**步骤 2：识别关键主题**

寻找来源之间的模式。哪些主题反复出现？哪些来源达成共识或有分歧？

**步骤 3：交叉引用声明**

对于每个主要声明，验证它是否出现在源材料中。记录哪个来源支持每个观点。

**步骤 4：创建结构化摘要**

按主题组织发现。包括：
- 主要声明
- 来自来源的支持证据
- 冲突的观点（如有）

**步骤 5：验证引用**

检查每个声明都引用了正确的源文档。如果引用不完整，返回步骤 3。
````

这个示例展示了工作流如何应用于不需要代码的分析任务。检查清单模式适用于任何复杂的多步骤过程。

**示例 2：PDF 表单填写工作流**（适用于有代码的 Skill）：

````markdown  theme={null}
## PDF 表单填写工作流

复制此检查清单并在完成时勾选项目：

```
任务进度：
- [ ] 步骤 1：分析表单（运行 analyze_form.py）
- [ ] 步骤 2：创建字段映射（编辑 fields.json）
- [ ] 步骤 3：验证映射（运行 validate_fields.py）
- [ ] 步骤 4：填写表单（运行 fill_form.py）
- [ ] 步骤 5：验证输出（运行 verify_output.py）
```

**步骤 1：分析表单**

运行：`python scripts/analyze_form.py input.pdf`

这将提取表单字段及其位置，保存到 `fields.json`。

**步骤 2：创建字段映射**

编辑 `fields.json` 为每个字段添加值。

**步骤 3：验证映射**

运行：`python scripts/validate_fields.py fields.json`

在继续之前修复任何验证错误。

**步骤 4：填写表单**

运行：`python scripts/fill_form.py input.pdf fields.json output.pdf`

**步骤 5：验证输出**

运行：`python scripts/verify_output.py output.pdf`

如果验证失败，返回步骤 2。
````

清晰的步骤防止 Claude 跳过关键验证。检查清单帮助 Claude 和你自己跟踪多步骤工作流的进度。

### 实现反馈循环

**常见模式**：运行验证器 → 修复错误 → 重复

此模式极大地提高了输出质量。

**示例 1：风格指南合规**（适用于没有代码的 Skill）：

```markdown  theme={null}
## 内容审查过程

1. 按照 STYLE_GUIDE.md 中的指南起草内容
2. 对照检查清单审查：
   - 检查术语一致性
   - 验证示例遵循标准格式
   - 确认所有必需部分都存在
3. 如果发现问题：
   - 记录每个问题并附上具体部分引用
   - 修订内容
   - 再次审查检查清单
4. 只有在满足所有要求后才继续
5. 完成并保存文档
```

这展示了使用参考文档而非脚本的验证循环模式。"验证器"是 STYLE\_GUIDE.md，Claude 通过阅读和比较来执行检查。

**示例 2：文档编辑过程**（适用于有代码的 Skill）：

```markdown  theme={null}
## 文档编辑过程

1. 对 `word/document.xml` 进行编辑
2. **立即验证**：`python ooxml/scripts/validate.py unpacked_dir/`
3. 如果验证失败：
   - 仔细阅读错误消息
   - 修复 XML 中的问题
   - 再次运行验证
4. **仅在验证通过后才继续**
5. 重新打包：`python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. 测试输出文档
```

验证循环可以及早发现错误。

## 内容指南

### 避免时间敏感信息

不要包含会过时的信息：

**坏的例子：时间敏感**（将会变得错误）：

```markdown  theme={null}
如果你在 2025 年 8 月之前执行此操作，请使用旧 API。
2025 年 8 月之后，请使用新 API。
```

**好的例子**（使用"旧模式"部分）：

```markdown  theme={null}
## 当前方法

使用 v2 API 端点：`api.example.com/v2/messages`

## 旧模式

<details>
<summary>旧版 v1 API（2025-08 已弃用）</summary>

v1 API 使用：`api.example.com/v1/messages`

此端点已不再受支持。
</details>
```

旧模式部分提供历史上下文，而不使主要内容混乱。

### 使用一致的术语

选择一个术语并在整个 Skill 中坚持使用：

**好的——一致**：

* 始终使用 "API endpoint"
* 始终使用 "field"
* 始终使用 "extract"

**坏的——不一致**：

* 混用 "API endpoint"、"URL"、"API route"、"path"
* 混用 "field"、"box"、"element"、"control"
* 混用 "extract"、"pull"、"get"、"retrieve"

一致性帮助 Claude 理解和遵循指令。

## 常见模式

### 模板模式

为输出格式提供模板。将严格程度与你的需求匹配。

**对于严格需求**（如 API 响应或数据格式）：

````markdown  theme={null}
## 报告结构

始终使用此精确的模板结构：

```markdown
# [分析标题]

## 执行摘要
[一个段落的关键发现概述]

## 关键发现
- 发现 1 及支持数据
- 发现 2 及支持数据
- 发现 3 及支持数据

## 建议
1. 具体可操作的建议
2. 具体可操作的建议
```
````

**对于灵活指导**（当适配有用时）：

````markdown  theme={null}
## 报告结构

这是一个合理的默认格式，但请根据分析情况做出最佳判断：

```markdown
# [分析标题]

## 执行摘要
[概述]

## 关键发现
[根据你的发现调整部分]

## 建议
[根据具体上下文量身定制]
```

根据特定分析类型按需调整部分。
````

### 示例模式

对于输出质量取决于看到示例的 Skill，提供输入/输出对，就像在普通的 prompt 中一样：

````markdown  theme={null}
## 提交消息格式

按照以下示例生成提交消息：

**示例 1：**
输入：Added user authentication with JWT tokens
输出：
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**示例 2：**
输入：Fixed bug where dates displayed incorrectly in reports
输出：
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**示例 3：**
输入：Updated dependencies and refactored error handling
输出：
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

遵循此风格：type(scope): 简短描述，然后详细说明。
````

示例比仅靠描述更能帮助 Claude 理解期望的风格和细节程度。

### 条件工作流模式

引导 Claude 通过决策点：

```markdown  theme={null}
## 文档修改工作流

1. 确定修改类型：

   **创建新内容？** → 遵循下面的"创建工作流"
   **编辑现有内容？** → 遵循下面的"编辑工作流"

2. 创建工作流：
   - 使用 docx-js 库
   - 从零构建文档
   - 导出为 .docx 格式

3. 编辑工作流：
   - 解包现有文档
   - 直接修改 XML
   - 每次更改后进行验证
   - 完成后重新打包
```

<Tip>
  如果工作流变得庞大或复杂且包含许多步骤，请考虑将其推送到单独的文件中，并告诉 Claude 根据手头任务读取适当的文件。
</Tip>

## 评估和迭代

### 先构建评估

**在编写大量文档之前创建评估。** 这确保你的 Skill 解决的是真实问题，而不是记录想象中的问题。

**评估驱动开发：**

1. **识别差距**：在没有 Skill 的情况下，让 Claude 运行代表性任务。记录具体的失败或缺失的上下文
2. **创建评估**：构建三个测试这些差距的场景
3. **建立基线**：测量 Claude 在没有 Skill 时的表现
4. **编写最小指令**：创建刚好足够的内容来解决差距并通过评估
5. **迭代**：执行评估，与基线比较，并进行优化

这种方法确保你解决的是实际问题，而不是预测可能永远不会出现的需求。

**评估结构**：

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages in the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

<Note>
  此示例演示了一个带有简单测试评分标准的数据驱动评估。我们目前不提供内置方式运行这些评估。用户可以创建自己的评估系统。评估是衡量 Skill 有效性的真相来源。
</Note>

### 与 Claude 迭代开发 Skill

最有效的 Skill 开发过程涉及 Claude 本身。与一个 Claude 实例（"Claude A"）合作，创建一个将被其他实例（"Claude B"）使用的 Skill。Claude A 帮助你设计和优化指令，而 Claude B 在真实任务中测试它们。这之所以有效，是因为 Claude 模型既理解如何编写有效的 agent 指令，也理解 agent 需要什么信息。

**创建新 Skill：**

1. **在不使用 Skill 的情况下完成任务**：使用正常的 prompting 与 Claude A 解决问题。在工作的过程中，你自然会提供上下文、解释偏好并分享过程知识。注意你反复提供的信息。

2. **识别可重用模式**：完成任务后，找出你提供的对于未来类似任务有用的上下文。

   **示例**：如果你完成了一个 BigQuery 分析，你可能提供了表名、字段定义、过滤规则（如"始终排除测试账户"）和常见查询模式。

3. **让 Claude A 创建一个 Skill**："创建一个 Skill，捕捉我们刚刚使用的这个 BigQuery 分析模式。包含表 schema、命名规范，以及关于过滤测试账户的规则。"

   <Tip>
     Claude 模型天生理解 Skill 格式和结构。你不需要特殊的系统 prompt 或"writing skills" skill 就能让 Claude 帮助创建 Skill。只需要求 Claude 创建一个 Skill，它就会生成具有适当前置元数据和正文内容的结构正确的 SKILL.md 内容。
   </Tip>

4. **审查简洁性**：检查 Claude A 是否添加了不必要的解释。问："去掉关于 win rate 是什么意思的解释——Claude 已经知道了。"

5. **改进信息架构**：让 Claude A 更有效地组织内容。例如："把它组织成表 schema 在单独的参考文件中。我们以后可能会添加更多表。"

6. **在类似任务上测试**：在相关用例中使用带有该 Skill 的 Claude B（一个加载了 Skill 的全新实例）。观察 Claude B 是否找到了正确的信息，是否正确应用了规则，以及是否成功处理了任务。

7. **基于观察迭代**：如果 Claude B 遇到困难或遗漏了什么，带着具体细节回到 Claude A："当 Claude 使用这个 Skill 时，它忘记按 Q4 过滤日期。我们应该添加关于日期过滤模式的部分吗？"

**迭代现有 Skill：**

在改进 Skill 时继续使用同样的层级模式。你在以下之间交替：

* **与 Claude A 合作**（帮助优化 Skill 的专家）
* **用 Claude B 测试**（使用 Skill 执行实际工作的 agent）
* **观察 Claude B 的行为**并将见解带回 Claude A

1. **在真实工作流中使用 Skill**：给 Claude B（已加载 Skill）分配实际任务，不是测试场景

2. **观察 Claude B 的行为**：注意它在哪些地方遇到困难、成功或做出意外选择

   **示例观察**："当我让 Claude B 做区域销售报告时，它编写了查询但忘记过滤掉测试账户，即使 Skill 提到了这条规则。"

3. **回到 Claude A 寻求改进**：分享当前的 SKILL.md 并描述你观察到的现象。问："我注意到 Claude B 在我要求区域报告时忘记过滤测试账户。Skill 提到了过滤，但可能不够突出？"

4. **审查 Claude A 的建议**：Claude A 可能建议重新组织以使规则更突出，使用更强硬的语言如"MUST filter"而不是"always filter"，或重构工作流部分。

5. **应用并测试更改**：用 Claude A 的改进更新 Skill，然后在类似请求上再次用 Claude B 测试

6. **基于使用重复**：随着遇到新场景，继续这个观察-优化-测试循环。每次迭代都基于真实的 agent 行为而非假设改进 Skill。

**收集团队反馈：**

1. 与队友分享 Skill 并观察他们的使用
2. 询问：Skill 是否在预期时激活？指令是否清晰？缺少什么？
3. 纳入反馈，解决你自己使用模式中的盲点

**为什么这种方法有效**：Claude A 理解 agent 的需求，你提供领域专业知识，Claude B 通过真实使用揭示差距，迭代优化基于观察到的行为而非假设改进 Skill。

### 观察 Claude 如何导航 Skill

在迭代 Skill 时，注意 Claude 在实践中实际如何使用它们。观察：

* **意外的探索路径**：Claude 是否以你没有预料到的顺序读取文件？这可能表明你的结构不如你想象的直观
* **错过的连接**：Claude 是否未能跟踪到重要文件的引用？你的链接可能需要更明确或更突出
* **对某些部分的过度依赖**：如果 Claude 反复读取同一个文件，考虑该内容是否应该放在主 SKILL.md 中
* **被忽略的内容**：如果 Claude 从未访问某个捆绑文件，它可能是不必要的，或者在主指令中没有得到足够的信号提示

基于这些观察而非假设进行迭代。Skill 元数据中的 'name' 和 'description' 尤其关键。Claude 在决定是否触发 Skill 响应当前任务时使用它们。确保它们清楚地描述 Skill 做什么以及何时应该使用它。

## 应避免的反模式

### 避免 Windows 风格路径

始终在文件路径中使用正斜杠，即使在 Windows 上：

* ✓ **好的**：`scripts/helper.py`、`reference/guide.md`
* ✗ **避免**：`scripts\helper.py`、`reference\guide.md`

Unix 风格路径可在所有平台上使用，而 Windows 风格路径在 Unix 系统上会导致错误。

### 避免提供过多选项

除非必要，不要呈现多种方法：

````markdown  theme={null}
**坏的例子：太多选择**（令人困惑）：
"你可以使用 pypdf、或 pdfplumber、或 PyMuPDF、或 pdf2image、或..."

**好的例子：提供默认选项**（带有逃生出口）：
"使用 pdfplumber 进行文本提取：
```python
import pdfplumber
```

对于需要 OCR 的扫描 PDF，改用 pdf2image 和 pytesseract。"
````

## 高级：包含可执行代码的 Skill

以下部分聚焦于包含可执行脚本的 Skill。如果你的 Skill 只使用 markdown 指令，请跳至[有效 Skill 检查清单](#checklist-for-effective-skills)。

### 解决问题，不要推卸

在为 Skill 编写脚本时，处理错误条件，而不是将问题推给 Claude。

**好的例子：显式处理错误**：

```python  theme={null}
def process_file(path):
    """处理文件，如果文件不存在则创建。"""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # 创建带有默认内容的文件，而不是失败
        print(f"文件 {path} 未找到，创建默认文件")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # 提供替代方案而不是失败
        print(f"无法访问 {path}，使用默认值")
        return ''
```

**坏的例子：推卸给 Claude**：

```python  theme={null}
def process_file(path):
    # 只是失败，让 Claude 想办法
    return open(path).read()
```

配置参数也应该有理由和文档，以避免"巫毒常量"（Ousterhout 定律）。如果你不知道正确的值，Claude 又怎么会知道呢？

**好的例子：自文档化**：

```python  theme={null}
# HTTP 请求通常在 30 秒内完成
# 较长的超时时间考虑了慢连接
REQUEST_TIMEOUT = 30

# 3 次重试平衡了可靠性和速度
# 大多数间歇性故障在第二次重试时解决
MAX_RETRIES = 3
```

**坏的例子：魔法数字**：

```python  theme={null}
TIMEOUT = 47  # 为什么是 47？
RETRIES = 5   # 为什么是 5？
```

### 提供工具脚本

即使 Claude 可以编写脚本，预制脚本也提供优势：

**工具脚本的好处**：

* 比生成的代码更可靠
* 节省 token（无需在上下文中包含代码）
* 节省时间（无需代码生成）
* 确保跨使用的一致性

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="在指令文件旁边捆绑可执行脚本" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

上图展示了可执行脚本如何与指令文件协同一同工作。指令文件（forms.md）引用脚本，Claude 可以在不将其内容加载到上下文中的情况下执行它。

**重要区分**：在你的指令中明确说明 Claude 应该：

* **执行脚本**（最常见）："运行 `analyze_form.py` 提取字段"
* **读取作为参考**（对于复杂逻辑）："参见 `analyze_form.py` 了解字段提取算法"

对于大多数工具脚本，执行是首选，因为它更可靠和高效。有关脚本执行如何工作的详细信息，请参见下面的[运行时环境](#runtime-environment)部分。

**示例**：

````markdown  theme={null}
## 工具脚本

**analyze_form.py**：从 PDF 提取所有表单字段

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

输出格式：
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**：检查重叠的边界框

```bash
python scripts/validate_boxes.py fields.json
# 返回："OK" 或列出冲突
```

**fill_form.py**：将字段值应用于 PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### 使用视觉分析

当输入可以渲染为图像时，让 Claude 进行分析：

````markdown  theme={null}
## 表单布局分析

1. 将 PDF 转换为图像：
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. 分析每个页面图像以识别表单字段
3. Claude 可以直观地看到字段位置和类型
````

<Note>
  在此示例中，你需要编写 `pdf_to_images.py` 脚本。
</Note>

Claude 的视觉能力有助于理解布局和结构。

### 创建可验证的中间输出

当 Claude 执行复杂的开放任务时，可能会出错。"计划-验证-执行"模式通过让 Claude 首先以结构化格式创建计划，然后在执行前用脚本验证该计划，从而及早发现错误。

**示例**：想象让 Claude 基于电子表格更新 PDF 中的 50 个表单字段。没有验证的话，Claude 可能引用不存在的字段、创建冲突的值、遗漏必需字段，或错误地应用更新。

**解决方案**：使用上面展示的工作流模式（PDF 表单填写），但添加一个中间的 `changes.json` 文件，在应用更改之前对其进行验证。工作流变为：分析 → **创建计划文件** → **验证计划** → 执行 → 验证。

**为什么这种模式有效：**

* **及早发现错误**：验证在更改应用之前发现问题
* **机器可验证**：脚本提供客观验证
* **可逆计划**：Claude 可以在计划上迭代而不触碰原始文件
* **清晰的调试**：错误消息指向具体问题

**何时使用**：批量操作、破坏性更改、复杂验证规则、高风险操作。

**实现提示**：使验证脚本详细化，提供具体的错误消息，如"字段 'signature\_date' 未找到。可用字段：customer\_name、order\_total、signature\_date\_signed"，以帮助 Claude 修复问题。

### 打包依赖项

Skill 运行在具有平台特定限制的代码执行环境中：

* **claude.ai**：可以从 npm 和 PyPI 安装包，以及从 GitHub 仓库拉取
* **Anthropic API**：没有网络访问，没有运行时的包安装

在 SKILL.md 中列出所需的包，并在[代码执行工具文档](/en/docs/agents-and-tools/tool-use/code-execution-tool)中验证它们是否可用。

### 运行时环境

Skill 运行在具有文件系统访问、bash 命令和代码执行能力的代码执行环境中。有关此架构的概念解释，请参见概述中的 [Skills 架构](/en/docs/agents-and-tools/agent-skills/overview#the-skills-architecture)。

**这如何影响你的编写：**

**Claude 如何访问 Skill：**

1. **元数据预加载**：启动时，所有 Skill YAML 前置元数据中的名称和描述被加载到系统 prompt 中
2. **文件按需读取**：Claude 使用 bash Read 工具在需要时从文件系统访问 SKILL.md 和其他文件
3. **脚本高效执行**：工具脚本可以通过 bash 执行，而无需将其完整内容加载到上下文。只有脚本的输出消耗 token
4. **大文件无上下文惩罚**：参考文件、数据或文档在被实际读取之前不会消耗上下文 token

* **文件路径很重要**：Claude 像文件系统一样导航你的 skill 目录。使用正斜杠（`reference/guide.md`），而不是反斜杠
* **描述性命名文件**：使用表示内容的名称：`form_validation_rules.md`，而不是 `doc2.md`
* **按发现组织**：按领域或功能组织目录结构
  * 好的：`reference/finance.md`、`reference/sales.md`
  * 坏的：`docs/file1.md`、`docs/file2.md`
* **捆绑全面的资源**：包含完整的 API 文档、广泛的示例、大型数据集；在访问之前没有上下文惩罚
* **对确定性操作优先使用脚本**：编写 `validate_form.py`，而不是要求 Claude 生成验证代码
* **明确执行意图**：
  * "运行 `analyze_form.py` 提取字段"（执行）
  * "参见 `analyze_form.py` 了解提取算法"（作为参考阅读）
* **测试文件访问模式**：通过使用真实请求测试来验证 Claude 能够导航你的目录结构

**示例：**

```
bigquery-skill/
├── SKILL.md（概述，指向参考文件）
└── reference/
    ├── finance.md（收入指标）
    ├── sales.md（管道数据）
    └── product.md（使用分析）
```

当用户询问收入时，Claude 读取 SKILL.md，看到对 `reference/finance.md` 的引用，并调用 bash 仅读取该文件。sales.md 和 product.md 文件保留在文件系统中，在需要之前消耗零上下文 token。这种基于文件系统的模型正是实现渐进式信息披露的基础。Claude 可以导航并有选择地加载每个任务所需的确切内容。

有关技术架构的完整细节，请参见 Skills 概述中的 [Skills 如何工作](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

### MCP 工具引用

如果你的 Skill 使用 MCP（Model Context Protocol）工具，始终使用完全限定的工具名称以避免"tool not found"错误。

**格式**：`ServerName:tool_name`

**示例**：

```markdown  theme={null}
使用 BigQuery:bigquery_schema 工具检索表 schema。
使用 GitHub:create_issue 工具创建 issues。
```

其中：

* `BigQuery` 和 `GitHub` 是 MCP 服务器名称
* `bigquery_schema` 和 `create_issue` 是这些服务器中的工具名称

没有服务器前缀，Claude 可能无法定位工具，尤其在多个 MCP 服务器可用时。

### 避免假设工具已安装

不要假设包已可用：

````markdown  theme={null}
**坏的例子：假设已安装**：
"使用 pdf 库处理文件。"

**好的例子：明确说明依赖项**：
"安装必需的包：`pip install pypdf`

然后使用它：
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## 技术说明

### YAML 前置元数据要求

SKILL.md 前置元数据需要 `name`（最多 64 个字符）和 `description`（最多 1024 个字符）字段。有关完整的结构细节，请参见 [Skills 概述](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。

### Token 预算

将 SKILL.md 正文保持在 500 行以下以获得最佳性能。如果你内容超过此限制，请使用前面描述的渐进式信息披露模式将其拆分为单独的文件。有关架构细节，请参见 [Skills 概述](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

## 有效 Skill 检查清单

在分享 Skill 之前，验证：

### 核心质量

* [ ] 描述具体且包含关键术语
* [ ] 描述包含 Skill 做什么以及何时使用它
* [ ] SKILL.md 正文在 500 行以内
* [ ] 额外细节在单独的文件中（如需要）
* [ ] 无时间敏感信息（或在"旧模式"部分）
* [ ] 整个文档术语一致
* [ ] 示例是具体的，而非抽象的
* [ ] 文件引用仅一层深度
* [ ] 适当使用渐进式信息披露
* [ ] 工作流有清晰的步骤

### 代码和脚本

* [ ] 脚本解决问题而非推卸给 Claude
* [ ] 错误处理是显式且有帮助的
* [ ] 无"巫毒常量"（所有值都有理由）
* [ ] 所需包已在指令中列出并验证可用
* [ ] 脚本有清晰的文档
* [ ] 无 Windows 风格路径（全部使用正斜杠）
* [ ] 关键操作有验证/校验步骤
* [ ] 对质量关键的任务包含反馈循环

### 测试

* [ ] 至少创建了三个评估
* [ ] 已使用 Haiku、Sonnet 和 Opus 测试
* [ ] 已使用真实使用场景测试
* [ ] 已纳入团队反馈（如适用）

## 接下来

<CardGroup cols={2}>
  <Card title="开始使用 Agent Skills" icon="rocket" href="/en/docs/agents-and-tools/agent-skills/quickstart">
    创建你的第一个 Skill
  </Card>

  <Card title="在 Claude Code 中使用 Skills" icon="terminal" href="/en/docs/claude-code/skills">
    在 Claude Code 中创建和管理 Skills
  </Card>

  <Card title="通过 API 使用 Skills" icon="code" href="/en/api/skills-guide">
    通过编程方式上传和使用 Skills
  </Card>
</CardGroup>
