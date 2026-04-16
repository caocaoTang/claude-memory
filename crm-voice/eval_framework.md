---
name: Eval Framework
description: 意图识别自动化评测框架现状、QA环境问题
type: project
---

## 评测脚本
Java 版：`IntentEvalRunner.java`（@SpringBootTest + Dubbo）
Prompt Key: `qima-crm#m2l-agent#tangcaocao`

## 测试用例（4个录音）
52894720（烘焙续费）/ 53027928（餐饮培训）/ 53087584（超市拜访）/ 53143622（烘焙复盘）

## 当前状态
- 4份报告全部 0% 召回（QA 环境 aigc-common-service LLM 后端超时）
- 飞书报告：https://qima.feishu.cn/docx/XjIGdtSi8oc87Dxb2vkcuz8pnRc

## 待办
- QA aigc 服务恢复后重跑 eval

**Why:** QA 环境 LLM 后端持续超时，eval 无法产出有效结果
