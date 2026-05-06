---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# 编写计划

## 概述

编写全面的实施计划，假设工程师对我们的代码库零上下文，且品味 questionable。记录他们需要知道的一切：每个任务要触摸哪些文件、代码、测试、他们可能需要检查的文档、如何测试它。将整个计划作为小任务提供给他们。DRY。YAGNI。TDD。频繁提交。

假设他们是熟练的开发人员，但几乎不了解我们的工具集或问题域。假设他们不太了解良好的测试设计。

**在开始时宣布：** "我正在使用 writing-plans skill 来创建实施计划。"

**上下文：** 如果在隔离的 worktree 中工作，它应该是在执行时通过 `superpowers:using-git-worktrees` skill 创建的。

**将计划保存到：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （用户对计划位置的偏好覆盖此默认值）

## 范围检查

如果 spec 涵盖多个独立子系统，应该在头脑风暴期间将其分解为子项目 spec。如果没有，建议将其拆分为单独的计划——每个子系统一个。每个计划都应该产生独立的、可测试的软件。

## 文件结构

在定义任务之前，先规划将创建或修改哪些文件，以及每个文件负责什么。这是将分解决策锁定的地方。

- 设计具有清晰边界和明确定义接口的单元。每个文件应有一个清晰的职责。
- 你对可以一次性保持在上下文中的代码推理最好，当文件专注时你的编辑更可靠。更喜欢小的、专注的文件，而不是做太多的大文件。
- 一起变化的文件应该在一起。按职责拆分，而不是按技术层。
- 在现有代码库中，遵循既定模式。如果代码库使用大文件，不要单方面重构——但如果你修改的文件已经变得笨拙，在计划中包含拆分是合理的。

此结构通知任务分解。每个任务都应该产生独立且有意义的更改。

## 小任务粒度

**每个步骤是一个操作（2-5 分钟）：**
- "编写失败的测试" - 步骤
- "运行它以确保失败" - 步骤
- "编写使测试通过的最小代码" - 步骤
- "运行测试并确保通过" - 步骤
- "提交" - 步骤

## 计划文档标头

**每个计划必须以此标头开始：**

```markdown
# [功能名称] 实施计划

> **对于 agent 工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 来逐任务实施此计划。步骤使用复选框（`- [ ]`）语法进行跟踪。

**目标：** [一句话描述此计划构建的内容]

**架构：** [2-3 句话关于方法]

**技术栈：** [关键技术/库]

---
```

## 任务结构

````markdown
### 任务 N：[组件名称]

**文件：**
- 创建：`exact/path/to/file.py`
- 修改：`exact/path/to/existing.py:123-145`
- 测试：`tests/exact/path/to/test.py`

- [ ] **步骤 1：编写失败的测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **步骤 2：运行测试以验证失败**

运行：`pytest tests/path/test.py::test_name -v`
预期：FAIL 并显示 "function not defined"

- [ ] **步骤 3：编写最小实现**

```python
def function(input):
    return expected
```

- [ ] **步骤 4：运行测试以验证通过**

运行：`pytest tests/path/test.py::test_name -v`
预期：PASS

- [ ] **步骤 5：提交**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 无占位符

每个步骤必须包含工程师需要的实际内容。这些都是**计划失败**——永远不要写它们：
- "TBD"、"TODO"、"稍后实施"、"填写详细信息"
- "添加适当的错误处理" / "添加验证" / "处理边界情况"
- "为上述内容编写测试"（没有实际测试代码）
- "类似于任务 N"（重复代码——工程师可能按乱序阅读任务）
- 描述做什么但不显示如何做的步骤（代码步骤需要代码块）
- 引用任何任务中未定义的类型、函数或方法

## 记住
- 始终使用精确的文件路径
- 每个步骤的完整代码——如果步骤更改代码，显示代码
- 带有预期输出的精确命令
- DRY、YAGNI、TDD、频繁提交

## 自我审查

编写完整计划后，用全新的眼光查看 spec 并对照它检查计划。这是一个你自己运行的检查表——而不是 subagent dispatch。

**1. Spec 覆盖：** 浏览 spec 中的每个部分/需求。你能指向一个实现它的任务吗？列出任何空白。

**2. 占位符扫描：** 在计划中搜索危险信号——上述"无占位符"部分中的任何模式。修复它们。

**3. 类型一致性：** 你在后续任务中使用的类型、方法签名和属性名称是否与你在早期任务中定义的匹配？任务 3 中调用的函数 `clearLayers()` 但任务 7 中是 `clearFullLayers()` 是一个 bug。

如果发现问题，内联修复它们。无需重新审查——只需修复并继续。如果你发现没有任务的 spec 需求，添加任务。

## 执行交接

保存计划后，提供执行选择：

**"计划完成并保存到 `docs/superpowers/plans/<filename>.md`。两种执行选项：**

**1. Subagent 驱动（推荐）** - 我为每个任务 dispatch 一个新的 subagent，在任务之间审查，快速迭代

**2. 内联执行** - 在此会话中使用 executing-plans 执行任务，批量执行并带有检查点

**哪种方法？"**

**如果选择 Subagent 驱动：**
- **必需子技能：** 使用 superpowers:subagent-driven-development
- 每任务新 subagent + 两阶段审查

**如果选择内联执行：**
- **必需子技能：** 使用 superpowers:executing-plans
- 带检查点的批量执行以供审查
