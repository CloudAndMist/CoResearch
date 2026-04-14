# CLAUDE.md — 研究 Wiki 工作流定义

> 这是整个知识库的"宪法"。LLM 每次操作前都应先阅读此文件，了解知识库的结构和规则。

---

## 知识库概述

- **维护者**：[请填写你的名字]
- **研究方向**：[请填写：NLP / CV / GNN / 多模态 / 其他]
- **导师**：[请填写导师姓名]
- **研究组**：[请填写研究组名称]
- **创建时间**：[YYYY-MM-DD]

---

## 目录结构

```
local/
├── raw/                    # 原始资料（PDF、网页剪藏、代码等）
│   ├── papers/             # PDF 论文原文
│   ├── clips/              # 网页/推文剪藏
│   ├── experiments/        # 实验原始数据
│   └── meetings/           # 会议录音/草稿
└── wiki/                   # LLM 编译后的结构化知识页（此目录同步至飞书）
    ├── papers/             # 论文结构化笔记
    ├── concepts/           # 概念定义与解释
    ├── experiments/        # 实验记录摘要
    └── meetings/           # 组会记录整理
```

---

## 三大操作规范

### Ingest（摄取新内容）

将新材料转化为结构化 Wiki 页面的流程：

**PDF 论文 → `wiki/papers/`**
```
Prompt 模板：
"请阅读这篇论文，提取以下信息并以 Markdown 格式输出：
1. 标题、作者、年份、发表期刊/会议
2. 研究问题（一句话）
3. 核心方法（2-3个要点）
4. 主要贡献（3点）
5. 实验结论（最重要的2-3个数字/结果）
6. 局限性
7. 与我研究方向的关联
输出文件名格式：YYYY-作者-关键词.md
填充 YAML frontmatter（type: paper, share: false）"
```

**网页/链接 → `wiki/concepts/`**
```
Prompt 模板：
"请提炼以下网页内容的核心观点，生成一篇概念笔记：
1. 核心定义（1-2句话）
2. 关键要点（3-5条）
3. 与已有知识的关联
4. 保留原始链接作为来源
输出 YAML frontmatter（type: concept, share: true）"
```

**实验记录 → `wiki/experiments/`**
```
Prompt 模板：
"请将以下实验日志整理为结构化实验记录：
1. 实验目标
2. 关键配置（超参数、数据集、模型）
3. 结果对比表格
4. 结论与下一步
不要改变任何实验数字！"
```

### Query（查询知识库）

- 回答问题时优先查询 `wiki/` 目录下的编译后内容
- 明确说明信息来源的文件名（如：「根据 wiki/papers/2024-Attention-Transformers.md」）
- 如果知识库中没有相关信息，明确告知，不要编造
- 可以主动发现相关文件之间的关联并提示用户

### Lint（质量检查）

每周运行一次，检查以下问题：

1. **格式完整性**：每篇 `papers/` 笔记是否包含所有 frontmatter 字段
2. **来源完整性**：概念笔记是否有来源链接
3. **孤立节点**：是否有没有被任何其他页面引用的页面
4. **标签一致性**：domain/type 等标签是否规范
5. **index.md 更新**：目录是否反映了最新文件

---

## Frontmatter 字段规范

```yaml
---
title: "文章标题"
aliases: []                         # 别名（可选）
date: YYYY-MM-DD                    # 创建/更新日期
week: YYYY-WNN                      # 所属周（如 2026-W15）
type: paper                         # paper | concept | experiment | meeting
domain: nlp                         # nlp | cv | ml | rl | gnn | multimodal | other
venue: NeurIPS 2024                 # 期刊/会议（type=paper 时填写）
year: 2024                          # 论文年份
authors: [Author1, Author2]         # 作者列表
arxiv: "2301.12345"                 # arXiv ID（可选）
status: processed                   # reading | processed | archived
share: false                        # true = 下次同步时发布到团队飞书知识库
tags: [transformer, attention]      # 自由标签
---
```

### 关键字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `title` | ✅ | 页面标题 |
| `date` | ✅ | 创建日期 |
| `week` | ✅ | 用于按周筛选同步内容 |
| `type` | ✅ | 内容类型 |
| `share` | ✅ | **`true` 才会同步至团队飞书知识库** |
| `status` | ✅ | 当前处理状态 |
| `domain` | 建议 | 便于分类和检索 |

---

## 文件命名规范

| 类型 | 命名格式 | 示例 |
|------|----------|------|
| 论文笔记 | `YYYY-作者姓-关键词.md` | `2024-Vaswani-Attention.md` |
| 概念笔记 | `概念名称.md`（英文小写下划线）| `contrastive_learning.md` |
| 实验记录 | `YYYY-MM-DD-实验描述.md` | `2026-04-14-bert-finetune.md` |
| 组会记录 | `YYYY-MM-DD-meeting.md` | `2026-04-14-meeting.md` |

---

## 双向链接规范

在 Obsidian 中使用 `[[]]` 创建双向链接，建立知识关联：

```markdown
# 正确示例
参考 [[Vaswani-Attention]] 中提出的 Scaled Dot-Product Attention 机制...
这与 [[contrastive_learning]] 中的对比损失有异曲同工之妙...

# 创建新概念时自动链接
[[BERT]] 是 [[Transformer]] 架构的双向预训练语言模型...
```

---

## 同步策略

- 每周自动同步：`share: true` 的文件
- 不同步：`raw/` 目录下的原始资料
- 不同步：`share: false` 的私人笔记（如未发表实验）
- 同步时 LLM 自动生成摘要附加在文章开头

---

## 版本记录

| 日期 | 变更说明 |
|------|----------|
| 2026-04-14 | 初始版本 |
