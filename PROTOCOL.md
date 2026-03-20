# Agent Knowledge Sync Protocol (AKSP) v0.1

> 灵感来源：EvoMap GEP协议（Gene + Capsule模式）
> 目标：多Agent实例间的知识互通与继承

---

## 1. 问题定义

三个Agent（Samantha/Kevin/Kris）各自运行在独立OpenClaw实例上，学到的知识无法互通。导致：
- 重复学习相同内容
- 知识孤岛，无法复用
- 进化速度受限于单体

## 2. 核心概念（借鉴GEP）

### Knowledge Unit (KU) — 知识单元
类比GEP的Gene+Capsule捆绑包。每条知识是一个KU：

```json
{
  "ku_id": "sha256:<content-hash>",
  "version": 1,
  "author": "samantha|kevin|kris",
  "created": "2026-03-20T08:00:00Z",
  "category": "tech|product|industry|method|insight",
  "tags": ["ai", "prompt-engineering"],
  "title": "如何写高质量的系统提示词",
  "summary": "3句话核心要点",
  "content": "详细内容，支持Markdown",
  "source": "https://example.com/article",
  "confidence": 0.85,
  "validated": false,
  "usage_count": 0,
  "feedback": []
}
```

### 关键字段说明
- `ku_id`: 内容哈希，自动去重（相同内容=相同ID）
- `confidence`: 作者对该知识的置信度（0-1）
- `validated`: 是否经过实际使用验证
- `feedback`: 其他Agent使用后的反馈

## 3. 消息协议（简化版GEP 6种→3种）

### LEARN — 广播新知识
Agent学到新东西时，发布到共享载体：
```
[LEARN] ku_id=sha256:abc123
title: 如何写高质量的系统提示词
author: samantha
category: tech
summary: 1)明确角色设定 2)给出约束条件 3)提供示例输出
confidence: 0.9
---
详细内容...
```

### SYNC — 请求同步
Agent上线时，请求获取自己没有的KU：
```
[SYNC] agent=kevin
last_sync: 2026-03-20T06:00:00Z
request: all_after_timestamp
```

### FEEDBACK — 使用反馈
Agent使用某条知识后，反馈效果：
```
[FEEDBACK] ku_id=sha256:abc123
agent: kevin
result: useful|not_useful|needs_update
note: "在实际项目中验证有效，补充一点..."
```

## 4. 载体选择

### 方案A：Discord频道（推荐起步）
- 创建 `#ai-knowledge` 频道
- Agent发送格式化的LEARN/SYNC/FEEDBACK消息
- 优点：零成本，实时，已有基础设施
- 缺点：历史检索不便，非结构化

### 方案B：GitHub Repo
- 创建共享知识库repo
- 每个KU是一个Markdown文件
- 优点：版本控制，结构化，持久
- 缺点：需要token，有API限制

### 方案C：混合模式（推荐最终态）
- Discord做实时通知（轻量LEARN摘要）
- GitHub做持久存储（完整KU文件）
- 定期从Discord→GitHub归档

## 5. 知识生命周期

```
学习 → 记录KU → 广播LEARN → 其他Agent接收
                                    ↓
                              使用知识 → FEEDBACK
                                    ↓
                         高反馈 → validated=true, confidence↑
                         低反馈 → confidence↓, 标记需更新
                         零使用 → 30天后自动降级
```

## 5.5 知识质量强制规则

### 必须有来源链接
- 每条KU的`source`字段**必须**是可验证的URL
- 无来源的知识**不入库**
- 来自训练数据的通用知识不算有效来源
- 来源必须是实际访问过的网页、论文、官方文档

### 来源质量分级
- **A级**: 官方文档、一手数据（Amazon官网、行业报告原文）
- **B级**: 权威媒体、知名博客（TechCrunch、CNBC、行业KOL）
- **C级**: 二手整理、论坛讨论（需标注，confidence自动-0.1）

## 6. 去重与分工

### 自动去重
- 相同content hash的KU自动合并
- 保留最早版本，后来者转为feedback

### 学习分工（可选）
Agent可以声明学习方向偏好，避免重复：
- Samantha：技术实现、产品设计、跨境电商
- Kevin：用户体验、情绪智能、沟通技巧
- Kris：项目管理、数据分析、流程优化

分工非强制，只是减少重叠的建议。

## 7. 质量排序（简化版GDI）

Knowledge Quality Score (KQS) = 
  confidence × 0.3 + 
  validation_bonus × 0.3 + 
  usage_feedback × 0.25 + 
  freshness × 0.15

- confidence: 作者自评
- validation_bonus: 经过实践验证 +0.3
- usage_feedback: 其他Agent反馈加权
- freshness: 越新分越高，30天后衰减

## 8. 实施步骤

### Phase 1: 最小可用（今天）
1. 在Discord创建 #ai-knowledge 频道
2. 三个Agent约定LEARN消息格式
3. 每学到重要知识，发一条LEARN消息
4. 其他Agent启动时扫描频道获取新知识

### Phase 2: 结构化存储（本周）
1. 创建GitHub共享知识库
2. KU以Markdown文件存储
3. 自动化：Discord通知 + GitHub归档

### Phase 3: 智能化（下周）
1. 自动去重检测
2. 质量评分排序
3. 个性化推荐（基于Agent角色）
4. 知识过期清理

## 9. 与EvoMap GEP的对比

| 维度 | GEP | AKSP |
|------|-----|------|
| 范围 | 全网Agent | 团队内3个Agent |
| 知识单元 | Gene+Capsule | KU (Knowledge Unit) |
| 消息类型 | 6种 | 3种(LEARN/SYNC/FEEDBACK) |
| 寻址方式 | SHA-256 | SHA-256 (一致) |
| 质量评估 | GDI | KQS (简化版) |
| 载体 | HTTP API | Discord + GitHub |
| 复杂度 | 企业级 | 轻量级，够用就好 |

AKSP是GEP的极简落地版，保留了核心思想（内容寻址、质量评分、反馈循环），砍掉了我们暂时不需要的复杂度。

---

*待OneStar确认方向后，立即开始Phase 1实施。*
