# 自由排版与质感技巧

## CSS 变量化规范（必须遵守）

所有主要颜色和尺寸定义在 `:root`，正文样式只引用变量——这是"一句话改色/改字号"的基础：

    :root {
      --cover-font: 'Noto Serif SC', serif; /* 全局字体，一处改全局换 */
      --bg-color: #1a1a1a;      /* 背景颜色 */
      --title-color: #ffffff;   /* 标题颜色 */
      --accent-color: #e8734a;  /* 强调色 */
      --title-size: 64px;       /* 标题字号 */
      --card-radius: 16px;      /* 卡片圆角 */
    }

- 每个变量后写中文注释（充当变量清单，方便后续对话修改）
- 封面内文本一律 `font-family: var(--cover-font)` 或 `inherit`，不硬编码字体名

## 质感模拟

- **纸张噪点**（最常用）：SVG feTurbulence 滤镜 data-uri 作为叠加背景：
  `url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.08'/%3E%3C/svg%3E")`
- **牛皮纸/揉皱纸/磨砂玻璃/反光**：多重径向、线性渐变叠加 + box-shadow
- **摄影级纹理**（树影、水波、真实风景——CSS 无法模拟）：用高质量占位图
  `background-image: url('https://picsum.photos/seed/shadow/800/1200')`，不要退化成纯色，这对还原设计的灵魂至关重要

## 图片占位与替换

- 照片、插画、人物一律 `<img style="object-fit:cover">` + picsum 占位，**不要**用 background-image（用户之后给图时直接换 src 即可）
- 仅全屏纯装饰纹理底图才用 background-image
- 用户提供替换图时：拷贝到封面同目录，src 用相对路径

## 3:4 画布骨架

    <body style="margin:0">
      <div class="cover" style="width:1086px;height:1448px;overflow:hidden;position:relative">
        <!-- 封面内容 -->
      </div>
    </body>

## 缩略图可读性

小红书 feed 里封面很小。导出后自查：主标题在 200px 宽的缩略图下是否仍然可读？不行就加大字号/对比度。
