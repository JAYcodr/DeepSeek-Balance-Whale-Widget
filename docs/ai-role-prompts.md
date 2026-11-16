# 用生图做一只新的小鲸鱼

角色就是一张透明底 PNG。生图模型**不能直接出透明底**，所以流程是：
**纯色幕布出图 → 抠掉幕布 → 拖进挂件**。全程两条命令。

## 1. 出图（提示词可直接抄）

要点只有三条：

- **Q 版半身 / 全身贴纸**，不要长焦、不要场景；
- **纯平涂幕布背景**，颜色必须是角色身上**没有**的颜色（蓝白角色用绿幕 `#00b140`，暖色角色用蓝幕 `#2E7BE4`）；
- 写死「**居中留边距，画面无文字无水印**」。

### 同角色换表情（有参考图时）

拿现有角色图当参考图（`--ref` / 工具的 `reference_images`），只改表情段：

```
参考图是蓝发鲸鱼娘角色。用同一角色、同一画风画一张 Q 版半身立绘：
{表情：开心地笑，眼睛弯成月牙、脸颊泛红、一只手比耶 / 困倦，半睁眼打哈欠，头侧冒一滴小汗珠 / 惊讶，眼睛放大、嘴巴张成小 o、呆毛竖起}；
纯平涂亮绿色背景 #00b140，无渐变无阴影，角色居中留边距，画面无文字无水印。
```

### 全新角色（无参考图时）

```
Q 版圆滚滚小鲸鱼吉祥物：深蓝与奶白配色，肚皮奶白，短鳍，眼睛大而圆，歪着头坐着，头顶一个小水花；
厚涂二次元贴纸风格，纯平涂亮绿色背景 #00b140，无渐变无阴影，居中留边距，画面无文字无水印。
```

一次出 3–4 张挑一张；表情组建议一次给 3 条不同表情的提示词（同一张参考图），风格才不跑。

## 2. 抠幕布（一行 Python）

```bash
python3 - <<'PY'
from PIL import Image, ImageFilter
import sys
from collections import Counter

def border_color(im):
    w, h = im.size; px = im.load(); c = Counter()
    for x in range(0, w, 3):
        for y in (0, 1, 2, h-3, h-2, h-1): c[px[x, y][:3]] += 1
    for y in range(0, h, 3):
        for x in (0, 1, 2, w-3, w-2, w-1): c[px[x, y][:3]] += 1
    return c.most_common(1)[0][0]

def key(src, dst, tol_low=40, tol_high=120, erode=2, pad=1024, margin=0.04):
    im = Image.open(src).convert("RGBA"); W, H = im.size
    bg = border_color(im); px = im.load()
    alpha = Image.new("L", (W, H), 255); ap = alpha.load()
    for y in range(H):
        for x in range(W):
            r, g, b, _ = px[x, y]
            d = abs(r-bg[0]) + abs(g-bg[1]) + abs(b-bg[2])
            ap[x, y] = 0 if d <= tol_low else (255 if d >= tol_high else int(255*(d-tol_low)/(tol_high-tol_low)))
    alpha = alpha.filter(ImageFilter.MinFilter(erode*2+1)).filter(ImageFilter.GaussianBlur(0.8))
    im.putalpha(alpha)
    bb = im.getbbox()
    if bb: im = im.crop(bb)
    inner = int(pad*(1-margin*2)); w, h = im.size; s = inner/max(w, h)
    im = im.resize((max(1, int(w*s)), max(1, int(h*s))), Image.LANCZOS)
    canvas = Image.new("RGBA", (pad, pad), (0, 0, 0, 0))
    canvas.alpha_composite(im, ((pad-im.width)//2, pad-im.height))
    canvas.save(dst)
    print(dst, "bg=", bg, "->", canvas.size)

key(sys.argv[1] if len(sys.argv) > 1 else "raw.png", "role.png")
PY
```

- `tol_low/tol_high`：幕布不干净（有渐变、有阴影）时调大；边缘吃掉了角色就调小。
- 输出固定 1024×1024 透明底、角色贴底居中 —— 和挂件的落点一致。
- 角色身上有幕布同色时，抠完会有洞：换一个幕布颜色重出图，比后期补更快。

## 3. 装进去

把 `role.png` **拖到小鲸鱼身上**（或 ⌘/Ctrl+V 粘贴），或走菜单「角色 → 导入」。
导入后自动出现在角色列表里并切换过去；动图会保留帧（GIF/APNG 不裁剪）。

## 4. 想做表情组？

同一角色的 N 张表情图分别导入，命名为「小鲸鱼·开心 / 困 / 惊讶」。
挂件目前**不会**自动按状态换表情（这一层还没做，见 PR 里的动效提案：
额度高→开心、快用完→发愁、久置→睡着）。
