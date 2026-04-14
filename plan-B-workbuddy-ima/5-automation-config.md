# Plan B：自动化配置指南

> **适用对象**：管理员  
> **目标**：配置每周六自动 Lint 检查和生成组会议题

---

## 方案选择

| 方案 | 难度 | 稳定性 | 推荐度 |
|------|------|--------|--------|
| **方案 A：WorkBuddy 自动化** | ⭐ 简单 | ⭐⭐⭐ 高 | ✅ 推荐 |
| **方案 B：GitHub Actions** | ⭐⭐ 中等 | ⭐⭐⭐ 高 | 备用 |

---

## 方案 A：WorkBuddy 自动化（推荐）

### 步骤 1：准备必要信息

你需要收集以下信息：

```yaml
# IMA API Key（从 IMA 设置获取）
IMA_API_KEY: "your_ima_api_key_here"

# IMA 共享知识库 ID
SHARED_KB_ID: "your_shared_kb_id_here"

# LLM API Key（可选，用于生成议题）
LLM_API_KEY: "your_claude_or_gpt_api_key"

# （可选）微信通知配置
WECHAT_ENABLED: true  # 是否启用微信通知
```

### 步骤 2：创建自动化任务

在 WorkBuddy 中执行：

```bash
# 打开 WorkBuddy 自动化面板
# 路径：左侧栏 → 自动化 → 新建自动化
```

配置参数：

```toml
name = "研究组周会总结"
schedule = "每周六 17:00"
status = "active"

prompt = """
请执行研究组周会总结任务：

## 任务步骤

1. **获取本周新增内容**
   - 调用 IMA API 获取共享知识库本周新增的笔记/文档
   - API Key: {{IMA_API_KEY}}
   - 时间范围：上周日至今

2. **生成组会议题**
   - 使用 LLM 分析本周内容
   - 生成 3-5 个讨论议题
   - 每个议题包含：
     * 标题
     * 背景（1-2句话）
     * 讨论点（2-3个）
     * 相关文献链接

3. **保存到 IMA 共享知识库**
   - 创建新笔记：「[{{date}}] 周会议题」
   - 位置：共享知识库/📁 03-组会讨论/
   - 标签：#会议纪要 #自动化生成
   - 内容：Lint 报告摘要 + 组会议题

## 输出格式示例

📝 **本周组会议题**（{{date}}）

### 议题 1：[主题名称]
**背景**：简述这个议题的来源和重要性

**讨论点**：
- 问题 1
- 问题 2
- 问题 3

**相关文献**：
- [论文标题](链接)

---

### 议题 2：[主题名称]
...

## 注意事项
- 如果本周没有新内容，发送提示消息
- 确保链接可点击
- 使用适当的 emoji 增加可读性
"""
```

### 步骤 3：测试自动化

1. 保存自动化配置
2. 点击「手动触发」测试一次
3. 检查 IMA 共享知识库「📁 03-组会讨论」是否生成新笔记

---

## 方案 B：GitHub Actions（备用）

如果你更熟悉代码，可以使用 GitHub Actions。

### 步骤 1：创建 GitHub 仓库

1. 在 GitHub 创建新仓库，命名为 `research-group-automation`
2. 克隆到本地

### 步骤 2：创建工作流文件

创建文件 `.github/workflows/weekly-summary.yml`：

```yaml
name: Weekly Research Group Summary

on:
  schedule:
    # 每周六 17:00 (UTC+8 = UTC 09:00)
    - cron: '0 9 * * 6'
  workflow_dispatch:  # 允许手动触发

jobs:
  generate-summary:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
      
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'
        
    - name: Install dependencies
      run: |
        pip install requests openai
        
    - name: Generate weekly summary
      env:
        IMA_API_KEY: ${{ secrets.IMA_API_KEY }}
        SHARED_KB_ID: ${{ secrets.SHARED_KB_ID }}
        LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
      run: |
        python scripts/generate_summary.py
```

### 步骤 3：创建 Python 脚本

创建文件 `scripts/generate_summary.py`：

```python
#!/usr/bin/env python3
"""
研究组周会总结生成脚本
"""
import os
import requests
import json
from datetime import datetime, timedelta
from openai import OpenAI

# 配置
IMA_API_KEY = os.environ.get('IMA_API_KEY')
SHARED_KB_ID = os.environ.get('SHARED_KB_ID')
LLM_API_KEY = os.environ.get('LLM_API_KEY')

def get_weekly_content():
    """获取本周 IMA 知识库新增内容"""
    # IMA API 调用示例（需要根据实际 API 文档调整）
    headers = {
        'Authorization': f'Bearer {IMA_API_KEY}',
        'Content-Type': 'application/json'
    }
    
    # TODO: 根据 IMA 实际 API 调整
    # response = requests.get(f'https://api.ima.qq.com/v1/knowledge-bases/{SHARED_KB_ID}/entries', headers=headers)
    # return response.json()
    
    # 模拟数据（实际使用时替换为真实 API 调用）
    return [
        {"title": "论文笔记：Transformer 新变体", "date": "2025-04-10"},
        {"title": "实验记录：ResNet 调参", "date": "2025-04-12"},
    ]

def generate_topics(content):
    """使用 LLM 生成组会议题"""
    client = OpenAI(api_key=LLM_API_KEY)
    
    prompt = f"""
基于以下本周研究组知识库新增内容，生成组会议题：

{json.dumps(content, ensure_ascii=False, indent=2)}

请生成 3-5 个讨论议题，每个议题包含：
1. 标题
2. 背景（1-2句话）
3. 讨论点（2-3个）
4. 相关文献

格式为 Markdown。
"""
    
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "你是一位研究组助手，擅长整理学术讨论议题。"},
            {"role": "user", "content": prompt}
        ]
    )
    
    return response.choices[0].message.content

def save_to_ima(title, content, kb_id):
    """保存到 IMA 共享知识库"""
    # TODO: 根据 IMA 实际 API 调整
    # 调用 IMA API 创建新笔记
    print(f"保存到 IMA: {title}")
    return True

def main():
    print("开始生成周会总结...")
    
    # 1. 获取本周内容
    content = get_weekly_content()
    print(f"获取到 {len(content)} 条内容")
    
    if not content:
        print("本周无新增内容")
        return
    
    # 2. 生成议题
    topics = generate_topics(content)
    print("议题生成完成")
    
    # 3. 保存到 IMA
    date_str = datetime.now().strftime('%Y-%m-%d')
    title = f"[{date_str}] 周会议题（自动生成）"
    success = save_to_ima(title, topics, SHARED_KB_ID)
    
    if success:
        print(f"✅ 已保存到 IMA: {title}")
    else:
        print("❌ 保存失败")

if __name__ == '__main__':
    main()
```

### 步骤 4：配置 GitHub Secrets

1. 打开 GitHub 仓库 → Settings → Secrets and variables → Actions
2. 添加以下 Secrets：
   - `IMA_API_KEY`：IMA API Key
   - `SHARED_KB_ID`：IMA 共享知识库 ID
   - `LLM_API_KEY`：LLM API Key（Claude/GPT/DeepSeek）

### 步骤 5：测试

1. 提交代码到 GitHub
2. 进入 Actions 页面
3. 手动触发工作流测试

---

## 方案 C：本地 Python 脚本（轻量）

如果你不想使用 GitHub Actions，可以直接在本地运行 Python 脚本：

### 步骤 1：安装依赖

```bash
pip install requests openai
```

### 步骤 2：创建脚本

使用上面的 `generate_summary.py` 脚本，在本地定时运行（如使用 Windows 任务计划程序或 cron）。

---

## 自动化输出格式

### IMA 笔记格式

生成的组会议题将保存为 IMA 笔记，格式如下：

```markdown
# [2025-04-14] 周会议题（自动生成）

> 自动生成时间：2025-04-14 17:00:00
> 本周新增内容：5 条

---

## 🔍 Lint 报告摘要

- 健康度评分：85/100 🟡
- 发现问题：重复内容 1 组，信息缺失 2 条
- 建议：合并重复笔记，补充缺失信息

---

## 📝 组会议题

### 议题 1：Transformer 新变体调研
**背景**：本周张三分享了一篇关于 Transformer 改进的论文...

**讨论点**：
- 该方法与标准 Transformer 的性能对比
- 是否适用于我们的任务
- 实现复杂度评估

**相关文献**：
- [论文标题](IMA内链接)

---

### 议题 2：实验结果分析
**背景**：李四完成了消融实验...

**讨论点**：
- ...

---

## 💡 备注

- 本议题由 WorkBuddy 自动生成
- 如有问题请联系管理员
```
}
```

---

## 故障排查

### 问题 1：自动化没有触发

- 检查 WorkBuddy/GitHub Actions 运行状态
- 查看日志中的错误信息
- 确认时间设置正确（注意时区）

### 问题 2：IMA API 调用失败

- 确认 API Key 有效
- 检查 API 调用频率限制
- 查看 IMA 官方 API 文档是否有更新

### 问题 3：IMA 笔记保存失败

- 检查 IMA API Key 是否有效
- 确认共享知识库 ID 正确
- 检查 API 调用权限（是否有写入权限）
- 查看 IMA API 返回的错误信息

---

## 进阶：自定义议题生成逻辑

你可以根据需要调整议题生成 Prompt：

```python
CUSTOM_PROMPT = """
你是一位专业的学术研究助手。请基于本周知识库内容，生成组会议题。

## 分析维度
1. **新方法/新技术**：是否有值得关注的论文或技术
2. **实验进展**：实验结果是否达到预期
3. **问题/阻碍**：是否有需要讨论解决的问题
4. **下一步计划**：下周的工作重点

## 输出要求
- 每个议题控制在 100 字以内
- 使用 bullet points 列出讨论点
- 标注出需要重点讨论的内容（用 🔥 标记）

## 本周内容
{content}

请生成议题：
"""
```

---

## 进阶：知识库 Lint（质量检查）

> **参考**：Karpathy LLM Wiki 的 Lint 概念  
> **作用**：让 LLM 作为"质检员"，定期检查知识库的完整性、准确性、一致性

### 为什么需要 Lint

没有 Lint 的知识库可能出现：
- 📛 **重复内容**：同一篇论文被多人记录
- ⚠️ **信息缺失**：笔记缺少关键信息（方法、结论）
- 🔗 **断链**：引用的链接失效或指向错误
- 📊 **标签混乱**：分类不统一，难以检索
- ⏰ **内容过时**：早期笔记需要更新但未标记

### WorkBuddy Lint 自动化配置

在原有"周会总结"自动化基础上，增加 Lint 检查步骤：

#### 配置参数

```toml
name = "研究组知识库 Lint 检查"
schedule = "每周六 16:30"  # 比周会总结提前 30 分钟
status = "active"

prompt = """
请执行研究组知识库 Lint（质量检查）任务：

## 配置信息
IMA_API_KEY = "{{IMA_API_KEY}}"
FEISHU_WEBHOOK = "{{FEISHU_WEBHOOK}}"
SHARED_KB_ID = "{{SHARED_KB_ID}}"

## 任务步骤

### 步骤 1：获取本周及上周内容
调用 IMA API 获取过去 14 天的所有条目：
- 用于检测重复内容
- 用于识别信息缺失

### 步骤 2：执行 Lint 检查

#### 检查项 1：重复内容检测
- 比较标题相似度（>80% 视为重复）
- 比较内容摘要相似度（>70% 视为重复）
- 记录重复条目 ID 和标题

#### 检查项 2：信息完整性检查
- 标题是否清晰（长度 5-100 字）
- 是否有内容摘要（非空检查）
- 是否有来源信息（论文链接、会议名称等）
- 是否有标签/分类

#### 检查项 3：标签规范性检查
- 标签是否统一（如 "Transformer" vs "transformer"）
- 是否有推荐的标准标签

#### 检查项 4：内容质量评估（使用 LLM）
对每篇笔记，使用 LLM 评估：
- 是否包含核心信息（问题、方法、结论）
- 是否有明确的研究价值
- 是否需要补充或更新

### 步骤 3：生成 Lint 报告

报告格式：

```
🔍 知识库 Lint 报告（{{date}}）

## 📊 总体统计
- 本周新增：X 条
- 总条目数：Y 条
- 健康度评分：Z/100

## ⚠️ 发现的问题

### 重复内容（N 组）
- 🔴 [条目A标题] 与 [条目B标题] 内容重复，建议合并

### 信息缺失（M 条）
- 🟡 [条目标题]：缺少来源链接
- 🟡 [条目标题]：内容过短，建议补充

### 标签问题（K 条）
- 🟠 发现不一致标签："NLP" vs "nlp"，建议统一为 "NLP"

### 质量建议（L 条）
- 💡 [条目标题]：建议补充实验结论部分
- 💡 [条目标题]：该论文有后续研究，建议更新

## ✅ 健康条目
- [条目1标题]
- [条目2标题]

## 🎯 建议行动
1. 合并重复条目（预计节省 X 分钟）
2. 补充缺失信息（预计需要 Y 分钟）
3. 统一标签规范
"""

### 步骤 4：推送到飞书
- 仅推送给管理员（可配置）
- 如果健康度 < 70，标记为 🔴 需要关注
- 如果健康度 70-90，标记为 🟡 建议优化
- 如果健康度 > 90，标记为 🟢 状态良好

## 注意事项
- Lint 报告在周会前 30 分钟发送，给管理员预留处理时间
- 重复内容检测使用相似度算法，不是完全匹配
- LLM 质量评估仅供参考，最终判断由人工确认
- 建议每月进行一次"深度 Lint"（检查所有历史内容）
"""
```

#### Python Lint 脚本示例

```python
#!/usr/bin/env python3
"""
知识库 Lint 检查脚本
用于 WorkBuddy 自动化或独立运行
"""
import os
import requests
import json
from datetime import datetime, timedelta
from difflib import SequenceMatcher

# 配置
IMA_API_KEY = os.environ.get('IMA_API_KEY')
FEISHU_WEBHOOK = os.environ.get('FEISHU_WEBHOOK')
LLM_API_KEY = os.environ.get('LLM_API_KEY')

def similarity(a, b):
    """计算两个字符串的相似度"""
    return SequenceMatcher(None, a, b).ratio()

def detect_duplicates(entries, threshold=0.8):
    """检测重复内容"""
    duplicates = []
    n = len(entries)
    
    for i in range(n):
        for j in range(i + 1, n):
            title_sim = similarity(entries[i]['title'], entries[j]['title'])
            content_sim = similarity(
                entries[i].get('content_preview', ''),
                entries[j].get('content_preview', '')
            )
            
            if title_sim > threshold or content_sim > 0.7:
                duplicates.append({
                    'entry_a': entries[i],
                    'entry_b': entries[j],
                    'title_similarity': title_sim,
                    'content_similarity': content_sim
                })
    
    return duplicates

def check_completeness(entry):
    """检查条目完整性"""
    issues = []
    
    # 标题长度检查
    if len(entry['title']) < 5:
        issues.append("标题过短")
    if len(entry['title']) > 100:
        issues.append("标题过长")
    
    # 内容检查
    if not entry.get('content_preview'):
        issues.append("缺少内容摘要")
    elif len(entry.get('content_preview', '')) < 50:
        issues.append("内容过短")
    
    # 来源检查
    if not entry.get('source') and not entry.get('url'):
        issues.append("缺少来源信息")
    
    # 标签检查
    if not entry.get('tags'):
        issues.append("缺少标签")
    
    return issues

def check_tag_consistency(entries):
    """检查标签一致性"""
    tag_variants = {}
    
    for entry in entries:
        for tag in entry.get('tags', []):
            lower_tag = tag.lower()
            if lower_tag not in tag_variants:
                tag_variants[lower_tag] = set()
            tag_variants[lower_tag].add(tag)
    
    # 找出有多个变体的标签
    inconsistent = {k: v for k, v in tag_variants.items() if len(v) > 1}
    return inconsistent

def llm_quality_check(entry, llm_client):
    """使用 LLM 评估内容质量"""
    prompt = f"""
    请评估以下研究笔记的质量：
    
    标题：{entry['title']}
    内容：{entry.get('content_preview', 'N/A')}
    
    请从以下维度评估（1-5分）：
    1. 信息完整性（是否包含问题、方法、结论）
    2. 研究价值（是否有学术/工程价值）
    3. 可理解性（他人能否看懂）
    
    输出格式：
    - 各维度评分
    - 总体评价（优秀/良好/需改进）
    - 具体改进建议（如有）
    """
    
    # 调用 LLM API
    # response = llm_client.chat.completions.create(...)
    # return parse_response(response)
    
    return {"score": 4, "evaluation": "良好", "suggestions": []}

def calculate_health_score(entries, duplicates, completeness_issues, tag_issues):
    """计算知识库健康度评分"""
    if not entries:
        return 100
    
    base_score = 100
    
    # 重复内容扣分
    duplicate_penalty = len(duplicates) * 5
    base_score -= min(duplicate_penalty, 30)
    
    # 完整性问题扣分
    completeness_penalty = len(completeness_issues) * 2
    base_score -= min(completeness_penalty, 30)
    
    # 标签问题扣分
    tag_penalty = len(tag_issues) * 3
    base_score -= min(tag_penalty, 20)
    
    return max(0, base_score)

def generate_lint_report(entries, duplicates, completeness_issues, tag_issues, health_score):
    """生成 Lint 报告"""
    report = f"""🔍 知识库 Lint 报告（{datetime.now().strftime('%Y-%m-%d')}）

## 📊 总体统计
- 检查条目数：{len(entries)} 条
- 健康度评分：{health_score}/100

"""
    
    # 健康度状态
    if health_score >= 90:
        report += "- 状态：🟢 健康\n\n"
    elif health_score >= 70:
        report += "- 状态：🟡 建议优化\n\n"
    else:
        report += "- 状态：🔴 需要关注\n\n"
    
    # 重复内容
    if duplicates:
        report += f"## 🔴 重复内容（{len(duplicates)} 组）\n"
        for dup in duplicates[:5]:  # 最多显示 5 组
            report += f"- 「{dup['entry_a']['title']}」与「{dup['entry_b']['title']}」\n"
            report += f"  相似度：{dup['title_similarity']:.1%}（标题）/{dup['content_similarity']:.1%}（内容）\n"
        report += "\n"
    
    # 完整性问题
    if completeness_issues:
        report += f"## 🟡 信息缺失（{len(completeness_issues)} 条）\n"
        for issue in completeness_issues[:10]:  # 最多显示 10 条
            report += f"- 「{issue['title']}」：{', '.join(issue['issues'])}\n"
        report += "\n"
    
    # 标签问题
    if tag_issues:
        report += f"## 🟠 标签不一致（{len(tag_issues)} 组）\n"
        for tag, variants in tag_issues.items():
            report += f"- 「{tag}」存在变体：{', '.join(variants)}\n"
        report += "\n"
    
    # 建议行动
    report += "## 🎯 建议行动\n"
    if duplicates:
        report += "1. 合并重复条目\n"
    if completeness_issues:
        report += "2. 补充缺失信息\n"
    if tag_issues:
        report += "3. 统一标签规范\n"
    if not any([duplicates, completeness_issues, tag_issues]):
        report += "✅ 知识库状态良好，继续保持！\n"
    
    return report

def save_lint_report_to_ima(report, kb_id):
    """保存 Lint 报告到 IMA 共享知识库"""
    # TODO: 根据 IMA 实际 API 调整
    # 调用 IMA API 创建新笔记
    date_str = datetime.now().strftime('%Y-%m-%d')
    title = f"[{date_str}] Lint 报告（自动生成）"
    print(f"保存 Lint 报告到 IMA: {title}")
    # 实际应调用 IMA API 创建笔记
    return True

def main():
    print("开始执行知识库 Lint 检查...")
    
    # 1. 获取条目（模拟数据，实际应调用 IMA API）
    entries = [
        {"title": "Transformer 论文笔记", "content_preview": "这篇论文提出了...", "tags": ["NLP", "Transformer"]},
        {"title": "transformer 总结", "content_preview": "这篇论文提出了...", "tags": ["nlp", "transformer"]},
        {"title": "短", "content_preview": "", "tags": []},
    ]
    
    # 2. 检测重复
    duplicates = detect_duplicates(entries)
    print(f"发现 {len(duplicates)} 组重复内容")
    
    # 3. 检查完整性
    completeness_issues = []
    for entry in entries:
        issues = check_completeness(entry)
        if issues:
            completeness_issues.append({
                'title': entry['title'],
                'issues': issues
            })
    print(f"发现 {len(completeness_issues)} 条完整性问题")
    
    # 4. 检查标签一致性
    tag_issues = check_tag_consistency(entries)
    print(f"发现 {len(tag_issues)} 组标签不一致")
    
    # 5. 计算健康度
    health_score = calculate_health_score(entries, duplicates, completeness_issues, tag_issues)
    print(f"健康度评分：{health_score}/100")
    
    # 6. 生成报告
    report = generate_lint_report(entries, duplicates, completeness_issues, tag_issues, health_score)
    
    # 7. 保存到 IMA（可选，也可仅在 WorkBuddy 中展示）
    success = save_lint_report_to_ima(report, SHARED_KB_ID)
    if success:
        print("✅ Lint 报告已保存到 IMA")
    else:
        print("❌ Lint 报告保存失败")

if __name__ == '__main__':
    main()
```

### Lint 检查时机建议

| 检查类型 | 频率 | 时间 | 接收人 |
|---------|------|------|--------|
| **快速 Lint** | 每周 | 周六 16:30 | 管理员 |
| **周会总结** | 每周 | 周六 17:00 | 全组成员 |
| **深度 Lint** | 每月 | 月末 | 管理员 |

### Lint 与周会总结的关系

```
周六 16:30        周六 17:00
   │                  │
   ▼                  ▼
┌─────────┐      ┌─────────────┐
│ Lint 检查 │  →  │ 周会总结生成 │
│ 质量报告  │      │ 组会议题生成 │
└─────────┘      └─────────────┘
   │                  │
   ▼                  ▼
WorkBuddy展示    保存到 IMA
（仅管理员）     「📁 03-组会讨论」
                        │
                        ▼
                  周日18:30组会时
                  全员一起查看
```

---

## 总结

| 步骤 | 行动项 |
|------|--------|
| 1 | 选择自动化方案（推荐 WorkBuddy） |
| 2 | 收集 IMA API Key 和共享库 ID |
| 3 | 配置自动化任务 |
| 4 | 测试并验证 |
| 5 | 通知成员开始使用 |

---

> **文档版本**：v1.0  
> **最后更新**：2026-04-14
