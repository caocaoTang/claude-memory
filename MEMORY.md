# Memory Index

## 记忆管理规则

按项目/话题分类存储，每个分类独立双队列淘汰（A1 观察队列 → Am 核心队列）。
- 新记忆进 A1，引用 ≥2 次晋升 Am
- A1 满时淘汰最老条目，Am 满时淘汰最久未引用条目
- 各分类独立淘汰，互不影响
- 每次读取/引用记忆时更新 _queue.json 的 hits 和 lastHit

## 分类目录

### _global/ — 跨项目（用户画像、通用偏好）
- [user_profile.md](_global/user_profile.md) — 有赞产品研发，CRM AI 工具
- [feedback_thread_pool.md](_global/feedback_thread_pool.md) — 线程池必须独立，不能复用
- [pip_conf_corruption.md](_global/pip_conf_corruption.md) — /Library 下 pip.conf 被 null 字节污染，pip install 报 embedded null byte 时查它

### crm-master/ — 智能CRM Agent 主应用
- [deployment.md](crm-master/deployment.md) — JS 双版本保留防滚动发布 404；merge_pre 分支命名+检测；专用 pre-deploy skill

### crm-master-fill/ — Fill 信息补全平台
- [enrichment_design.md](crm-master-fill/enrichment_design.md) — 四源并行架构，已基本实现
- [qxb_api.md](crm-master-fill/qxb_api.md) — 启信宝 API 接入细节，IP 白名单待配

### crm-voice/ — 实时访中智能助手
- [architecture.md](crm-voice/architecture.md) — 完整链路、快慢双通道优化方案
- [eval_framework.md](crm-voice/eval_framework.md) — 评测框架，QA aigc 超时待恢复

### salesbuddy/ — 销售助手
- [overview.md](salesbuddy/overview.md) — AI 录音分析+客户管理，Vue 2 前端
- [product_positioning.md](salesbuddy/product_positioning.md) — 工具→Agent/虚拟中层，1000 人规模是隐性 deadline
- [v3_prototype_spec.md](salesbuddy/v3_prototype_spec.md) — v3 HTML 是能力参考（字段 schema），布局不照抄
- [first_view_todo.md](salesbuddy/first_view_todo.md) — 工作台首页=今日待办 TODO 流，按必做/重要/加分分组，v3 仅参考
- [spiced_framework.md](salesbuddy/spiced_framework.md) — SPICED（非 SPACE）6 维度，覆盖率+缺口+动作三段式
- [tl_role.md](salesbuddy/tl_role.md) — TL=催节点+定时机，不做面谈沉淀，销售端质量评估已承载诊断
- [meeting_2026_04_20.md](salesbuddy/meeting_2026_04_20.md) — 销售端优先+TL demo 本周+类目按有赞+美业业务自收集
- [feedback_gap_not_percentage.md](salesbuddy/feedback_gap_not_percentage.md) — SPICED 禁止裸进度条，必带缺口描述

### crm-feishu-voice/ — 飞书语音录制+实时转写
- [architecture.md](crm-feishu-voice/architecture.md) — Java17+Vue3，听悟ASR，WebSocket实时流，多平台录音
- [motivation.md](crm-feishu-voice/motivation.md) — 妙记拿不到销售私聊/外呼录音；自研价值在流程化不在覆盖

### wechat-mp/ — 微信公众号「AI 工坊笔记」
- [account.md](wechat-mp/account.md) — AppID/Secret、API限制（个人未认证号）
- [content_plan.md](wechat-mp/content_plan.md) — 7栏目日更规划、黑金风格、发布工具链
- [article_log.md](wechat-mp/article_log.md) — 已发布文章清单+话题关键词，防重复
- [feedback_signature.md](wechat-mp/feedback_signature.md) — 文末署名沿用统一格式，禁止自创个人化署名
- [feedback_no_article_ordering.md](wechat-mp/feedback_no_article_ordering.md) — 正文禁止上一篇/下一篇/第 N 篇等跨篇顺序引用
- [mobile_preview_workflow.md](wechat-mp/mobile_preview_workflow.md) — 推草稿前用 mobile_preview.py 截手机屏自测

### _refs/ — 外部资源引用
- [github_memory_sync.md](_refs/github_memory_sync.md) — 记忆自动同步 caocaoTang/claude-memory
