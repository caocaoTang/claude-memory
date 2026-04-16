---
name: GitHub memory sync
description: Claude 记忆文件自动同步到 GitHub 私有仓库 caocaoTang/claude-memory
type: reference
---

- 仓库：https://github.com/caocaoTang/claude-memory（私有）
- GitHub 账号：caocaoTang（邮箱 caocaotang@gmail.com）
- 同步脚本：~/.claude/sync-memory.sh
- Hook：PostToolUse → Write|Edit → async
- 机制：写入 memory/ 路径下文件后自动 commit + push

换设备恢复：
1. `gh repo clone caocaoTang/claude-memory /tmp/cm`
2. `cp -r /tmp/cm/* ~/.claude/projects/-Users-<username>/memory/`
3. 复制 sync-memory.sh + settings.json hook
