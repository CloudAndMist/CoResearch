# CoResearch - 研究组知识库管理方案

> 一套面向学术研究团队的轻量级知识库管理解决方案

---

## 项目背景

### 灵感来源

本项目受 [Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 启发，Karpathy 提出了一种基于 LLM 的知识库管理范式，核心包含三个操作：

- **Ingest（摄取）**：将新材料输入给 LLM，更新知识库
- **Query（查询）**：向 LLM 提问，基于知识库内容给出回答  
- **Lint（质检）**：让 LLM 检查知识的完整性、准确性、一致性

### 研究组需求

目前研究组面临的问题：

> "每周组会前，大家各自阅读的论文和笔记分散在各处——有的在本地文件夹，有的在云笔记，有的在聊天记录里。组会时难以高效分享和讨论，也无法有效构建全组的知识库。"

### 核心设想

基于 Karpathy 的思想，我们设计了**分布式个人知识库 + 中央共享知识库 + 自动化组会议题生成**的解决方案：

```
┌─────────────────────────────────────────────────────────────┐
│                    分布式个人知识库                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ 成员A笔记  │  │ 成员B笔记 │  │ 成员C笔记  │  ...              │
│  │ (本地IMA) │  │ (本地IMA) │  │ (本地IMA) │                  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                   │
│       └──────────────┼──────────────┘                       │
│                      ▼                                      │
│           ┌──────────────────────┐                          │
│           │   中央共享知识库      │                            │
│           │   (IMA共享库)         │                          │
│           └──────────┬───────────┘                          │
│                      ▼                                      │
│           ┌──────────────────────┐                          │
│           │  周六WorkBuddy自动化   │                          │
│           │  - Lint质检           │                          │
│           │  - 议题生成            │                          │
│           │  - 周会总结            │                          │
│           └──────────┬───────────┘                          │
│                      ▼                                      │
│           ┌──────────────────────┐                          │
│           │   周日组会             │                          │
│           │   全员一起查看议题      │                          │
│           └──────────────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

### 方案演进

我们探索了多种技术组合，最终沉淀出两个不同侧重点的解决方案：

- **Plan A**: Obsidian + Git + 飞书 —— 强调技术自主和数据可控，适合有技术背景的团队
- **Plan B**: WorkBuddy + IMA —— 强调开箱即用和降低门槛，适合快速落地

---

## 项目简介

CoResearch 提供两种知识库管理方案，帮助研究团队高效地收集、整理和共享学术资料：

- **Plan A**: Obsidian + Git + 飞书 —— 技术型方案，完全可控
- **Plan B**: WorkBuddy + IMA —— 轻量方案，开箱即用

---

## 快速开始

### 方案选型

不确定选哪个方案？查看 [compare-A-B.md](./compare-A-B.md) 详细对比。

**快速决策**：
- **追求简单快速（推荐）** → [Plan B](./plan-B-workbuddy-ima/1-README.md)
- 可拓展性较强                → [Plan A](./plan-A-obsidian-feishu/README.md)

---

## 方案概览

### Plan A: Obsidian + Git + 飞书

适合对数据可控性要求高、有技术背景的团队。

**核心特点**：
- 本地 Markdown 文件，完全可控
- Git 版本管理，完整历史记录
- 飞书群定时推送组会议题
- 自动化 Lint 代码检查

**目录结构**：
```
plan-A-obsidian-feishu/
├── README.md                    # 方案总览
├── research-group-wiki-guide.md # 完整部署指南
├── CLAUDE.md                    # AI 工作流定义
└── architecture.md              # 架构图
```

### Plan B: WorkBuddy + IMA

适合追求开箱即用、快速落地的团队。

**核心特点**：
- IMA 个人知识库 + 共享知识库
- 内置 AI 辅助阅读和笔记
- WorkBuddy 自动化定时任务
- 无需 Git 等技术配置

**目录结构**：
```
plan-B-workbuddy-ima/
├── 1-README.md              # 方案总览（先读这个）
├── 2-setup-guide.md         # 管理员部署指南
├── 3-member-quickstart.md   # 成员快速上手指南
├── 4-IMA-feishu-analysis.md # 方案可行性分析
├── 5-automation-config.md   # 自动化配置指南
└── 6-ima-skills-api-guide.md # IMA API 使用指南
```

---

## 工作流程对比

### Plan A 工作流程

```
成员本地 Obsidian
    ↓ 编辑笔记
本地 Git 仓库
    ↓ git commit & push
GitHub 远程仓库
    ↓ GitHub Actions 触发
自动生成 Lint 报告 + 组会议题
    ↓ 飞书 Webhook
飞书群组通知
```

### Plan B 工作流程

```
成员本地 IMA
    ↓ 阅读论文、做笔记
IMA 共享知识库
    ↓ WorkBuddy 定时触发（周六 17:00）
自动生成 Lint 报告 + 组会议题
    ↓ 保存到 IMA
组会时在 IMA 中一起查看
```

---

## 核心概念

### 1. Ingest（摄取）

将外部资料输入知识库：
- **Plan A**: 拖拽 PDF/网页到 Obsidian，使用模板创建笔记
- **Plan B**: 拖拽 PDF/网页到 IMA，AI 自动解析内容

### 2. Query（查询）

基于知识库内容进行问答：
- **Plan A**: Obsidian 搜索 + LLM 辅助
- **Plan B**: IMA 内置 AI 问答

### 3. Lint（质检）

定期检查知识库质量：
- **Plan A**: `lint_wiki.py` 脚本
- **Plan B**: WorkBuddy 自动化 Lint 任务

---

## 文件说明

| 文件/文件夹 | 说明 |
|------------|------|
| `README.md` | 本文件，项目总览 |
| `compare-A-B.md` | Plan A 与 Plan B 详细对比 |
| `plan-A-obsidian-feishu/` | Plan A 完整文档 |
| `plan-B-workbuddy-ima/` | Plan B 完整文档 |

---

## 贡献与反馈

欢迎通过以下方式参与：

1. **提交 Issue**: 发现问题或有新需求
2. **分享经验**: 你的使用案例可以帮助他人
3. **改进文档**: 发现文档不清或错误之处

---

## 许可证

MIT License

---

## 致谢

- 灵感来源：[Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- 工具支持：[Obsidian](https://obsidian.md/), [IMA](https://ima.qq.com/), [WorkBuddy](https://www.codebuddy.cn/)

---

*最后更新：2026-04-14*
