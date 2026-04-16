---
name: Fill enrichment architecture
description: crm-master Fill 信息补全四源并行架构和实施进展
type: project
---

crm-master 客户信息补全模块，2026-04-13 确定方案，已基本实现。

## 四数据源并行
- AMAP（高德）：门店地址、门店电话 — 有置信度
- Qixinbao（启信宝）：企业名称、KP(法人)、企业电话、邮箱、经营范围、企业地址 — 有置信度
- Network（联网搜索）：小红书、公众号、小程序、商家简介 — 无置信度
- Youzan（有赞小程序）：是否开通 — 依赖 AMAP+Qixinbao 手机号结果

## 已实现功能（截至 2026-04-16）
- 四数据源并行编排 + 重试机制（最多3次，递增延迟）
- 启信宝 top 3 候选匹配，综合评分（名称+地址+经营范围）
- Fill 表格内补全入口，进度条实时渲染
- 置信度徽章在门店/企业信息区块标题内，统一蓝色标签
- 额度管理：每次最多10条，每日最多20条，Redis 原子计数
- 空行过滤：前后端双重保护
- 新建空白表格 + 表格重命名

**Why:** 销售需要批量补全客户企业信息，提升拓客效率
**How to apply:** FillEnrichmentService 独立于 EnrichmentService，不改动老流程
