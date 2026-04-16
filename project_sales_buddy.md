---
name: Sales Buddy Project
description: crm-master-salesbuddy 项目背景、架构、功能模块、当前进度
type: project
---

# Sales Buddy 项目

## 项目概述
Sales Buddy 是有赞 CRM 的销售助手功能，帮助销售通过 AI 分析录音自动提取客户档案、生成跟进建议。

**Why:** 销售拜访后需要手动整理客户信息，AI 录音分析可自动提取客户档案字段、意向分级、痛点等。

## 代码仓库
- 路径: `/Users/tangcaocao/workspace/crm-master-salesbuddy`
- 基于 crm-master 主仓库，7 模块 Maven monorepo + Vue.js 2 前端
- 前端组件在 `crm-master-front/ui/components/salesbuddy/` 下

## 核心文件
| 文件 | 职责 |
|------|------|
| `sales-buddy.vue` | 父组件，sidebar 导航（工作台 + 客户盘点），全局弹窗 |
| `tab-dashboard.vue` | 工作台：本周统计、最近动态 |
| `tab-customer.vue` | 客户列表（表格/卡片视图）+ 客户详情（左右两栏 1:1） |
| `analysis-drawer.vue` | 录音分析抽屉（从右侧 35vw 滑入），上传+进度+结果展示 |
| `SalesBuddyController.java` | REST 端点，角色检测（sales-operation） |
| `SalesBuddyServiceImpl.java` | 核心业务：客户 CRUD、录音提交、分析轮询、档案合并 |
| `SalesBuddyEventServiceImpl.java` | 事件/动态记录（含备注，存在事件表中） |
| `salesbuddy.js` | 前端 API 层 |

## 功能模块
1. **客户管理**: 新增/编辑/删除客户，客户档案字段编辑，同名检查
2. **录音分析**: 提交飞书妙记/文档链接 → 后端异步 AI 分析 → 轮询进度 → Markdown 结果
   - taskId 存 Redis（前缀 `youzan:crm-master_kv:sb:audio:task:`，TTL 2h）
   - 分析完成后自动合并档案字段到客户
3. **历史录音**: 点击可恢复查看，processing 状态恢复轮询
4. **管理角色**: `sales-operation`（销售运营）可全局查看所有用户数据
   - Vesta 角色实时查询，不缓存
   - 客户列表显示归属销售列（runtime 查 UserDependService）
   - Service 层 userId=null 时跳过用户过滤
5. **备注**: 存在事件表中（NOTE_ADDED 类型），显示在客户动态时间线
6. **客户详情页**: 左栏档案信息（可编辑），右栏备注+录音历史+动态时间线（近10条）

## 关键设计决策
- 录音分析从独立 Tab 改为客户详情页右侧抽屉（2026-04-15）
- 抽屉关闭时清空状态，打开时按 prospectId 重新加载（支持跨客户并行分析）
- 同客户互斥：有 processing 记录时禁用提交
- 查看历史录音时上传面板整体灰色不可交互
- 关闭抽屉自动刷新详情页录音历史列表
- 管理员新建客户归属管理员自己
- 归属销售姓名运行时查用户服务，不改表

## Vuex 状态
- `isSalesOperation`: 是否为销售运营角色（mounted 时查询）

## How to apply
持续迭代中，后续沟通都围绕这个项目。注意前端是 Vue 2 语法（非 Vue 3），构建用 webpack。
