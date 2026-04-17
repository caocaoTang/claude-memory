---
name: 系统级 pip.conf 被 null 字节污染
description: /Library/Application Support/pip/pip.conf 文件尾部有大量 null 字节污染，会让 subprocess 报 "embedded null byte"
type: project
originSessionId: 8d8a409b-cdd9-4d67-a01c-cff66003153f
---
用户 Mac 上的系统级 pip 配置 `/Library/Application Support/pip/pip.conf` 被 null 字节污染：文件实际 2MB，但正常配置只占头部约 140 字节（`[global]` + index-url / extra-index-url / trusted-host），剩下全是 `\x00`。pip 读取时会把 null 字节拼进 `trusted-host` 值，然后传给 `subprocess.Popen`，立刻触发 `ValueError: embedded null byte`。

**Why:** 2026-04-17 修 hermes 启动报错时发现——hermes 的 kqueue/stdin 报错其实是表象，真正卡住重建 venv 的是这个污染的 pip.conf 让 `pip install` 直接崩溃。文件 ls 时间戳 `6 30  2025`，ownership 是 `root:wheel`、权限 `-rwxrwxrwx`（rwx 给所有人，无需 sudo 即可改写）。

**How to apply:** 用户在 Mac 上用任何 Python 项目跑 `pip install` 时如果见到 `ValueError: embedded null byte`（尤其在 subprocess.Popen 里），先查 `/Library/Application Support/pip/pip.conf` 文件大小；正常应该 < 1KB。备份再用 `printf` 写回干净内容即可（不要用 `cp` 备份到同目录，目录是 `755 root:wheel`，备份到 `/tmp`）。干净内容示例：

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
extra-index-url = http://pypi.qima-inc.com/youzan/dev/+simple/
trusted-host = pypi.qima-inc.com
```

本次污染副本留在 `/tmp/pip.conf.bak.corrupted`。
