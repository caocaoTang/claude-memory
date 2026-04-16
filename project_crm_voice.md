---
name: CRM Voice Project
description: crm-voice 项目架构、完整处理链路、已完成改动、当前性能优化方向
type: project
---

## 项目位置
`/Users/tangcaocao/workspace/crm-voice`

## 架构
- Spring Boot + Dubbo + NSQ 微服务（Java 17）
- 阿里云听悟 WebSocket 实时 ASR（双通道：employee/customer）
- LLM 意图识别：通过 AgentForge 平台管理 prompt，走 Dubbo → UnifiedModelService（aigc-common-service）
- Prompt key 格式：`{bizSource}#{scene}#{tempName}`
- QA 环境 prompt key：`qima-crm#m2l-agent#tangcaocao`（已从 `qima-crm#ai-crm-agent#test` 改过来）

## 完整实时处理链路

```
ASR(听悟) → TranscriptionResultChanged(中间结果,含stash_result) → 仅推前端打字机效果
         → SentenceEnd(终态) → NSQ(voice_final_text)
  → VoiceFinalTextConsumer
    → LLMClient.analyzeIntent()【LLM调用1：意图识别】
    → shouldTriggerAnalysis(confidence≥0.7, 非重复, 非none)
    → NSQ(crm_voice_analysis) + Redis pending
  → VoiceAnalysisConsumer
    → 知识库查询(ThinkTank/CaseSearch)
    → LLMClient.generateAnswer()【LLM调用2：答案生成】
    → Redis completed
  → SSEPushScheduler(500ms轮询Redis)
  → SSE → 前端
```

## 关键文件
- `VoiceWebSocketHandler.java` — WebSocket 处理与转发
- `AliyunTingwuASRServiceImpl.java` — 听悟集成，handleTingwuMessage()
- `VoiceFinalTextConsumer.java` — 终态文本消费，触发意图分析
- `VoiceAnalysisConsumer.java` — 知识库查询 + 答案生成
- `LLMClient.java` — LLM 调用封装
- `LlmCommonService.java` — Dubbo → UnifiedModelService 封装
- `SSEPushScheduler.java` — 500ms 轮询 Redis 推 SSE
- `AIAnalysisServiceImpl.java` — 触发条件判断

## 意图类型
`knowledge_query` / `pain_point` / `opportunity` / `case_query` / `none`

## 触发条件（shouldTriggerAnalysis）
- 关键词触发（字数≥8）
- 长句触发（字数≥80）
- 时间触发（字数≥30 + 停顿≥10s）
- 置信度≥0.7 + 非重复 + 非none

## 听悟 ASR 关键信息
- 已开启 `enable_intermediate_result: true`，output_level=2
- 中间结果事件：TranscriptionResultChanged / SentenceResultChanged
- 终态事件：SentenceEnd
- **stash_result**：中间结果中已稳定不会变的文本（当前代码未使用）
- 中间结果仅转发前端，不触发任何分析
- 远程会议模式支持双通道(employee/customer)，各有独立 asrSubSessionId

## Dubbo 环境配置
- dev/qa 都走 tether-qa: `dubbo://tether-qa.s.qima-inc.com:8700`
- Tether HTTP 网关(8680)需要 header `X-Request-Protocol: dubbo`
- QA 的 aigc-common-service 存在 LLM 后端超时问题（116s超时），prod 基本正常

## 已完成的改动

### 1. System Prompt 重构
- 删除 Chapter 13（死占位符）、Chapter 16（执行指令移到 user prompt）
- 去重规则改为引用 user message 里的 `[已识别意图列表]`

### 2. LLMClient.java 改动
- 对话历史改为单个 UserChatMessage（含说话人标签）
- formatUserPrompt 重写，移除与 system prompt 冲突的去重指令

### 3. Eval Runner PROMPT_KEY 修正
- 从 `qima-crm#ai-crm-agent#test` 改为 `qima-crm#m2l-agent#tangcaocao`

## 当前用户痛点：响应太慢

用户反馈：实时对话中意图识别和问题响应延迟太大，CSM 说完话需要等很久才看到提示。

### 延迟瓶颈分析
| 环节 | 预估耗时 | 说明 |
|------|----------|------|
| 等断句(SentenceEnd) | 0-20s | 长句时等用户说完才开始处理 |
| LLM调用1(意图识别) | 2-5s | Dubbo→aigc→Azure |
| NSQ跳转 | 100-500ms | voice_final_text→crm_voice_analysis |
| 知识库查询 | 1-3s | ThinkTank + CaseSearch |
| LLM调用2(答案生成) | 2-5s | 又一次完整LLM |
| SSE轮询 | 0-500ms | 固定500ms polling |
| **总计** | **~5-15s（说完后）** | |

### 已讨论的优化方案：快慢双通道

**核心思路**：利用听悟 stash_result（已稳定中间文本）做分级预处理，不等断句完成。

**快通道（规则驱动，覆盖~60-70%意图）**：
1. stash_result 更新时，本地规则引擎检测"疑问词+功能词"模式
2. 命中明确知识问答 → 立即用稳定关键词查知识库（与用户说话并行）
3. SentenceEnd 到达时知识库已就绪 → 跳过 LLM 意图识别 → 直接 1 次 LLM 生成答案（流式输出）
4. 目标延迟：说完后 1-2s 开始出答案

**慢通道（LLM驱动，兜底）**：
- pain_point / opportunity / 模糊表达 → 等 SentenceEnd → 走原流程
- 目标延迟：与当前一致

**其他优化项**：
- LLM 改流式输出（chatCompletionStreamWithCache），体感首字 <1s
- 去掉 NSQ 跳转，VoiceFinalTextConsumer 直接处理
- SSE 改推送模式，去掉 500ms 轮询
- 知识库查询并行化 + 商家档案 session 预热

### 待落地的改动点
1. `AliyunTingwuASRServiceImpl` 解析 stash_result，加规则引擎预判
2. 新增 PreFetchService 在中间结果阶段异步查知识库
3. `VoiceFinalTextConsumer` 检查预取结果，有则跳过意图识别直接生答案
4. LLM 调用改流式

**Why:** CSM 在实时通话中需要立刻看到意图识别和建议答案，当前 5-15s 延迟严重影响使用体验
**How to apply:** 后续围绕这个优化方向做具体设计和编码
