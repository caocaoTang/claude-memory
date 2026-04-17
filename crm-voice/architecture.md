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
当前链路全串行：ASR → NSQ → LLM意图 → NSQ → 知识库 → LLM生成 → SSE

**Why:** CSM 实时通话需即时提示，当前延迟严重影响体验

## 业界调研结论（2026-04）
业界标配：快慢双通道 + 渐进式响应，所有生产级系统都用混合方案（Observe.AI、Balto、ASAPP 等）

### 快通道上下文问题的解法
核心挑战：规则命中当前句但主语在上文（代词指代），三种业界方案：
1. **实体槽位追踪**（主流）：Redis 维护 current_entities，代词直接从槽位取最近同类实体填充，纯内存操作覆盖 80% 场景
2. **小模型指代消解改写**（2024-2025 趋势）：用 Haiku 级小模型将"最近3句+当前句"改写为自包含完整句，100-400ms，可与知识库检索并行
3. **前缀预取/投机执行**：ASR 中间结果就开始预检索知识库，最终结果出来时检索大概率已完成

### 延迟预算参考
快通道端到端 <1s（意图50ms + 检索200ms + 流式生成500ms），慢通道 <2s

### 计划优化方向
1. 快通道 + 实体槽位：关键词命中 → 从 Redis entities 补全 → 直接打知识库
2. 代词/模糊主语 → 小模型改写（100-400ms）替代完整 LLM 意图识别
3. 并行化：意图识别与知识库预取同时跑
4. 流式输出：答案生成 streaming 推前端

**How to apply:** 下一步出具体改造方案，对着代码设计快慢通道分流逻辑
