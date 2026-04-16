---
name: Qixinbao API details
description: 启信宝 API 接入细节（认证、接口、IP白名单）
type: project
---

## 接口
- 1.31 企业模糊搜索：`GET https://api.qixin.com/APIService/v2/search/advSearch`
- 1.51 企业联系方式：`GET https://api.qixin.com/APIService/enterprise/getContactInfo`
- 基本信息接口（企业地址、经营范围等）

## 认证
Header: appkey + timestamp + sign（MD5(appkey+timestamp+secret_key)）+ Auth-Version=2.0

## 待办
- QXB IP 白名单待配置（代码已就绪，等基础设施）

**Why:** 启信宝数据源用于补全企业维度信息
**How to apply:** QixinbaoService.java 封装所有 API 调用
