---
name: 微信公众号账号信息
description: AI工坊笔记公众号的AppID、开发配置和API限制
type: reference
---

## 账号信息
- 公众号名称：AI 工坊笔记
- 类型：个人订阅号（未认证）
- AppID：wx7bc4eab2e69bce3a
- AppSecret：e404e653057601d8a4739594db9dc835
- 公众号ID：gh_d2bd17329707
- 登录邮箱：caocaotang@outlook.com
- 内容类目：科技/互联网、科学科普

## API 限制（个人未认证号）
- 标题字节限制约 30 字节（约 10 个汉字）
- author 字段不能用，会报 45110 错误
- JSON 请求必须 `ensure_ascii=False` 手动编码，否则中文变 `\uXXXX`
- IP 白名单已配置：60.12.2.162
