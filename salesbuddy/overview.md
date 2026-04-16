---
name: Sales Buddy overview
description: crm-master-salesbuddy 销售助手项目概述、架构、功能模块
type: project
---

## 项目概述
Sales Buddy 是有赞 CRM 的销售助手，AI 分析录音自动提取客户档案、生成跟进建议。
代码路径: `/Users/tangcaocao/workspace/crm-master-salesbuddy`（7 模块 Maven monorepo + Vue.js 2）

## 核心功能
1. **客户管理**: CRUD、档案字段编辑、同名检查
2. **录音分析**: 飞书妙记/文档 → 异步 AI 分析 → Markdown 结果 → 自动合并档案
3. **管理角色**: `sales-operation` 可全局查看所有用户数据（Vesta 角色实时查询）
4. **备注+动态**: 事件表存储，客户详情页时间线展示

## 关键设计
- 录音分析为客户详情页右侧抽屉（35vw）
- 同客户互斥：有 processing 记录时禁用提交
- 前端 Vue 2 语法，构建用 webpack

**Why:** 销售拜访后手动整理客户信息效率低，AI 录音分析自动提取
**How to apply:** 持续迭代中，注意前端是 Vue 2 非 Vue 3
