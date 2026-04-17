---
name: crm-master 部署流程
description: crm-master 应用前后端一体部署的关键约束：JS 必须保留 2 版防滚动发布 404；merge 分支命名与检测方式；专用 skill 路径。
type: project
originSessionId: 72735785-37c3-4b1c-bae7-8de773bc6636
---
# crm-master 部署关键约束

## 应用基本信息
- 仓库：`git@gitlab.qima-inc.com:enable-platform/crm-master.git`
- 本地路径：`/Users/tangcaocao/workspace/crm-master`
- 类型：JAVA（前后端一体）
- 前端：Vue2 + webpack2，产物输出到 `crm-master-web/src/main/resources/static/crm-master/js/`，跟随 jar 一起打包
- 模板：FreeMarker `index.ftl / wecom.ftl / fill.ftl` 读 `manifest.json` 注入 hash 化脚本

## JS 双版本保留（避免滚动发布瞬间 404）

**Why**：线上多机滚动发布时，老版本 HTML 已缓存在浏览器，引用旧 hash JS；请求被 LB 路由到已升级机器会 404。

**How to apply**：
- `crm-master-front/scripts/clean-old-assets.js` + `package.json` build 脚本：build 前读 `manifest.json`，保留上版文件，webpack 产出新版 → 目录常驻 2 版
- 不要改回 `rm -rf ../crm-master-web/.../js/*`
- `app.css` 目前未 contenthash 化，CSS 在发版瞬间仍会覆盖导致样式错乱（已告知用户，暂不处理）

## Merge 分支约定
- 命名：`merge_pre/<8位随机>`（如 `merge_pre/m7epudbg`），每次 Hera 创建一个新的
- 查当前预发在用的 merge 分支：OPS pods API `branch` 字段
  - `https://idea-plugin-portal.prod.qima-inc.com/api/v1.0/ops/application/pods?app_name=crm-master&buId=1&bu_id=1&namespace=pre&standard_env=pre`
  - 需要 `Authorization: Bearer <OPS_JWT_TOKEN>`
  - 返回 `branch` 为 `master` 说明当前预发无集成分支

## 专用 skill
- `crm-master-pre-deploy` — 自动把当前开发分支加入 pre 集成分支，处理 JS 冲突（保留 master 为上一版），rebuild 前端，推送 merge 分支并切回
- 脚本：`~/.claude/skills/crm-master-pre-deploy/scripts/deploy-pre.sh`
- 关键设计：
  - 开始先 `git fetch origin master` + 本地 `master` 快进到 `origin/master`（被其他 worktree 占用则仅更新 remote ref）
  - opscli add-branch 前对远端 `merge_pre/*` 打 snapshot，调用后 diff 识别新建分支，避免错选别人的旧 merge 分支
  - **Case B（复用已有 merge_pre）先 presync 再 opscli**：checkout merge_pre → merge origin/master → JS 冲突取 master 版本 → rebuild + push；让 OPS 服务端 merge(master→merge_pre) 变 fast-forward，避免 opscli 因服务端合并冲突失败（2026-04-17 salesbuddy 部署踩过的坑）

## opscli 认证
- 用户使用 `OPS_JWT_TOKEN` 环境变量，没配 `~/.ops-token` 文件
