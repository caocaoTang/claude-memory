---
name: Qixinbao enrichment feature design
description: crm-master 客户信息补全新增启信宝数据源的完整设计方案及实施进展
type: project
---

crm-master 客户信息补全模块新增启信宝（Qixinbao）数据源，2026-04-13 确定方案，已基本实现。

## 整体架构

四个数据源并行处理：
- AMAP（高德）：门店地址、门店电话 — 有置信度
- Qixinbao（启信宝）：企业名称、KP(法人)、企业电话、邮箱、经营范围、企业地址 — 有置信度
- Network（联网搜索）：小红书、公众号、小程序、商家简介 — 无置信度，搜到直接填
- Youzan（有赞小程序）：是否开通 — 依赖AMAP和Qixinbao的手机号结果，需等它们完成

## 已实现功能（截至 2026-04-16）

- 四数据源并行编排 + 重试机制（最多3次，递增延迟）
- 启信宝 top 3 候选匹配：取前3个候选，分别查联系方式/基本信息，综合评分（名称匹配+地址区域+经营范围类目）
- Fill 表格内补全入口，进度条实时渲染
- 置信度徽章显示在门店信息/企业信息区块标题内
- 所有数据源状态标签统一蓝色
- 额度管理：每次最多10条，每日最多20条，Redis 原子计数
- 空行过滤：前后端双重保护，customerName 为空的行不提交
- 新建空白表格：默认4列+30行，表格名称自动去重
- 表格重命名：下拉列表 ✏️ 按钮 + 顶部文件名点击编辑
- 记忆自动同步 GitHub（PostToolUse hook）

## 启信宝 API
- 1.31 企业模糊搜索 + 1.51 联系方式 + 基本信息接口
- 认证：Header appkey + timestamp + sign（MD5）+ Auth-Version=2.0
- QXB IP 白名单待配置（代码已就绪）

## 线程池
每个数据源独立线程池，绝对不能复用（嵌套 CompletableFuture 会导致线程饥饿死锁）

**Why:** 销售需要批量补全客户企业信息（KP、联系方式），提升拓客效率

**How to apply:** FillEnrichmentService 独立于 EnrichmentService，不改动老流程；前端在 fill.ftl 中实现
