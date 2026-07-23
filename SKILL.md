---
name: cover-reverse
description: Reverse-engineer a reference cover image into an editable, self-contained HTML master and export a platform-sized PNG (Xiaohongshu 3:4 by default). Use when the user provides a reference image and asks to 照这张图做封面, 临摹这个封面/排版, 逆向还原这个设计, clone this cover style, or asks to recolor an existing cover with another image's palette (配色换肤). Learns only the design language (layout, palette, type) — never copies the original's text, signatures, photos, or illustrations.
---

# Cover Reverse（封面临摹）

看到好看的封面，临摹下来变成你自己的可编辑母版——学的是设计语法，换的是你的内容。

参考图逆向还原封面。三种玩法，产物都是"PNG + 可编辑 HTML 母版"。

## 玩法一：逆向还原新封面

输入：参考图（必须）+ 封面文案。缺文案时先问一句再动手。

1. 读参考图，按 [references/reverse-analysis.md](references/reverse-analysis.md) 完成五步分析
2. 生成自包含 HTML：
   - 画布尺寸按 [references/sizes.md](references/sizes.md) 选定
   - 骨架与 CSS 变量规范见 [references/layout-tips.md](references/layout-tips.md)
   - 中文字体只从 [references/fonts.md](references/fonts.md) 映射表选
   - 照片/插画元素用 `<img>` + 占位图；可调版支持点击换本地图（见"玩法四"）
3. **交付只出一个 `主题_小红书封面_可调版.html`（带控制面板 + 拖拽 + 点击换图），先不出 PNG。** 列出可调整项，告诉用户怎么用（改文字/拖位置/点背景换图 → 复制参数发我）。
4. **等用户参数确定**（把复制的参数 + 本地图发回）**再出正式 PNG**：按参数回填到干净母版 → 导出（见"导出与自查"），交付 `主题_小红书封面_v1.png` + 干净母版 HTML。

## 玩法二：对话修改

用户说"标题换楷体""背景换米色""标题往下挪"之类：

1. 找到最新版本的 HTML 母版
2. 只改对应的 CSS 变量或布局；换字体时同步更新 Google Fonts link
3. 另存为 v(n+1)，重新导出 PNG。旧版本永不删除

## 玩法三：配色换肤

用户给另一张图说"用这张图的配色改封面"：

1. 从新图提取一套和谐色板（背景、主文字、强调色……与现有封面的颜色变量一一对应）
2. 只替换 `:root` 里的颜色变量，版式、字体、字号一律不动
3. 另存为 v(n+1)，重新导出 PNG

## 玩法四：可调版（自助调参 + 拖拽 → 复制参数回填）

可调版 HTML 是**玩法一的唯一交付物**：左边实时预览，右边控制面板，用户自己改文字/颜色/字号、拖动元素定位、**点封面背景/图片换本地图**，满意后点「复制参数」，把参数块（+本地图）粘回对话；我据此在**干净母版**上出正式 PNG（面板不进成品，参数确定后才出图）。构建模式、拖拽、点击换图、复制参数格式、回填步骤见 [references/control-panel.md](references/control-panel.md)。

用户粘参数回来时 = 走此文件的"回填出正式版"：把参数覆盖到干净母版 → 另存 v(n+1) → 按下方导出。

## 导出与自查（按序探测，装了哪个用哪个）

以下 `W×H` 为所选画布尺寸。依次探测三种工具，探测方式：命令 `--version`/`which` 是否可用。

**1. agent-browser（首选）**——顺序固定，先 open 再 set viewport：

    agent-browser --session cover --allow-file-access open "file://<html绝对路径>"
    agent-browser --session cover set viewport W H
    agent-browser --session cover wait --load networkidle
    agent-browser --session cover wait 1800
    agent-browser --session cover screenshot "<png绝对路径>"
    agent-browser --session cover close

**2. Playwright**：

    npx -y playwright screenshot --viewport-size "W,H" --wait-for-timeout 2500 "file://<html绝对路径>" "<png绝对路径>"

（若报浏览器未安装，先 `npx playwright install chromium` 或降到方案 3。）

**3. 系统 Chrome 无头**（macOS 路径如下，Linux 用 `google-chrome`）：

    "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu \
      --hide-scrollbars --window-size=W,H --virtual-time-budget=5000 \
      --screenshot="<png绝对路径>" "file://<html绝对路径>"

**都没有**：告诉用户直接用可调版面板里的「下载 PNG」按钮（浏览器内导出），并说明精细效果以正式导出为准。

**自查（每次导出后必做）**：
- `sips -g pixelWidth -g pixelHeight <png>`（或 PIL）确认输出等于 W×H，不等说明 viewport 没生效，重截。
- 用 Read 看截图：中文字体是否真加载（未加载会退成默认黑体）、布局有无溢出破损、
  主标题在缩略图尺寸下是否可读（阈值见 sizes.md）。有问题先修再交付。
- 字体失败按 [references/fonts.md](references/fonts.md) 回退规则处理。文件名含中文用引号包路径。

## 输出位置

跟随用户当前工作目录/用户指定的项目文件夹；没有明确位置时问一句存哪里。
