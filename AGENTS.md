# AGENTS.md — 甲方 AI 使用指南（AIP 协调仓库）

> 甲方（客户）的 AI 助手请阅读本文件。
> 此仓库是甲方与开发方之间的「需求协调账本」：甲方在这里提交需求（Intent），
> 开发方在这里回复实现方案（Proposal），全部以 git 提交留痕。
> 你的职责：把甲方的想法整理成合规的需求文件并提交，以及向甲方汇报进度。
> 你不需要安装任何 AIP 工具——本指南就是全部所需。

## 一、提交新需求

甲方说出一个想法后，按以下步骤操作：

1. `git pull --ff-only` 同步最新状态
2. 查看 `.aip/intents/` 目录，确定下一个编号（已有最大编号 + 1，如已有 I-001、I-002 则新需求为 I-003）
3. 参照下方模板，把甲方的想法整理成 JSON，写入 `.aip/intents/I-{编号}-{简短英文slug}.json`（slug 用 kebab-case，如 `batch-refund`）
4. `git add .aip/intents/I-xxx-xxx.json` → `git commit -m "intent: {标题}"` → `git push`

### Intent JSON 模板（照抄结构，替换内容）

```json
{
  "id": "I-003",
  "created_by": "customer-a",
  "created_at": "2026-09-30T08:00:00+08:00",
  "title": "不超过 80 字的需求标题",
  "description": "需求正文：要做什么、为什么、有什么约束",
  "priority": "medium",
  "raw_quote": "甲方表达这个需求时的原话，一字不改",
  "customer_analysis": null,
  "assumptions": [],
  "needs_clarification": [],
  "acceptance_criteria": ["可验收的标准，逐条列出"],
  "status": "submitted",
  "history": [
    {
      "at": "2026-09-30T08:00:00+08:00",
      "from_status": null,
      "to_status": "submitted",
      "by": "customer-a",
      "note": "创建"
    }
  ]
}
```

字段要求：

| 字段 | 要求 |
|---|---|
| title / description | **必填**。title ≤ 80 字；description 写清做什么、为什么、约束 |
| raw_quote | **强烈建议**。甲方原话原文——你的转述可能丢失信息 |
| acceptance_criteria | **强烈建议**。逐条写"怎么算做完" |
| priority | 可选：low / medium / high / critical，默认 medium |
| needs_clarification | 可选：你没把握、需要甲方补充说明的问题 |
| assumptions | 可选：你（或甲方）的猜测，每条含 statement（内容）与 basis（依据） |
| customer_analysis | 可选：如果你了解甲方项目，可给线索（suspected_files 等），但这只是待验证假设，不是答案 |
| created_by | 甲方的标识（如 customer-a），全流程保持一致 |
| created_at / history 的 at | ISO 8601 格式，带时区 |
| id / status / history | 按模板固定填：status 为 `submitted`，history 只有一条 from_status 为 null 的记录 |

## 二、汇报进度

读 `.aip/intents/I-*.json` 的 status 字段，翻译给甲方：

| status | 含义 |
|---|---|
| submitted | 已提交，等待开发方分析 |
| analyzing / proposal_ready | 分析中 / 方案已出，等待甲方确认 |
| approved / implementing | 甲方已确认，开发中 |
| done | 已完成 |
| rejected / blocked / canceled | 被拒绝 / 需补充信息 / 已取消 |

方案详情在 `.aip/analysis/I-{编号}-proposal.json`（只读），可摘要转述给甲方。

## 三、开发方要求补充信息时（blocked）

status 变为 `blocked` 表示开发方需要更多信息：读该 Intent 的 history 最新一条 note，
把问题转述给甲方。**甲方的答复直接通过聊天/邮件转达开发方，不要自行修改账本文件**——
状态流转由开发方维护。

## 四、禁区（必须遵守）

- 只允许**新增** `.aip/intents/` 下的 JSON 文件；不得修改或删除任何其他文件
- 不得修改已存在的 Intent（包括 status 与 history）——状态由开发方维护
- 不得改动 `.aip/` 其他目录（analysis / project-map / memory / tasks）及 `.aip/config.json`
- push 被拒绝（编号冲突或远端更新）时：`git pull --rebase` 后改用新编号重新提交；**绝不 force push**

## 五、如果配置了 AIP MCP

若环境中配置了 AIP MCP Server（工具：`submit_intent` / `list_my_intents` /
`get_intent_detail` / `get_project_map`），**优先使用这些工具**——它们自动处理编号、
格式校验与提交。本指南的流程是没有 MCP 时的等价方案。
