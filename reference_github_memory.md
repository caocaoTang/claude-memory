---
name: GitHub memory sync
description: Claude 记忆文件自动同步到 GitHub 私有仓库 caocaoTang/claude-memory
type: reference
---

记忆文件通过 PostToolUse hook 自动同步到 GitHub：
- 仓库：https://github.com/caocaoTang/claude-memory（私有）
- GitHub 账号：caocaoTang（邮箱 caocaotang@gmail.com）
- 同步脚本：~/.claude/sync-memory.sh
- Hook 配置：~/.claude/settings.json → PostToolUse → Write|Edit → async
- 同步机制：每次写入 memory/ 路径下的文件后，异步 commit + push

换设备恢复步骤：
1. gh repo clone caocaoTang/claude-memory /tmp/cm
2. cp /tmp/cm/* ~/.claude/projects/-Users-<username>/memory/
3. 复制 sync-memory.sh 和 settings.json hook 配置
