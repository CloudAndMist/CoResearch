# IMA Skills API 使用指南

> **适用对象**：管理员  
> **目标**：使用 WorkBuddy 自动化操作 IMA 知识库

---

## 一、IMA Skills API 概述

IMA Skills 是腾讯 IMA 提供的开放 API 接口，允许外部程序（如 WorkBuddy）通过 API Key 访问和操作 IMA 知识库。

### 支持的操作

| 操作类型 | 功能说明 | 典型用途 |
|---------|---------|---------|
| **读取** | 获取知识库列表 | 列出所有可访问的知识库 |
| **搜索** | 知识库内容搜索 | 查找特定主题的笔记/文档 |
| **写入** | 新建或追加文本笔记 | 自动记录会议纪要 |
| **上传** | 上传文件到指定知识库 | 批量导入论文 PDF |
| **收藏** | 将网页/微信文章加入知识库 | 自动保存有价值的内容 |

---

## 二、获取 API Key

### 步骤 1：访问 IMA Skills 专区

1. 打开 IMA 客户端或访问网页版：https://ima.qq.com
2. 点击左下角头像 → 设置
3. 找到 **"IMA Skills"** 或 **"开发者选项"** 入口

或者直接访问：**https://ima.qq.com/agent-interface**

### 步骤 2：生成 API Key

1. 进入 "灵感随时 claw" 入口
2. 在 "安装引导" 第 1 步点击 **"一键复制"** 安装指令
3. 在第 2 步获取 API Key

⚠️ **重要提示**：
- API Key **只显示一次**，请立即复制保存
- 如果丢失，需要撤销并重新生成
- 建议保存在密码管理器或团队机密库中

### 步骤 3：验证 API Key

获取 API Key 后，格式类似：
```
ima_sk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## 三、API 调用方式

### 基础信息

| 项目 | 内容 |
|------|------|
| API 基础地址 | `https://api.ima.qq.com/v1`（推测，以官方文档为准） |
| 认证方式 | Bearer Token |
| 请求格式 | JSON |

### 请求头示例

```http
Authorization: Bearer YOUR_IMA_API_KEY
Content-Type: application/json
```

### 常用 API 端点

> ⚠️ 以下端点基于 IMA Skills 能力推测，实际请以官方文档为准

#### 1. 获取知识库列表

```http
GET /knowledge-bases
```

**响应示例**：
```json
{
  "knowledge_bases": [
    {
      "id": "kb_123456",
      "name": "个人知识库",
      "type": "personal",
      "created_at": "2025-01-15T10:00:00Z"
    },
    {
      "id": "kb_789012",
      "name": "XX研究组共享库",
      "type": "shared",
      "created_at": "2025-04-01T08:00:00Z"
    }
  ]
}
```

#### 2. 搜索知识库内容

```http
POST /knowledge-bases/{kb_id}/search
Content-Type: application/json

{
  "query": "Transformer 注意力机制",
  "limit": 10,
  "time_range": {
    "start": "2025-04-07T00:00:00Z",
    "end": "2025-04-14T23:59:59Z"
  }
}
```

**响应示例**：
```json
{
  "results": [
    {
      "id": "doc_001",
      "title": "Attention Is All You Need 笔记",
      "content_preview": "这篇论文提出了 Transformer 架构...",
      "source_type": "pdf",
      "created_at": "2025-04-10T14:30:00Z",
      "author": "张三"
    }
  ]
}
```

#### 3. 创建文本笔记

```http
POST /knowledge-bases/{kb_id}/notes
Content-Type: application/json

{
  "title": "周会议题总结 - 2025-04-12",
  "content": "## 议题 1：...",
  "tags": ["会议纪要", "周会"]
}
```

#### 4. 上传文件

```http
POST /knowledge-bases/{kb_id}/files
Content-Type: multipart/form-data

file: [二进制文件内容]
metadata: {
  "title": "论文标题",
  "tags": ["论文", "Transformer"]
}
```

---

## 四、WorkBuddy 自动化集成

### 方案：使用 WorkBuddy 调用 IMA API

在 WorkBuddy 自动化配置中，可以使用以下方式调用 IMA API：

#### 配置示例

```toml
name = "研究组周会总结（IMA集成）"
schedule = "每周六 17:00"
status = "active"

prompt = """
请执行研究组周会总结任务，通过 IMA Skills API 获取数据：

## 配置信息
IMA_API_KEY = "{{IMA_API_KEY}}"
FEISHU_WEBHOOK = "{{FEISHU_WEBHOOK}}"
SHARED_KB_ID = "{{SHARED_KB_ID}}"  # 共享知识库ID

## 任务步骤

### 步骤 1：获取本周新增内容
调用 IMA API 获取共享知识库本周新增的笔记/文档：

```
GET https://api.ima.qq.com/v1/knowledge-bases/{SHARED_KB_ID}/entries
Headers:
  Authorization: Bearer {IMA_API_KEY}
  Content-Type: application/json

Query Parameters:
  start_date: {{last_saturday_date}}T00:00:00Z
  end_date: {{today_date}}T23:59:59Z
  limit: 50
```

### 步骤 2：分析内容生成议题
使用 LLM 分析获取到的内容，生成 3-5 个讨论议题。

每个议题包含：
- 标题
- 背景（1-2句话）
- 讨论点（2-3个）
- 相关文献链接

### 步骤 3：推送到飞书
将生成的议题通过飞书机器人 Webhook 发送到群组。

## 输出格式

📝 **本周组会议题**（{{date}}）

### 议题 1：[主题名称]
**背景**：简述这个议题的来源和重要性

**讨论点**：
- 问题 1
- 问题 2
- 问题 3

**相关文献**：
- [论文标题](IMA内链接)

---

## 注意事项
1. 如果 IMA API 调用失败，记录错误日志
2. 如果本周没有新内容，发送提示消息
3. 确保链接可点击
4. 使用适当的 emoji 增加可读性
"""
```

---

## 五、Python 脚本示例

如果需要更复杂的逻辑，可以使用 Python 脚本调用 IMA API：

```python
#!/usr/bin/env python3
"""
IMA Skills API 调用示例
用于 WorkBuddy 自动化或独立脚本
"""
import os
import requests
import json
from datetime import datetime, timedelta

# 配置
IMA_API_KEY = os.environ.get('IMA_API_KEY')
IMA_API_BASE = "https://api.ima.qq.com/v1"  # 请以官方文档为准

class IMAClient:
    """IMA Skills API 客户端"""
    
    def __init__(self, api_key):
        self.api_key = api_key
        self.headers = {
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        }
    
    def list_knowledge_bases(self):
        """获取知识库列表"""
        url = f"{IMA_API_BASE}/knowledge-bases"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def search_knowledge_base(self, kb_id, query, time_range=None, limit=20):
        """搜索知识库内容"""
        url = f"{IMA_API_BASE}/knowledge-bases/{kb_id}/search"
        
        payload = {
            "query": query,
            "limit": limit
        }
        
        if time_range:
            payload["time_range"] = time_range
        
        response = requests.post(url, headers=self.headers, json=payload)
        response.raise_for_status()
        return response.json()
    
    def get_recent_entries(self, kb_id, days=7):
        """获取最近 N 天的内容"""
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days)
        
        # 格式化时间
        time_range = {
            "start": start_date.strftime("%Y-%m-%dT%H:%M:%SZ"),
            "end": end_date.strftime("%Y-%m-%dT%H:%M:%SZ")
        }
        
        # 使用空查询获取所有内容
        return self.search_knowledge_base(
            kb_id=kb_id,
            query="",
            time_range=time_range,
            limit=50
        )
    
    def create_note(self, kb_id, title, content, tags=None):
        """创建文本笔记"""
        url = f"{IMA_API_BASE}/knowledge-bases/{kb_id}/notes"
        
        payload = {
            "title": title,
            "content": content
        }
        
        if tags:
            payload["tags"] = tags
        
        response = requests.post(url, headers=self.headers, json=payload)
        response.raise_for_status()
        return response.json()


def generate_weekly_summary(ima_client, shared_kb_id):
    """生成本周总结"""
    # 1. 获取本周内容
    entries = ima_client.get_recent_entries(shared_kb_id, days=7)
    
    if not entries.get('results'):
        return None, "本周暂无新增内容"
    
    # 2. 格式化内容供 LLM 分析
    content_text = "\n\n".join([
        f"标题：{entry['title']}\n内容：{entry.get('content_preview', 'N/A')}\n作者：{entry.get('author', '未知')}"
        for entry in entries['results']
    ])
    
    return entries['results'], content_text


# 使用示例
if __name__ == '__main__':
    # 初始化客户端
    client = IMAClient(IMA_API_KEY)
    
    # 获取知识库列表
    kbs = client.list_knowledge_bases()
    print("可用知识库：")
    for kb in kbs.get('knowledge_bases', []):
        print(f"  - {kb['name']} (ID: {kb['id']})")
    
    # 获取本周内容（需要替换为实际的共享知识库ID）
    # shared_kb_id = "kb_xxxxxx"
    # entries, content = generate_weekly_summary(client, shared_kb_id)
    # print(f"获取到 {len(entries)} 条内容")
```

---

## 六、响应延迟参考

| 操作类型 | 预计耗时 | 说明 |
|---------|---------|------|
| 列库/搜索 | 1-3 秒 | 快速响应 |
| 创建文本笔记 | 2-5 秒 | 包含内容处理 |
| 上传大文件/批量写入 | 10 秒至数分钟 | 取决于文件大小 |

---

## 七、故障排查

### 问题 1：API Key 无效

**症状**：返回 401 Unauthorized

**解决**：
1. 确认 API Key 正确复制（无多余空格）
2. 检查 API Key 是否已过期或被撤销
3. 重新生成 API Key

### 问题 2：知识库访问权限不足

**症状**：返回 403 Forbidden

**解决**：
1. 确认 API Key 所属账号有权限访问该知识库
2. 检查知识库是否设置了访问限制
3. 对于共享知识库，确认已被邀请加入

### 问题 3：内容解析失败

**症状**：网页/微信文章解析不完整

**解决**：
1. 网页结构复杂可能导致解析失败
2. 尝试重新上传或手动复制内容
3. 对于重要内容，建议保存为 PDF 后上传

---

## 八、安全建议

1. **API Key 管理**
   - 不要将 API Key 硬编码在代码中
   - 使用环境变量或密钥管理服务
   - 定期轮换 API Key

2. **权限控制**
   - 使用最小权限原则
   - 团队环境建议使用专用账号
   - 开启操作日志记录

3. **内容安全**
   - 敏感内容谨慎上传
   - 定期检查知识库访问日志

---

## 九、参考资源

- **IMA Skills 官方文档**：https://ima.qq.com/agent-interface
- **IMA 帮助中心**：https://ima.qq.com/help
- **API 状态页**：（如有）

---

## 十、更新日志

- **v1.0** (2026-04-14): 初始版本，基于 IMA Skills 公开信息整理

---

> ⚠️ **重要提示**：本文档中的 API 端点和参数基于 IMA Skills 能力推测，实际使用时请以 IMA 官方文档为准。建议在正式使用前进行充分测试。
