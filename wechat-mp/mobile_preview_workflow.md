---
name: 公众号文章手机端自测流程
description: 发草稿前用 Chrome headless + PIL 切片自查窄屏渲染，避免上次发布后手机端发现样式不兼容
type: project
originSessionId: acb4e9fe-cdc3-4b78-a378-98f5879e9531
---
## 自测脚本
`~/workspace/ai-workshop-covers/mobile_preview.py`

复用 publish.py 的 `md_to_html` 生成和草稿完全一致的 HTML，包裹进 iPhone 15 视口（390px），用 Chrome headless 截 390×9000 PNG。

## 用法
```
cd ~/workspace/ai-workshop-covers
python3 mobile_preview.py articles/00X-xxx.md
```

生成：
- `articles/00X-xxx_mobile.html`（可本地浏览器打开）
- `articles/00X-xxx_mobile.png`（长截图）

## 切片阅读
PNG 过长一次看不清细节，切 1600px 一段再 Read：
```python
from PIL import Image
img = Image.open('...mobile.png')
for i in range(n):
    img.crop((0, i*1600, 390, (i+1)*1600)).save(f'slice_{i+1}.png')
```

## 已知踩过的坑（都已在 publish.py 修掉）
1. **header `float:right`** 窄屏下右侧徽章被挤出 → 改单行居中 `text-align:center` + `span | span`
2. **长英文不换行** → 所有 p/li/blockquote 加 `word-wrap:break-word; overflow-wrap:break-word`
3. **strong 黄色高亮跨行断裂** → 加 `box-decoration-break:clone; -webkit-box-decoration-break:clone`
4. **table 列宽不受控溢出** → table 加 `table-layout:fixed`，td/th 加 `word-wrap:break-word`
5. **letter-spacing 中英混排折行算错** → 从 p 样式里删掉 `letter-spacing:0.4px`

**Why:** 2026-04-24 写 008 之前用户反馈"之前发布的手机上看样式不兼容"。直接发草稿用户需要打开手机后台验证，反馈链太长。
**How to apply:** 每篇推 publish.py 之前先跑 mobile_preview.py，切片 Read 检查 header / blockquote / table / 粗体 / 长英文引用，全部过了再推草稿。
