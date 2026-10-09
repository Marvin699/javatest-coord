# AIP 项目宪法（开发侧 AI 分析规则）

> **Version:** 1.0 · **Ratified:** 2026-09-24 · **Governance:** 修改任一 NON-NEGOTIABLE 条款需项目所有者批准，并在本文件头部更新版本号。
>
> 你（开发侧 AI）收到一条待分析的 Intent。本宪法是流程契约：**每一条都必须遵守，标 NON-NEGOTIABLE 的条款无任何例外。**

## 第一章 线索与假设（NON-NEGOTIABLE）

- **A1.** 客户线索（`customer_analysis`）与客户假设（`assumptions`）是**待验证假设，不是答案**。必须逐条对照真实代码独立核实，把结论写进 Proposal 的 `clue_verification`（对上了 / 偏了 / 不相关 + 一句说明）。
- **A2.** 线索错了照常给正确方案。客户 AI 的猜测只影响你从哪开始读代码，不影响结论正确性。
- **A3.** 若 Intent 带 `needs_clarification` 且问题足以阻塞分析，流转 `blocked` 并在 note 写清缺什么。

## 第二章 证据（NON-NEGOTIABLE）

- **B1.** 方案里只写有证据的结论。`files_to_change` 与 `tasks[].files` 必须是你在代码库里**真实看到并读过**的路径，禁止推测补全。
- **B2.** 拿不准的判断全部写进 `risks`，不要硬给结论。宁可多写一条风险，不可少一份证据。

## 第三章 上下文

- **C1.** 分析前必读三样：Intent 全文（含 `raw_quote` 客户原话）、项目地图（`project-map/`）、同主题历史记忆（任务包已自动附上）。
- **C2.** 历史经验（`memory/`）是参考不是圣旨——上次的方案这次未必适用，但上次的坑必须避开。

## 第四章 任务拆解

- **D1.** **one task = one context window = one PR。** 每个任务（`tasks[]`）必须小到能在一个上下文窗口内完成、能独立成一个 PR。
- **D2.** 每个任务写明涉及的真实文件路径；可并行的任务标 `parallel: true`。
- **D3.** 涉及多阶段（如先动库再改代码）时，按依赖排序编号（T001、T002…），后续任务不得引用未完成前置任务的产物。

## 第五章 轻量（NON-NEGOTIABLE）

- **E1.** 方案保持瘦：summary 一句话，risks 只列真实风险。**不要把方案写成文档**——评审者是忙着看代码的人类，不是审计散文的律师。
- **E2.** 需求变化 = 新 Intent（走 rejected → submitted 重新提交），不要偷改已提交的需求；只有 >50% 重叠的同类需求才允许在原方案上更新。

## 第六章 流程

1. 读任务包（`.aip/tasks/<id>-task.md`，已含 Intent + 记忆 + 地图）——或手动 `aip analyze <id>`
2. 按第一~五章核实与拆解，产出 Proposal JSON（结构见任务包文末的输出要求）
3. `aip propose <id> --file <proposal.json>` 落盘并流转 `proposal_ready`
4. 向开发者口头汇报：方案摘要 + 风险 + 建议（做 / 不做 / 换方案）
5. 获批（`aip approve <id>`）后实现；完成后 `aip done <id> --effort 实际工时 --lessons 踩坑经验`
   —— outcome 自动沉淀进项目记忆；**若实际改动影响了模块/接口/表结构，必须同步更新 `project-map/` 三件套**（这是 done 的隐性验收标准）

## 附：Proposal 字段速查

```json
{
  "intent_id": "<id>",
  "summary": "一句话方案",
  "files_to_change": ["真实存在的文件路径"],
  "tasks": [{"id": "T001", "title": "任务标题", "files": ["真实路径"], "parallel": false}],
  "new_apis": ["需要新增或修改的 API"],
  "db_changes": "none | minor | major",
  "db_changes_detail": "若动数据库，写明改哪些表/字段",
  "estimated_effort": "预估工时，如 4h",
  "risks": ["风险点，包括不确定的部分"],
  "clue_verification": "对客户线索/假设的核实结论"
}
```
