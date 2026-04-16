---
name: CRM Voice architecture
description: crm-voice 实时访中助手完整架构、处理链路、性能优化方向
type: project
---

## 项目位置
`/Users/tangcaocao/workspace/crm-voice`

## 架构
Spring Boot + Dubbo + NSQ 微服务（Java 17），阿里云听悟 WebSocket 实时 ASR（双通道）

## 完整实时处理链路
```
ASR(听悟) → TranscriptionResultChanged(中间结果) → 仅推前端打字机
         → SentenceEnd(终态) → NSQ(voice_final_text)
  → VoiceFinalTextConsumer → LLM意图识别
    → NSQ(crm_voice_analysis) + Redis pending
  → VoiceAnalysisConsumer → 知识库查询 → LLM答案生成 → Redis completed
  → SSEPushScheduler(500ms轮询) → SSE → 前端
```

## 意图类型
`knowledge_query` / `pain_point` / `opportunity` / `case_query` / `none`

## 触发条件
- 关键词触发（字数≥8）/ 长句触发（≥80字）/ 时间触发（≥30字+停顿≥10s）
- 置信度≥0.7 + 非重复 + 非none

## 当前痛点：响应太慢（5-15s）
已讨论快慢双通道优化方案：利用 stash_result 做规则预判，命中明确意图直接查知识库，跳过 LLM 意图识别。

**Why:** CSM 实时通话需即时提示，当前延迟严重影响体验
**How to apply:** 后续围绕快慢双通道做具体设计和编码
