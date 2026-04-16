---
name: Qixinbao enrichment feature design
description: crm-master 客户信息补全新增启信宝数据源的完整设计方案
type: project
---

crm-master 客户信息补全模块新增启信宝（Qixinbao）数据源，2026-04-13 确定方案。

## 整体架构

四个数据源并行处理：
- AMAP（高德）：门店地址、门店电话 — 有置信度
- Qixinbao（启信宝）：企业名称、KP(法人)、企业电话、邮箱 — 有置信度
- Network（联网搜索）：小红书、公众号、小程序、商家简介 — 无置信度，搜到直接填
- Youzan（有赞小程序）：是否开通 — 依赖AMAP和Qixinbao的手机号结果，需等它们完成

## 启信宝 API
- 1.31 企业模糊搜索：`GET https://api.qixin.com/APIService/v2/search/advSearch`（keyword + region + matchType + skip）
- 1.51 企业联系方式：`GET https://api.qixin.com/APIService/enterprise/getContactInfo`（keyword=企业全名/注册号/信用代码）
- 认证：Header 里 appkey + timestamp + sign（MD5(appkey+timestamp+secret_key)）+ Auth-Version=2.0
- 1.51 仅在模糊搜索置信度≥阈值时调用，节省API配额

## 执行流程
1. LLM提取品牌名
2. AMAP + Qixinbao + Network 三路并行（省市区必填，不存在依赖）
3. Qixinbao: 1.31模糊搜索 → LLM评分 → 置信度够高则调1.51查联系方式
4. AMAP和Qixinbao完成后，汇总手机号 → 查有赞小程序绑定
5. 各数据源独立推送事件，前端独立渲染

## 交互设计
- Fill表格内操作，点击"启信宝补全"触发
- 省市区合为一列，必填
- 置信度用颜色标识：🟢≥80自动填入、🟡40-79标黄可点击查看候选、🔴<40不填
- 高德和启信宝各自独立置信度，联网搜索和有赞无置信度
- Copilot item卡片三栏布局（高德|启信宝|联网搜索）+ 有赞小程序状态
- 表格顶部汇总：高可信/需确认/未匹配条数

## 线程池
每个数据源独立线程池，绝对不能复用（用户强调会卡死）

## Region映射
省市区中文名 → 国家区划码（精确到城市级4位），内置静态映射表

**Why:** 销售需要批量补全客户企业信息（KP、联系方式），提升拓客效率

**How to apply:** 实现时参考现有 EnrichmentService 的异步处理模式，新增 QixinbaoService、独立线程池、事件流扩展
