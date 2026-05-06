---
phase: XX-name
plan: YY
subsystem: [主要分类]
tags: [可搜索的技术标签]
requires:
  - phase: [前置阶段]
    provides: [该阶段构建了什么]
provides:
  - [构建/交付的要点列表]
affects: [阶段名称或关键词列表]
tech-stack:
  added: [新增的库/工具]
  patterns: [架构/代码模式]
key-files:
  created: [创建的重要文件]
  modified: [修改的重要文件]
key-decisions:
  - "决策 1"
patterns-established:
  - "模式 1：描述"
duration: Xmin
completed: YYYY-MM-DD
---

# 阶段 [X]：[名称] 摘要（复杂版）

**[描述成果的实质性一句话——不是"阶段完成"或"实现已完成"]**

## 性能
- **耗时：** [时间]
- **任务：** [完成数量]
- **文件修改：** [数量]

## 成就
- [关键成果 1]
- [关键成果 2]

## 任务提交
1. **任务 1：[任务名称]** - `hash`
2. **任务 2：[任务名称]** - `hash`
3. **任务 3：[任务名称]** - `hash`

## 创建/修改的文件
- `path/to/file.ts` - 功能描述
- `path/to/another.ts` - 功能描述

## 做出的决策
[关键决策及简要理由]

## 与计划的偏差（自动修复）
[按 GSD 偏差规则记录的详细自动修复记录]

## 遇到的问题
[计划工作中遇到的问题及解决方案]

## 下一阶段准备情况
[为下一阶段准备就绪的内容]
[阻塞项或关注点]
