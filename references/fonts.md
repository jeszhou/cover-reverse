# 字体选择规则 + 中文字体映射表（云端可加载）

## 第一原则：按**目标文案**的语言选字体，不是按参考图的语言

参考图是英文海报不代表成品是英文。常见翻车：照着英文参考图配了 `Archivo Black`，
结果用户的文案是中文——该字体没有中文字形，中文会掉回系统默认黑体，
参考图那股粗壮压迫感全没了，而且**从 HTML 源码上看不出来，只有截图自查才发现**。

**能不问就不问**——用户给了文案时，语言自动就知道了，不要多问一轮。
只有在做"无文案的排版母版"时才需要处理语言不确定，用下面两层办法。

## 双语就绪：降级链 + 度量预设

**第一层·字体降级链（解决"字形缺失"）**
浏览器是**逐字形**回退的，所以把中文字体挂在英文字体后面，中英混排会各归各位：

    font-family: 'Archivo Black', 'ZCOOL QingKe HuangYou', sans-serif;   /* 粗黑大字 */
    font-family: 'Playfair Display', 'ZCOOL XiaoWei', 'Noto Serif SC', serif;  /* 衬线标题 */
    font-family: 'EB Garamond', 'Noto Serif SC', serif;                  /* 正文/说明 */

英文字母走英文字体，中文字自动落到中文字体。小红书封面极常见的"中文主标题 + 英文装饰元素"
（credits 条、日期、条形码）靠这一条就全解决了，**零提问**。

**第二层·中/英度量预设（解决降级链解决不了的"度量"）**
降级链只保证字形不缺，保证不了排版对。中文是**全角方块字**，必须另配一套数值：

| 差异 | 说明 |
|---|---|
| 字距 | 英文大标题常用 20–30px 大字距；中文本身就有字身留白，超过 ~8px 就散架 |
| 字数 | 英文 5 个字母 ≈ 中文 3–4 个字；同字号下中文宽得多，容易撑爆画布 |
| 纵向占位 | **中文方块字占满整个字身框，比英文大写字母压得更低**——下方元素要下移避让，否则重叠 |

做母版时把这两套数值写成 JS 预设 + 面板上一个「中文 / 英文」切换按钮，让用户自己按，
比提前问一句更准（用户要看到效果才知道对不对）。切换只改字号/字距/位置，**不要动用户已输入的文字**。

> ⚠️ 纵向避让这条必须**实测**：中文大字压低导致下方 tagline 重叠，靠推理想不到，只有截图才看得见。

## 中文字体映射表

目标：让封面字体尽量贴近参考图。中文字体无法从截图里 1:1 提取（好字体多为商业字体，且截图只有像素），所以策略是"从下表选最接近的一款，再用 CSS 微调把气质逼近"。字体**只从此表选取**，保证 Google Fonts 云端可加载、截图不缺字、HTML 到哪都能打开。

## 字体库（全部实测 Google Fonts 可加载）

| 参考图字体气质 | font-family 值 | 特征 |
|---|---|---|
| 现代无衬线/黑体 | `'Noto Sans SC', sans-serif` | 中性黑体，多字重 100–900 |
| 经典衬线/宋体 | `'Noto Serif SC', serif` | 标准宋体，多字重 200–900 |
| 文艺细宋/小标宋 | `'ZCOOL XiaoWei', serif` | 清秀文艺，笔画细，适合文艺标题 |
| 粗壮硬黑/海报体 | `'ZCOOL QingKe HuangYou', sans-serif` | 方正硬朗，冲击力强 |
| 圆润卡通/活泼体 | `'ZCOOL KuaiLe', cursive` | 圆头笔画，轻松可爱 |
| 楷体/温暖手写 | `'LXGW WenKai TC', 'KaiTi', serif` | 柔和楷体，亲和 |
| 潦草手写/钢笔字 | `'Zhi Mang Xing', cursive` | 连笔手写，随性 |
| 毛笔/书法标题 | `'Ma Shan Zheng', cursive` | 端正毛笔楷 |
| 狂草/写意书法 | `'Liu Jian Mao Cao', cursive` | 飞白狂草，写意 |
| 细手写/清瘦硬笔 | `'Long Cang', cursive` | 清瘦手写，秀气 |
| 英文 | `'Inter', sans-serif` 等标准 Google Fonts | 按英文气质另配 |

## 匹配方法（两步）

1. **选类**：先按上表的"气质"大类选出最接近的一款。
2. **比细节再微调**：对照参考图逐项比对下面四个维度，用 CSS 把选中的字体往参考图方向推——同一款字能调出好几种气质，这一步决定像不像。

| 参考图特征 | 对应 CSS 调整 |
|---|---|
| 字更粗/更重 | 优先用该字体的高字重（`font-weight:700/900`，见下方字重加载）；无更高字重时用 `-webkit-text-stroke: 1px currentColor` 加描边增粗 |
| 字更细/更轻 | 用低字重（`font-weight:200/300`） |
| 字更宽/更扁 | `transform: scaleX(1.1)`（配 `display:inline-block`）或加 `letter-spacing` |
| 字更窄/更长 | `transform: scaleY(1.1)` 或 `transform: scaleX(0.9)` |
| 字距更疏 | `letter-spacing: 4px~12px`（大标题常见） |
| 字距更紧 | `letter-spacing: -2px` |
| 笔画更硬/更方 | 优先选硬黑类（ZCOOL QingKe HuangYou） |
| 笔画更圆/更软 | 优先选圆润/楷体类（ZCOOL KuaiLe、LXGW WenKai） |

把微调值也写成 CSS 变量（如 `--title-weight`、`--title-tracking`），方便用户后续再调。

## 加载方式

HTML `<head>` 中按需引入——只引用到的字体，且**带上要用到的字重**（`:wght@...`），否则 CSS 里写了 `font-weight:900` 但没加载该字重会退回默认粗细：

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@300;400;700;900&family=LXGW+WenKai+TC&display=swap" rel="stylesheet">

注意：`ZCOOL`/`Ma Shan Zheng`/`Zhi Mang Xing`/`Liu Jian Mao Cao`/`Long Cang`/`ZCOOL KuaiLe` 等只有单一字重（400），需要更粗时用 `-webkit-text-stroke` 描边，别在 URL 里写不存在的 `wght`。

## 字形陷阱自查

截图自查时除字体加载外，还要逐字检查**易混字形**：部分展示字体的个别字写法特殊，
小字号下会被读成别的字（实例：ZCOOL XiaoWei 的"己"竖笔冒过顶横，成品里被读作"已"）。
发现易混字时二选一：换用该字形清晰的字体（如 Noto Sans/Serif SC），或与用户商量微调文案避开该字。

## 回退规则

截图自查发现字体未加载（文字显示为默认黑体、微调没生效）时：检查 family 拼写与 `wght` 是否真实存在 → 仍失败则回退到表内最接近的系统字体（如 KaiTi），并在交付时告知用户匹配到了哪一款、做了哪些微调、与参考图还有什么差距。
