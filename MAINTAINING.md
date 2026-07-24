# 维护地图（想改什么 → 去哪儿改）

这份文件回答一个问题：**"我想改 XX，该动哪个文件？"**
（记录"改了什么"是另一个文件 `CHANGELOG.md`，别混。）

## 一张表看懂每个文件管什么

| 文件 | 管的事 |
|---|---|
| `SKILL.md` | 总入口：四种玩法的主流程、导出与自查步骤、产物存哪 |
| `references/reverse-analysis.md` | 怎么读参考图：五步分析、版权切分（学什么/换什么）、先问平台与文案的规则、还原度预判 |
| `references/control-panel.md` | 可调版控制面板的一切：**参数决策三层法**、语言预设、控件怎么绑（JS helper）、拖拽、点击换图、下载 PNG、复制参数、回填出正式版 |
| `references/sizes.md` | 各平台画布尺寸预设表（小红书/公众号/B站/方图/视频…） |
| `references/fonts.md` | 中文字体映射表、字体降级链、中/英度量预设、字形陷阱、字体加载失败的回退 |
| `references/layout-tips.md` | CSS 变量规范、纸张噪点等质感的 CSS 写法、图片占位、画布骨架、缩略图可读性 |
| `README.md` | 对外介绍页、FAQ（给用户看的，不是给模型看的规则） |
| `CHANGELOG.md` | 每次改动的记录（改完往顶部加一段） |
| `examples/` | 效果对比图示例 |
| `docs/playground/` | GitHub Pages 在线试玩页 |

## 常见需求 → 直接定位

| 我想改… | 去这里 |
|---|---|
| 面板该出现哪些滑块/控件、参数太多或太少 | `control-panel.md` → 「控件覆盖 · 参数决策法」三层 |
| 新增一类元素的参数配方（比如二维码、价签） | `control-panel.md` → 第二层「元素类型配方表」加一行 |
| 中/英语言预设的行为（切换时改哪些数值） | `control-panel.md` 第一层 + `fonts.md`「中/英度量预设」 |
| 加一个新平台尺寸 / 改某个尺寸数值 | `sizes.md` 的预设表 |
| 换/加中文字体、字体没加载出来 | `fonts.md` 字体映射表 + 加载方式 + 回退规则 |
| 改"先问平台/文案"的提问规则 | `reverse-analysis.md` 第 0b 步 |
| 改"学什么、绝不搬什么"的版权规则 | `reverse-analysis.md` 第 0a 步 |
| 改导出 PNG 的工具顺序 / 自查项 | `SKILL.md`「导出与自查」 |
| 改拖拽、点击换图、复制参数的实现 | `control-panel.md` 对应小节（都有可照抄代码块） |
| 改纸张噪点/纹理等质感做法 | `layout-tips.md`「质感模拟」 |
| 改对外介绍话术、FAQ | `README.md` |

## 改完的收尾三步（每次都做）

1. **记一笔**：在 `CHANGELOG.md` 顶部加一段（日期 + 新增/变更了什么）。
2. **同步 + 校验**：本仓库已被 `.codex` / `.agents` / `.Claude` / `.claude` 四个 skill 目录以**符号链接**共用，改完即时对所有 Agent 生效，无需复制；可跑一次校验：
   `python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py ~/.codex/skills/cover-reverse`
3. **提交**：`git add -A && git commit && git push`；大改再打新 tag（`v1.1.0` / `v2.0.0`），小修可不打。
