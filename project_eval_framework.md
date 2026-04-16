---
name: Eval Framework
description: 意图识别自动化评测框架现状、QA环境问题、飞书报告
type: project
---

## 评测脚本

### Python 版（已废弃）
`/Users/tangcaocao/workspace/crm-voice/tests/eval_intent.py`
- LiteLLM HTTP API 不支持 prompt_key（400 错误），已废弃

### Java 版（当前使用）
`/Users/tangcaocao/workspace/crm-voice/crm-voice-web/src/test/java/com/youzan/crm/voice/web/eval/IntentEvalRunner.java`

核心设计：
- `@SpringBootTest` + `@ActiveProfiles("dev")`，走 Dubbo → UnifiedModelService
- Prompt Key: `qima-crm#m2l-agent#tangcaocao`（2026-04-10 已修正）
- 消息格式复刻 `LLMClient.analyzeIntent()`

运行方式：
- IDE: `IntentEvalRunner#runAll()` 或单个 `run52894720()`
- 命令行: `mvn test -pl crm-voice-web -Dtest=IntentEvalRunner#runAll -Dspring.profiles.active=dev`

## 测试用例（4个录音）

| 文件 | 场景 | should_detect | should_not_detect |
|------|------|---------------|-------------------|
| `tests/cases/52894720.json` | 烘焙店续费/降版本 | 8 | 5 |
| `tests/cases/53027928.json` | 餐饮新商家上手培训 | 13 | 3 |
| `tests/cases/53087584.json` | 超市现场拜访+硬件调试 | 6 | 3 |
| `tests/cases/53143622.json` | 烘焙连锁店运营复盘 | 5 | 3 |

## 当前状态
- 4 份报告已生成但全部 0% 召回（LLM 调用失败）
- 原因：QA 环境 aigc-common-service LLM 后端超时（116s timeout）
- 报告已上传飞书：https://qima.feishu.cn/docx/XjIGdtSi8oc87Dxb2vkcuz8pnRc

## 待办
- QA aigc 服务恢复后重新跑 eval 生成有效报告
- 用有效报告数据更新飞书文档

**Why:** QA 环境 aigc-common-service 的 LLM 后端持续超时，导致 eval 无法产出有效结果
