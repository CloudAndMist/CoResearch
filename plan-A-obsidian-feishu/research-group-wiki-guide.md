# 计算机研究组本地/云端协同知识库范式

> **版本**：v1.0 · **更新日期**：2026-04-14
>
> **参考来源**：[Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) · 飞书开放平台 · 腾讯 IMA
>
> **适用场景**：计算机方向硕士/博士研究组，3~15 人规模，有一定 Git 使用基础

---

## 目录

1. [为什么需要这套范式](#1-为什么需要这套范式)
2. [核心设计理念](#2-核心设计理念)
3. [系统整体架构](#3-系统整体架构)
4. [本地知识库搭建](#4-本地知识库搭建)
5. [云端知识库配置（飞书）](#5-云端知识库配置飞书)
6. [自动化同步脚本](#6-自动化同步脚本)
7. [组会议题自动生成](#7-组会议题自动生成)
8. [GitHub Actions 定时工作流](#8-github-actions-定时工作流)
9. [Obsidian 笔记模板](#9-obsidian-笔记模板)
10. [快速启动清单](#10-快速启动清单)
11. [常见问题](#11-常见问题)
12. [进阶扩展方案](#12-进阶扩展方案)

---

## 1. 为什么需要这套范式

### 研究组的知识管理痛点

| 痛点 | 现状 | 本方案解决方式 |
|------|------|----------------|
| 论文看了就忘 | 笔记零散，无结构 | LLM 编译为结构化 Wiki 页面，永久可查 |
| 知识无法共享 | 各自为战，重复读论文 | 每周自动同步至团队飞书知识库 |
| 组会无实质内容 | 临时凑材料，讨论浅 | LLM 自动生成启发式议题 |
| 知识无法积累 | 毕业即清零 | Git 版本控制 + 云端归档，永久沉淀 |
| 信息格式混乱 | PDF/Word/微信/链接各异 | 统一转为 Markdown，LLM 可无差别处理 |

### 参考来源：Karpathy LLM Wiki

[Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 提出将 LLM 作为"程序员"、将个人 Wiki 作为"代码库"，通过三个核心操作持续维护知识：

- **Ingest（摄取）**：将新材料输入给 LLM，更新 Wiki 页面
- **Query（查询）**：向 LLM 提问，基于 Wiki 内容给出回答
- **Lint（质检）**：让 LLM 检查知识的完整性、准确性、一致性

本方案将这一理念扩展到**团队场景**，增加云端同步和自动化组会生成。

---

## 2. 核心设计理念

### 2.1 三层知识架构

```
原始来源层 (raw/)        →    Wiki 编译层 (wiki/)     →    Schema 定义层
PDF · 链接 · 笔记             结构化 Markdown 页面          CLAUDE.md · index.md
（人类可读的原材料）          （LLM 编译的知识单元）         （工作流规则与目录）
```

### 2.2 双轨运行模式

- **个人轨道**：每位成员独立维护本地 Obsidian Wiki，自由探索
- **团队轨道**：每周自动汇聚到飞书云端知识库，形成团队智慧

### 2.3 支持的内容格式

| 格式 | 处理方式 | 存储位置 |
|------|----------|----------|
| PDF 论文（arXiv、ACL 等）| Obsidian 内置 PDF 预览 + LLM 提取 | `raw/papers/` → `wiki/papers/` |
| 网页/推文/公众号文章 | Obsidian Web Clipper 一键剪藏 | `raw/clips/` → `wiki/concepts/` |
| 个人 Markdown 笔记 | 直接写入，遵循格式规范 | `wiki/` 对应目录 |
| 代码/实验记录 | Jupyter 导出 + LLM 总结 | `raw/experiments/` → `wiki/experiments/` |
| 会议/讲座 | 录音字幕 + LLM 整理 | `raw/meetings/` → `wiki/meetings/` |
| 截图/图片 | 存入 raw/，Obsidian 内嵌显示 | `raw/images/` |

---

## 3. 系统整体架构

```
┌──────────────────────────────────────────────────────────────────┐
│                      本地层（每位成员独立）                        │
│   Obsidian IDE  +  LLM Wiki (.md)  +  Git 版本控制               │
└───────────────────────────┬──────────────────────────────────────┘
                            │ git push（每周触发 / 手动触发）
                            ▼
┌──────────────────────────────────────────────────────────────────┐
│                    自动化同步层                                    │
│   GitHub Actions  +  Python 脚本  +  LLM API（DeepSeek/混元）   │
└──────────────┬─────────────────────────────┬─────────────────────┘
               │ 飞书 API 写入                │ 组会议程推送
               ▼                             ▼
┌─────────────────────────┐       ┌──────────────────────────────┐
│     飞书知识库（云端）    │       │     飞书机器人 → 研究组群组   │
│  团队共享 · 权限管理     │       │  每周组会议题 · 知识更新通报  │
└─────────────────────────┘       └──────────────────────────────┘
```

### 工具选型说明

| 层次 | 工具 | 选择理由 |
|------|------|----------|
| 本地 IDE | **Obsidian** | 免费、纯 Markdown、插件丰富、离线可用 |
| 版本控制 | **Git + GitHub** | 免费、协作成熟、可触发 Actions |
| 云端知识库 | **飞书知识库** | API 最完整、国内稳定、团队协作强 |
| 备选云端 | **腾讯 IMA** | 已开源，支持私有部署，RAG 能力强 |
| LLM 服务 | **DeepSeek V3** | 性价比极高（约 ¥1/百万 tokens），长文本优秀 |
| 自动化调度 | **GitHub Actions** | 免费、可靠、代码化配置 |

---

## 4. 本地知识库搭建

### 4.1 仓库目录结构

```
research-wiki/
├── .github/
│   └── workflows/
│       └── weekly-sync.yml        # 每周自动同步
├── local/
│   ├── raw/                       # 原始资料（不上传至飞书）
│   │   ├── papers/                # PDF 论文
│   │   ├── clips/                 # 网页剪藏
│   │   ├── experiments/           # 实验原始数据
│   │   └── meetings/              # 会议录音/原始纪要
│   └── wiki/                      # LLM 编译后的知识页（同步至飞书）
│       ├── papers/                # 论文结构化笔记
│       ├── concepts/              # 概念解释与定义
│       ├── experiments/           # 实验记录摘要
│       └── meetings/              # 组会记录整理
├── index.md                       # 知识库总目录（自动生成）
├── log.md                         # 操作日志（自动追加）
├── CLAUDE.md                      # Wiki 工作流定义（核心规则文件）
├── scripts/
│   ├── ingest.py                  # LLM 摄取脚本
│   ├── sync_to_feishu.py          # 飞书同步脚本
│   ├── generate_meeting_topics.py # 组会议题生成
│   └── lint_wiki.py               # 知识库质检
├── config/
│   └── feishu_config.yaml         # 飞书 API 配置（勿提交密钥！）
└── .gitignore                     # 忽略 PDF、密钥等大文件
```

### 4.2 安装 Obsidian 必备插件

在 Obsidian 中进入「设置 → 第三方插件」，安装以下插件：

| 插件名称 | 用途 |
|----------|------|
| **Obsidian Web Clipper** | 一键抓取网页/推文转为 Markdown |
| **Templater** | 笔记模板自动填充（日期、作者等元信息）|
| **Dataview** | 用 SQL 语法查询知识库（生成论文列表等）|
| **Git** | 在 Obsidian 内直接 git commit/push |
| **PDF++ ** | 增强 PDF 阅读体验，支持高亮批注导出 |
| **Excalidraw** | 绘制架构图、思维导图，嵌入 md 文件 |

### 4.3 CLAUDE.md —— Wiki 工作流定义文件

这是整套系统的"宪法"，告诉 LLM 应该如何处理这个知识库。

```markdown
# Wiki 工作流定义（CLAUDE.md）

## 知识库概述
本知识库是 [研究方向] 方向的个人研究 Wiki。
维护者：[姓名] | 研究方向：[方向] | 导师：[导师姓名]

## 目录结构规范
- `wiki/papers/`：论文阅读笔记，每篇论文一个 .md 文件
- `wiki/concepts/`：领域概念解释，每个概念一个 .md 文件
- `wiki/experiments/`：实验记录，按日期命名
- `wiki/meetings/`：组会记录，按日期命名

## Ingest 流程（摄取新内容）
1. 将原始资料放入对应的 raw/ 子目录
2. 对于 PDF 论文，在 raw/papers/ 中使用以下 Prompt：
   "请阅读这篇论文，提取：标题、作者、年份、发表会议/期刊、
   研究问题、核心方法、主要贡献（3点）、实验结论、局限性、
   与我已有研究的关联。输出为 Markdown 格式，存入 wiki/papers/"
3. 对于网页内容，使用 Obsidian Web Clipper 剪藏后，
   让 LLM 提炼核心观点，存入 wiki/concepts/

## Query 规范（查询知识库）
- 优先查询 wiki/ 目录下的编译后内容
- 回答时注明来源文件名
- 如果 wiki 中没有相关内容，明确说明

## Lint 规范（质量检查）
每周运行一次，检查：
1. 论文笔记是否包含所有必填字段（见 papers/ 模板）
2. 概念定义是否有来源引用
3. 是否有孤立节点（没有被任何其他页面引用）
4. index.md 是否已更新

## 标签系统
使用 YAML frontmatter 中的 tags 字段：
- `type`: paper | concept | experiment | meeting
- `domain`: nlp | cv | ml | rl | gnn | multimodal
- `status`: reading | processed | archived
- `week`: YYYY-Www（如 2026-W15）
- `share`: true | false（是否同步至团队知识库）
```

### 4.4 日常使用流程

```
每日工作流（约 10~30 分钟）：

1. 读论文/看资料时：
   → 使用 Web Clipper 或手动创建笔记
   → 在 Obsidian 中整理，添加 frontmatter 标签
   → 如果内容值得分享，设置 share: true

2. 完成笔记后：
   → 在终端运行 python scripts/lint_wiki.py 检查格式
   → git add . && git commit -m "add paper: xxx"
   → git push（自动触发 GitHub Actions 记录）

3. 每周日（自动触发或手动触发）：
   → GitHub Actions 运行，同步至飞书知识库
   → LLM 生成本周组会议程，推送至飞书群
```

---

## 5. 云端知识库配置（飞书）

### 5.1 创建飞书应用（一次性配置）

1. 登录 [飞书开放平台](https://open.feishu.cn)
2. 创建企业自建应用，命名为「研究组知识库Bot」
3. 在「权限管理」中开启以下权限：
   - `wiki:wiki` —— 知识库读写
   - `docx:document` —— 文档读写
   - `im:message` —— 发送消息
   - `bitable:app` —— 多维表格读写
4. 获取 `App ID` 和 `App Secret`，填入 `config/feishu_config.yaml`

```yaml
# config/feishu_config.yaml
# ⚠️ 请勿将此文件提交到 GitHub（已加入 .gitignore）
# 在 GitHub Actions 中使用 Secrets 传入

feishu:
  app_id: "cli_xxxxxxxxxx"
  app_secret: "xxxxxxxxxxxxxxxx"
  wiki_space_id: "xxxxxxxxxx"        # 知识库空间 ID
  webhook_url: "https://open.feishu.cn/open-apis/bot/v2/hook/xxx"  # 机器人 Webhook

llm:
  provider: "deepseek"               # deepseek | hunyuan | openai
  api_key: "sk-xxxxxxxxxx"
  model: "deepseek-chat"
  base_url: "https://api.deepseek.com/v1"
```

### 5.2 知识库结构设计

在飞书知识库中创建以下树形结构：

```
📚 研究组知识库
├── 📂 论文阅读笔记
│   ├── 📂 NLP / LLM 方向
│   ├── 📂 CV / 多模态方向
│   └── 📂 其他方向
├── 📂 领域概念解释
│   ├── 📂 基础概念
│   └── 📂 前沿技术
├── 📂 实验记录
│   └── 📂 [成员名字]的实验
├── 📂 组会记录
│   └── 📂 2026年
├── 📂 每周知识更新
│   └── 📂 2026年
└── 📂 共享资源
    ├── 📄 论文追踪表（多维表格）
    └── 📄 研究组规范
```

### 5.3 获取知识库空间 ID

```bash
# 先获取 tenant_access_token
curl -X POST "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal" \
  -H "Content-Type: application/json" \
  -d '{"app_id":"你的AppID","app_secret":"你的AppSecret"}'

# 再获取知识库列表
curl -X GET "https://open.feishu.cn/open-apis/wiki/v2/spaces" \
  -H "Authorization: Bearer <tenant_access_token>"
```

---

## 6. 自动化同步脚本

### 6.1 `scripts/sync_to_feishu.py`

```python
"""
sync_to_feishu.py
功能：扫描本周新增/修改的 wiki/ .md 文件，通过 LLM 生成摘要后同步至飞书知识库
使用：python scripts/sync_to_feishu.py [--days 7] [--dry-run]
"""

import os
import re
import yaml
import json
import argparse
import requests
from datetime import datetime, timedelta
from pathlib import Path
from openai import OpenAI

# ─── 配置加载 ────────────────────────────────────────────────────────────────

def load_config(config_path="config/feishu_config.yaml"):
    with open(config_path, "r", encoding="utf-8") as f:
        return yaml.safe_load(f)

# ─── 飞书 API 客户端 ──────────────────────────────────────────────────────────

class FeishuClient:
    BASE_URL = "https://open.feishu.cn/open-apis"

    def __init__(self, app_id: str, app_secret: str):
        self.app_id = app_id
        self.app_secret = app_secret
        self._token = None
        self._token_expire = 0

    def get_token(self) -> str:
        if self._token and datetime.now().timestamp() < self._token_expire:
            return self._token
        resp = requests.post(
            f"{self.BASE_URL}/auth/v3/tenant_access_token/internal",
            json={"app_id": self.app_id, "app_secret": self.app_secret}
        )
        data = resp.json()
        self._token = data["tenant_access_token"]
        self._token_expire = datetime.now().timestamp() + data["expire"] - 60
        return self._token

    def headers(self) -> dict:
        return {
            "Authorization": f"Bearer {self.get_token()}",
            "Content-Type": "application/json"
        }

    def create_wiki_node(self, space_id: str, parent_node_token: str,
                         title: str, obj_type: str = "doc") -> str:
        """在知识库中创建节点，返回 node_token"""
        resp = requests.post(
            f"{self.BASE_URL}/wiki/v2/spaces/{space_id}/nodes",
            headers=self.headers(),
            json={
                "obj_type": obj_type,
                "parent_node_token": parent_node_token,
                "node_type": "origin",
                "title": title
            }
        )
        result = resp.json()
        if result.get("code") != 0:
            raise Exception(f"创建节点失败: {result}")
        return result["data"]["node"]["obj_token"]

    def write_document_content(self, document_id: str, content_blocks: list):
        """向文档写入内容块"""
        resp = requests.patch(
            f"{self.BASE_URL}/docx/v1/documents/{document_id}/blocks/{document_id}",
            headers=self.headers(),
            json={"children": content_blocks, "index": 0}
        )
        return resp.json()

    def send_webhook_message(self, webhook_url: str, message: str):
        """通过 Webhook 向飞书群发送消息"""
        resp = requests.post(
            webhook_url,
            json={"msg_type": "text", "content": {"text": message}}
        )
        return resp.json()

# ─── LLM 摘要生成 ─────────────────────────────────────────────────────────────

class LLMSummarizer:
    SUMMARY_PROMPT = """请分析以下研究笔记，生成一个结构化摘要，格式如下：

**核心内容（50字内）**：[一句话概括]

**主要贡献/观点**：
- [要点1]
- [要点2]
- [要点3]

**关键词**：[用逗号分隔，最多6个词]

**与已有研究的潜在关联**：[如有，简要说明]

笔记内容：
{content}"""

    def __init__(self, cfg: dict):
        self.client = OpenAI(
            api_key=cfg["api_key"],
            base_url=cfg.get("base_url", "https://api.openai.com/v1")
        )
        self.model = cfg.get("model", "deepseek-chat")

    def summarize(self, content: str) -> str:
        resp = self.client.chat.completions.create(
            model=self.model,
            messages=[{
                "role": "user",
                "content": self.SUMMARY_PROMPT.format(content=content[:4000])
            }],
            max_tokens=600
        )
        return resp.choices[0].message.content

# ─── 文件扫描 ─────────────────────────────────────────────────────────────────

def get_changed_files(wiki_dir: str = "local/wiki", days: int = 7) -> list[Path]:
    """获取最近 N 天内修改的 .md 文件"""
    cutoff = datetime.now() - timedelta(days=days)
    changed = []
    for md_file in Path(wiki_dir).rglob("*.md"):
        mtime = datetime.fromtimestamp(md_file.stat().st_mtime)
        if mtime > cutoff:
            # 检查 frontmatter 中 share: true
            content = md_file.read_text(encoding="utf-8")
            if "share: true" in content:
                changed.append(md_file)
    return changed

def parse_frontmatter(content: str) -> tuple[dict, str]:
    """解析 YAML frontmatter，返回 (meta, body)"""
    if content.startswith("---"):
        end = content.find("---", 3)
        if end != -1:
            fm = yaml.safe_load(content[3:end])
            body = content[end+3:].strip()
            return fm or {}, body
    return {}, content

# ─── 内容转换为飞书文档块 ────────────────────────────────────────────────────

def markdown_to_feishu_blocks(md_content: str) -> list:
    """将 Markdown 转换为飞书文档块格式（简化版）"""
    blocks = []
    for line in md_content.split("\n"):
        line = line.strip()
        if not line:
            continue
        if line.startswith("## "):
            blocks.append({
                "block_type": 3,  # Heading2
                "heading2": {"elements": [{"text_run": {"content": line[3:]}}]}
            })
        elif line.startswith("### "):
            blocks.append({
                "block_type": 4,  # Heading3
                "heading3": {"elements": [{"text_run": {"content": line[4:]}}]}
            })
        elif line.startswith("- ") or line.startswith("* "):
            blocks.append({
                "block_type": 12,  # BulletedList
                "bullet": {"elements": [{"text_run": {"content": line[2:]}}]}
            })
        else:
            blocks.append({
                "block_type": 2,  # Text
                "text": {"elements": [{"text_run": {"content": line}}]}
            })
    return blocks

# ─── 主流程 ──────────────────────────────────────────────────────────────────

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--days", type=int, default=7, help="扫描最近N天的变更文件")
    parser.add_argument("--dry-run", action="store_true", help="不实际写入，仅打印")
    args = parser.parse_args()

    cfg = load_config()
    feishu = FeishuClient(cfg["feishu"]["app_id"], cfg["feishu"]["app_secret"])
    summarizer = LLMSummarizer(cfg["llm"])

    changed_files = get_changed_files(days=args.days)
    print(f"📂 发现 {len(changed_files)} 个需要同步的文件")

    synced = []
    for md_file in changed_files:
        content = md_file.read_text(encoding="utf-8")
        meta, body = parse_frontmatter(content)
        title = meta.get("title") or md_file.stem
        week_tag = meta.get("week", datetime.now().strftime("%Y-W%V"))

        print(f"  → 处理: {md_file.name}")

        # 生成 LLM 摘要
        summary = summarizer.summarize(body)
        full_content = f"# {title}\n\n> **LLM 自动摘要**\n\n{summary}\n\n---\n\n{body}"

        if args.dry_run:
            print(f"    [DRY RUN] 将同步至飞书: {title}")
            continue

        # 确定父节点（根据文件路径映射到飞书知识库目录）
        parent_token = cfg["feishu"].get("default_parent_token", "")
        path_parts = md_file.parts
        if "papers" in path_parts:
            parent_token = cfg["feishu"].get("papers_parent_token", parent_token)
        elif "concepts" in path_parts:
            parent_token = cfg["feishu"].get("concepts_parent_token", parent_token)
        elif "experiments" in path_parts:
            parent_token = cfg["feishu"].get("experiments_parent_token", parent_token)

        # 创建知识库节点并写入内容
        doc_token = feishu.create_wiki_node(
            space_id=cfg["feishu"]["wiki_space_id"],
            parent_node_token=parent_token,
            title=f"[{week_tag}] {title}"
        )
        blocks = markdown_to_feishu_blocks(full_content)
        feishu.write_document_content(doc_token, blocks)
        synced.append(title)
        print(f"    ✅ 已同步: {title}")

    # 推送同步完成通知
    if synced and not args.dry_run:
        msg = f"📚 本周知识库更新完成\n共同步 {len(synced)} 篇内容：\n"
        msg += "\n".join(f"  · {t}" for t in synced[:10])
        feishu.send_webhook_message(cfg["feishu"]["webhook_url"], msg)

    print(f"\n✅ 同步完成，共处理 {len(synced)} 个文件")

if __name__ == "__main__":
    main()
```

---

## 7. 组会议题自动生成

### 7.1 `scripts/generate_meeting_topics.py`

```python
"""
generate_meeting_topics.py
功能：分析本周知识库新增内容，LLM 生成结构化组会议程，推送至飞书群
使用：python scripts/generate_meeting_topics.py
"""

import os
import yaml
from datetime import datetime, timedelta
from pathlib import Path
from openai import OpenAI
import requests

# ─── Prompt 模板（核心设计）─────────────────────────────────────────────────

MEETING_AGENDA_PROMPT = """你是一位计算机研究组的学术助手。请分析以下本周研究组成员的学习内容，
生成一份高质量的组会议程，帮助成员进行深度讨论。

## 本周新增内容摘要
{weekly_content}

## 上周遗留待讨论事项
{last_week_items}

## 请生成包含以下部分的组会议程：

### 一、本周知识速览（5分钟）
简要列出本周每位成员的核心进展（1-2句话/人）

### 二、深度讨论议题（30分钟）
选择 2-3 个最具讨论价值的议题，每个议题包含：
- 背景说明（2-3句话）
- 核心争议点或待解问题
- 启发式讨论问题（3个开放性问题，能够引发思考）

选题标准：
- 优先选择多位成员工作存在交叉的议题
- 优先选择当前领域的热点或争议问题
- 优先选择有实践意义的方法论问题

### 三、跨方向关联发现（10分钟）
分析本周内容中，不同研究方向之间是否存在可借鉴的思路或方法

### 四、下周行动建议（5分钟）
- 建议每位成员重点关注的方向（基于本周讨论）
- 下周值得共同阅读的论文推荐（如有）

### 五、注意事项
⚠️ 本议程由 AI 自动生成，组会前请导师和同学根据实际情况调整"""

# ─── 内容收集 ─────────────────────────────────────────────────────────────────

def collect_weekly_content(wiki_dir: str = "local/wiki", days: int = 7) -> str:
    """收集本周所有新增 wiki 内容"""
    cutoff = datetime.now() - timedelta(days=days)
    contents = []

    for md_file in Path(wiki_dir).rglob("*.md"):
        mtime = datetime.fromtimestamp(md_file.stat().st_mtime)
        if mtime > cutoff:
            text = md_file.read_text(encoding="utf-8")
            # 只取前 500 字避免 token 过多
            contents.append(f"### {md_file.stem}\n{text[:500]}\n")

    return "\n---\n".join(contents) if contents else "本周暂无新增内容"

def get_last_week_items(meetings_dir: str = "local/wiki/meetings") -> str:
    """从上周组会记录中提取未完成的 Action Items"""
    meeting_files = sorted(Path(meetings_dir).glob("*.md"), reverse=True)
    if not meeting_files:
        return "无"
    last_meeting = meeting_files[0].read_text(encoding="utf-8")
    # 简单提取 Action Items 部分
    if "Action Item" in last_meeting or "行动项" in last_meeting:
        start = last_meeting.find("Action Item")
        if start == -1:
            start = last_meeting.find("行动项")
        return last_meeting[start:start+500]
    return "无遗留事项"

# ─── 生成与推送 ──────────────────────────────────────────────────────────────

def generate_agenda(cfg: dict) -> str:
    """调用 LLM 生成组会议程"""
    client = OpenAI(
        api_key=cfg["llm"]["api_key"],
        base_url=cfg["llm"].get("base_url", "https://api.openai.com/v1")
    )

    weekly_content = collect_weekly_content()
    last_week_items = get_last_week_items()

    resp = client.chat.completions.create(
        model=cfg["llm"]["model"],
        messages=[{
            "role": "system",
            "content": "你是一位严谨、富有洞察力的计算机学术助手，擅长发现研究中的关键问题和跨领域关联。"
        }, {
            "role": "user",
            "content": MEETING_AGENDA_PROMPT.format(
                weekly_content=weekly_content[:6000],
                last_week_items=last_week_items[:1000]
            )
        }],
        max_tokens=2000,
        temperature=0.7
    )

    return resp.choices[0].message.content

def save_agenda_to_wiki(agenda: str):
    """保存议程到本地 wiki/meetings/ 目录"""
    today = datetime.now().strftime("%Y-%m-%d")
    week = datetime.now().strftime("W%V")
    output_path = Path(f"local/wiki/meetings/meeting-{today}.md")
    output_path.parent.mkdir(parents=True, exist_ok=True)

    content = f"""---
title: "{datetime.now().strftime('%Y年第%V周')} 组会议程"
date: {today}
week: {datetime.now().strftime('%Y-W%V')}
type: meeting
share: true
---

{agenda}

---
*本议程由 AI 自动生成，生成时间：{datetime.now().strftime('%Y-%m-%d %H:%M')}*
"""
    output_path.write_text(content, encoding="utf-8")
    print(f"✅ 议程已保存至: {output_path}")
    return str(output_path)

def push_to_feishu(agenda: str, webhook_url: str):
    """推送精简版议程至飞书群"""
    # 提取前 500 字作为群消息预览
    preview = agenda[:500] + "\n\n[查看完整议程 → 飞书知识库]"
    week_str = datetime.now().strftime("%Y年第%V周")

    message = f"📋 {week_str}组会议程已生成\n{'='*30}\n{preview}"
    resp = requests.post(
        webhook_url,
        json={"msg_type": "text", "content": {"text": message}}
    )
    if resp.json().get("code") == 0 or resp.json().get("StatusCode") == 0:
        print("✅ 议程已推送至飞书群")
    else:
        print(f"⚠️ 推送失败: {resp.json()}")

def main():
    with open("config/feishu_config.yaml", "r", encoding="utf-8") as f:
        cfg = yaml.safe_load(f)

    print("🤖 正在生成本周组会议程...")
    agenda = generate_agenda(cfg)
    save_agenda_to_wiki(agenda)
    push_to_feishu(agenda, cfg["feishu"]["webhook_url"])
    print("\n🎉 组会议程生成完成！")

if __name__ == "__main__":
    main()
```

---

## 8. GitHub Actions 定时工作流

### 8.1 `.github/workflows/weekly-sync.yml`

```yaml
# .github/workflows/weekly-sync.yml
# 每周日晚 22:00 自动触发知识库同步和组会议程生成
# 也支持手动触发（workflow_dispatch）

name: 每周知识库同步

on:
  schedule:
    - cron: '0 14 * * 0'   # UTC 14:00 = 北京时间 22:00，每周日
  workflow_dispatch:         # 允许手动触发
    inputs:
      days:
        description: '扫描最近N天的变更（默认7天）'
        required: false
        default: '7'
      dry_run:
        description: '是否为演习模式（true/false）'
        required: false
        default: 'false'

jobs:
  sync-and-generate:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 获取完整历史，用于增量检测

      - name: 配置 Python 环境
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: 安装依赖
        run: |
          pip install openai requests pyyaml python-frontmatter

      - name: 创建配置文件
        run: |
          mkdir -p config
          cat > config/feishu_config.yaml << EOF
          feishu:
            app_id: "${{ secrets.FEISHU_APP_ID }}"
            app_secret: "${{ secrets.FEISHU_APP_SECRET }}"
            wiki_space_id: "${{ secrets.FEISHU_WIKI_SPACE_ID }}"
            webhook_url: "${{ secrets.FEISHU_WEBHOOK_URL }}"
            default_parent_token: "${{ secrets.FEISHU_DEFAULT_PARENT_TOKEN }}"
            papers_parent_token: "${{ secrets.FEISHU_PAPERS_PARENT_TOKEN }}"
            concepts_parent_token: "${{ secrets.FEISHU_CONCEPTS_PARENT_TOKEN }}"
          llm:
            provider: "deepseek"
            api_key: "${{ secrets.LLM_API_KEY }}"
            model: "deepseek-chat"
            base_url: "https://api.deepseek.com/v1"
          EOF

      - name: 同步内容至飞书知识库
        run: |
          DAYS=${{ github.event.inputs.days || '7' }}
          DRY_RUN=${{ github.event.inputs.dry_run || 'false' }}
          if [ "$DRY_RUN" = "true" ]; then
            python scripts/sync_to_feishu.py --days $DAYS --dry-run
          else
            python scripts/sync_to_feishu.py --days $DAYS
          fi

      - name: 生成本周组会议程
        run: python scripts/generate_meeting_topics.py

      - name: 提交自动生成的议程文件
        run: |
          git config user.name "Wiki Bot"
          git config user.email "wiki-bot@research-group.com"
          git add local/wiki/meetings/
          git diff --staged --quiet || git commit -m "chore: auto-generate meeting agenda [skip ci]"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: 上传同步日志
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: sync-log-${{ github.run_id }}
          path: |
            local/log.md
          retention-days: 30
```

### 8.2 配置 GitHub Secrets

在仓库的「Settings → Secrets and variables → Actions」中添加以下 Secrets：

| Secret 名称 | 说明 |
|-------------|------|
| `FEISHU_APP_ID` | 飞书应用 App ID |
| `FEISHU_APP_SECRET` | 飞书应用 App Secret |
| `FEISHU_WIKI_SPACE_ID` | 知识库空间 ID |
| `FEISHU_WEBHOOK_URL` | 飞书机器人 Webhook 地址 |
| `FEISHU_DEFAULT_PARENT_TOKEN` | 默认父节点 token |
| `FEISHU_PAPERS_PARENT_TOKEN` | 论文目录父节点 token |
| `FEISHU_CONCEPTS_PARENT_TOKEN` | 概念目录父节点 token |
| `LLM_API_KEY` | LLM API Key（DeepSeek/混元等）|

---

## 9. Obsidian 笔记模板

### 9.1 论文阅读笔记模板（`templates/paper-note.md`）

```markdown
---
title: "<% tp.file.title %>"
aliases: []
date: <% tp.date.now("YYYY-MM-DD") %>
week: <% tp.date.now("YYYY-[W]WW") %>
type: paper
domain: ""          # nlp | cv | ml | rl | gnn | multimodal
venue: ""           # NeurIPS | ICML | ACL | CVPR | ICLR ...
year: 
authors: []
arxiv: ""           # 如: 2301.12345
status: reading     # reading | processed | archived
share: false        # 是否同步至团队知识库
tags: []
---

# <% tp.file.title %>

## 📌 一句话总结
> 

## 🔍 研究问题
本文要解决的核心问题是什么？

## 💡 核心方法
方法的关键创新点：
- 
- 

## 📊 主要实验结果
| 数据集 | 指标 | 本文结果 | SOTA对比 |
|--------|------|----------|----------|
|        |      |          |          |

## ✅ 主要贡献
1. 
2. 
3. 

## ⚠️ 局限性与不足
- 
- 

## 🤔 个人思考
与我研究的关联：

启发性想法：

疑问点（待深入研究）：
- 

## 🔗 相关工作
- [[]] - 
- [[]] - 

## 📚 引用
```bibtex

```

---
*阅读时间：<% tp.date.now("YYYY-MM-DD") %> | 完成摘要：[ ]*
```

### 9.2 概念笔记模板（`templates/concept-note.md`）

```markdown
---
title: "<% tp.file.title %>"
date: <% tp.date.now("YYYY-MM-DD") %>
week: <% tp.date.now("YYYY-[W]WW") %>
type: concept
domain: ""
status: processed
share: true
tags: []
---

# <% tp.file.title %>

## 定义
> 用1-2句话给出精确定义

## 直觉理解
用类比或直觉解释：

## 技术细节
核心公式或算法（如有）：

## 典型应用场景
- 
- 

## 与相关概念的区别
| 概念 | 本概念 | 区别 |
|------|--------|------|
|      |        |      |

## 来源
- 论文：[[]]
- 教程：
- 来源链接：

## 个人注记
```

### 9.3 实验记录模板（`templates/experiment-note.md`）

```markdown
---
title: "实验记录 - <% tp.date.now('YYYY-MM-DD') %>"
date: <% tp.date.now("YYYY-MM-DD") %>
week: <% tp.date.now("YYYY-[W]WW") %>
type: experiment
status: running     # running | completed | failed | paused
share: false
tags: []
---

# 实验记录：<% tp.file.title %>

## 实验目标
验证什么假设，解决什么问题？

## 实验设置
- **模型**：
- **数据集**：
- **超参数**：
  - learning_rate: 
  - batch_size: 
  - epochs: 
- **硬件**：
- **代码版本**：[commit hash]

## 实验结果
| 实验组 | 配置变化 | 指标1 | 指标2 | 备注 |
|--------|----------|-------|-------|------|
| Baseline |        |       |       |      |
| Exp-1  |          |       |       |      |

## 结论
- ✅ 验证了：
- ❌ 未能验证：
- ❓ 新的疑问：

## 下一步
- [ ] 
- [ ] 

## 错误记录
（记录踩过的坑，防止重复）
```

---

## 10. 快速启动清单

### 第一次使用（管理员/导师）

- [ ] **Step 1**：在 GitHub 创建组织仓库 `research-wiki`，设置为私有
- [ ] **Step 2**：在飞书开放平台创建应用，获取 App ID 和 Secret
- [ ] **Step 3**：在飞书知识库创建团队空间，设置目录结构
- [ ] **Step 4**：在 GitHub 仓库 Secrets 中配置所有密钥（见第 8.2 节）
- [ ] **Step 5**：将本仓库模板 fork 给所有成员
- [ ] **Step 6**：运行一次手动触发测试：`workflow_dispatch`，验证同步正常

### 每位新成员（第一次加入）

- [ ] **Step 1**：安装 [Obsidian](https://obsidian.md/)（免费）
- [ ] **Step 2**：克隆个人仓库：`git clone https://github.com/[组织]/research-wiki-[姓名]`
- [ ] **Step 3**：用 Obsidian 打开仓库文件夹（Open folder as vault）
- [ ] **Step 4**：在 Obsidian 中安装必备插件（见第 4.2 节）
- [ ] **Step 5**：修改 `CLAUDE.md` 中的个人信息（姓名、研究方向）
- [ ] **Step 6**：创建第一篇论文笔记，使用模板（Templater 插件）
- [ ] **Step 7**：设置 `share: true`，手动触发一次同步测试

### 每周例行操作

**成员（每周日前）：**
- [ ] 整理本周笔记，将值得分享的内容设置 `share: true`
- [ ] 运行 `python scripts/lint_wiki.py` 检查格式
- [ ] `git commit && git push`

**自动化（每周日 22:00）：**
- [ ] GitHub Actions 自动触发
- [ ] 内容同步至飞书知识库
- [ ] 组会议程生成并推送至群组

**组会（每周固定时间）：**
- [ ] 所有成员提前阅读 AI 生成的议程
- [ ] 导师根据实际情况调整议题
- [ ] 组会后记录纪要存入 `wiki/meetings/`

---

## 11. 常见问题

**Q: 我不会用命令行，可以用这套方案吗？**

A: 可以，日常使用 Obsidian 完全是图形界面操作。只有第一次环境配置需要简单的命令行操作。自动化部分由 GitHub Actions 在后台完成，成员无感知。

---

**Q: 飞书 API 有什么限制？**

A: 飞书免费版知识库 API 每分钟请求上限约 100 次，对于研究组规模（每周几十篇笔记）完全足够。注意：知识库节点创建和文档写入是两个 API 调用，需要分别处理。

---

**Q: 如果我不想使用飞书，有替代方案吗？**

A: 有，可以选择：
- **腾讯 IMA**：已有官方 API Key 和 Skills 接入包，适合个人或小团队
- **Notion**：有完善的 API，国际化支持好
- **GitHub Wiki**：最简单，直接用 Git 管理，零额外成本

---

**Q: LLM API 费用大概多少？**

A: 使用 DeepSeek V3，每周处理约 50 篇笔记（每篇约 1000 字），费用约 **¥0.5~2 元/周**，全年不超过 ¥100。

---

**Q: 知识库内容的隐私问题？**

A: 建议将 GitHub 仓库设置为**私有（Private）**，飞书知识库设置为**仅团队成员可见**。如有高敏感内容（未发表工作、实验数据），在 frontmatter 中设置 `share: false`，不会同步至云端。

---

## 12. 进阶扩展方案

### 方案 A：腾讯 IMA 作为云端备选

适合已经在使用 IMA 的用户，或需要更强 RAG 问答能力的场景：

```python
# 使用 IMA API 写入知识库（替换 sync_to_feishu.py 中的云端部分）
import requests

IMA_API_BASE = "https://ima.qq.com/api/v1"

def sync_to_ima(content: str, title: str, api_key: str):
    """将内容写入腾讯 IMA 知识库"""
    headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}
    resp = requests.post(
        f"{IMA_API_BASE}/knowledge/add",
        headers=headers,
        json={"title": title, "content": content, "source": "research-wiki"}
    )
    return resp.json()
```

### 方案 B：飞书低代码自动化（无需编程）

如果团队不熟悉编程，可使用飞书内置的自动化功能：

1. 进入飞书多维表格 → 自动化
2. 选择模板「定时推送 AI 总结」
3. 配置：触发时间（每周日）、数据来源（知识库）、推送群组
4. 无需写一行代码，适合快速启动

### 方案 C：私有化部署 IMA

对于有服务器资源、注重数据隐私的研究组：

- IMA 已开源（[GitHub 地址](https://github.com/搜索ima腾讯开源)），支持私有部署
- 部署后团队成员通过局域网访问，数据不出组内
- 适合有未发表核心技术的研究组

### 方案 D：融合知识图谱

在基础范式之上，增加知识图谱能力：

```python
# 使用 Obsidian 的双向链接生成知识图谱
# 并通过 NetworkX 分析论文间的引用关系
import networkx as nx

def build_knowledge_graph(wiki_dir: str):
    """从 wiki/ 中的 [[双向链接]] 构建知识图谱"""
    G = nx.DiGraph()
    for md_file in Path(wiki_dir).rglob("*.md"):
        content = md_file.read_text(encoding="utf-8")
        links = re.findall(r'\[\[([^\]]+)\]\]', content)
        for link in links:
            G.add_edge(md_file.stem, link)
    return G
```

---

## 附录：核心文件一览

| 文件 | 说明 | 是否需要修改 |
|------|------|-------------|
| `CLAUDE.md` | Wiki 工作流定义（最重要！）| 每人必须修改 |
| `config/feishu_config.yaml` | 飞书 API 配置 | 管理员配置一次 |
| `scripts/sync_to_feishu.py` | 飞书同步脚本 | 可按需修改 |
| `scripts/generate_meeting_topics.py` | 组会议程生成 | 可修改 Prompt |
| `.github/workflows/weekly-sync.yml` | 定时任务配置 | 可修改触发时间 |
| `templates/paper-note.md` | 论文笔记模板 | 可按方向定制 |

---

> 📬 **反馈与贡献**
>
> 本范式处于持续迭代中。如有改进建议，欢迎在研究组内讨论，或通过 Git PR 方式提交修改。
>
> 最后更新：2026-04-14 | 维护者：研究组管理员
