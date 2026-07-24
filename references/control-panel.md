# 可调版控制面板（自助调参 → 复制参数回填）

**交付只出这一个"可调版 HTML"，先不出 PNG。** 左边封面实时预览，右边控制面板。用户自己拖滑块/改文字/拖位置/点背景换图，满意后点「复制参数」，把参数块（+本地图，如用了）粘回对话，我据此在**干净母版**上出**正式 PNG**——即参数确定后才出图。面板只是用户的调试台，不进成品。

（干净母版 = 不带面板的 HTML；出 PNG 时现从可调版剥离面板/拖拽脚本生成，或直接按参数新建，二选一。）

## 结构约定

- 页面 `body` 用 flex 横向排列：左 `.cover`（1086×1448，导出对象），右 `.panel`（`position:sticky`，宽度 320px，不导出）。
- **`.cover` 必须包一层 `.coverwrap` 并自动缩放适配屏幕**（1086×1448 原尺寸会占满/溢出屏幕，很难操作）。缩放只影响预览，导出仍按全尺寸干净母版出图，清晰度不受影响。照抄：

  ```html
  <div class="coverwrap" id="coverwrap"><div class="cover" id="cover"> …封面… </div></div>
  ```
  ```css
  .coverwrap{flex:0 0 auto}
  .cover{width:1086px;height:1448px;transform-origin:top left; /* 其余不变 */}
  ```
  ```html
  <script>
    function fit(){
      var availW=window.innerWidth-320-72, availH=window.innerHeight-48;   // 320=面板宽
      var s=Math.min(availW/1086, availH/1448, 1); s=Math.max(s,0.2);
      var cover=document.getElementById('cover'), wrap=document.getElementById('coverwrap');
      cover.style.transform='scale('+s+')'; wrap.style.width=(1086*s)+'px'; wrap.style.height=(1448*s)+'px';
    }
    window.addEventListener('resize',fit); fit();
  </script>
  ```
  缩放与拖拽兼容：拖拽的 `scaleOf()` 读 `getBoundingClientRect().width/offsetWidth` 正好等于当前缩放，屏幕位移除以它即得封面内位移，实测准确。
- `.panel` 独立于 `.cover`，所以永远不会出现在封面里；正式版由我用参数在干净母版上重出，不从这个文件截图。
- 面板背景用深色（`#1e1e1e`）区别于封面，避免误认。

## 控件覆盖 · 参数决策法（先决定"配哪些控件"，再按下方 helper 绑定）

面板参数**数量随图而变是正常的**——元素多则参数多，元素少则参数少。但"变"必须有规律、可预测，不能每张图随缘（这是不同 Agent/不同次生成差异过大的根源）。按下面**三层**决策，保证跨图、跨次、跨 Agent 收敛一致。

### 第一层 · 结构必有（不论什么图，以下永远都在）

- **文案**：每个文本槽一个 `<input type=text>`（有几段文字配几个）；含固定的「我的署名」框（见下）。
- **图片**：每张图都给**换图能力**——点击换图（主推）+ 图片链接输入框（备选），见下方「点击换图」。
- **颜色**：每个 `--*-color` 变量一个 `<input type=color>` + 显示当前 hex。
- **语言/排版预设**：面板固定给「中文预设 / 英文预设」两个切换按钮，**不论用户给的是中文还是英文文案都要有**。切中文时：切到中文字体降级链，并因中文是全角方块字、纵向占位比英文低而**同步放松行高与顶距**；切英文时反之。目的是"换语言不散版"。（这一条以前是条件功能，现固定为必有，避免不同 Agent 有时做有时不做。）
- **出图按钮**：暂停拖拽 / 重置所有位置 / 复制参数 / 下载 PNG，四个固定都要有。
- **画布尺寸**：按**用户选定平台**的尺寸（sizes.md，动手前先问平台、不默认小红书）固定使用，**不做滑块**。

### 第二层 · 逐元素配方（核心：不列"元素"，列"元素类型"）

先把图里识别到的**每个元素**归到下面某一类（承接 reverse-analysis.md 第 3 步的元素清点），再按该类"配方"挂上数值/位置控件。好处：遇到清单里没写过的新元素（二维码、邮票齿边、价格标签……）也能先归类再套配方，**不会漏、不会各凭发挥**。

| 元素类型 | 典型 | 配方（给它挂这些控件） |
|---|---|---|
| **文字类** | 大字 / 小标题 / 副标题 / tagline / 日期 / 竖排小字 | 内容框 + 字号 + 字距 +（大标题或中文再加**行高**）+ 颜色 + 位置(拖拽) |
| **图片类** | 封面照片 / logo | 换图 + 高度或尺寸 +（有圆角则加圆角）+ 间距 + 位置 |
| **线条/形状类** | 装饰横线 / 分割线 / 描边框 | 粗细 + 长度 + 颜色 + 位置 |
| **图标/标记类** | 树形标记 / 印章 / 二维码 | 宽 + 高 + 位置（+ 若压在别的元素上，加"压住量"） |
| **质感/背景类** | 等高线 / 纸张颗粒 / 四边压暗 | 浓度或强度 + 颜色 |

**配方规则**：
- 图里有几个元素，就逐个套配方；**没有的类型不配**（没有竖排就别给"竖排字号"滑块）。
- 同类的多个元素各配一份（三张照片 = 三组图片控件；可共用"高度/间距"、各自"位置"，按版式定）。
- 控件类型对应下方 JS helper：内容→`bindText`、颜色→`bindColor`、字号/字距/行高/位置等数值→`bindRange`。

**署名**（第一层里的固定文本框，细节）：面板固定有「我的署名」文本框（id `cSign`），绑定封面上的署名元素（id `elSign`，可拖拽）。位置参考原图署名位——版权切分抹掉了别人的署名，这里放用户自己的。用户清空输入框则隐藏署名元素（bindText 基础上加一行 `e.style.display=i.value?'':'none'`）。

### 数值范围参考

字号 120–360、字距 -12–20、行高 1.0–1.6、位置 20–70%、线条粗细 1–12px、质感浓度 0–1……按变量实际调整，区间要让人"拖得动、看得出变化"。

### 交付前自查（生成面板后逐条核对）

- **第一层是否全在？** 每个颜色变量都有取色器？署名框 + 四个出图按钮都在？画布尺寸没被做成滑块？
- **第二层是否和图里元素一一对上？** 图里有的元素却没配控件 = 漏；配了图里根本没有的控件 = 多。
- 判据不是"参数够不够多"，而是**"每个元素都被它该有的控件覆盖到、且没有多余控件"**。

## JS 绑定（三个 helper，照抄即可）

```html
<script>
  var root=document.documentElement;
  // 文本：输入框 → 元素 textContent
  function bindText(inputId,elId){var i=document.getElementById(inputId),e=document.getElementById(elId);
    i.addEventListener('input',function(){e.textContent=i.value});}
  // 颜色：取色器 → CSS 变量 + 显示 hex
  function bindColor(inputId,varName,valId){var i=document.getElementById(inputId),v=document.getElementById(valId);
    i.addEventListener('input',function(){root.style.setProperty(varName,i.value);v.textContent=i.value;});}
  // 数值：滑块 → CSS 变量(带单位) + 显示
  function bindRange(inputId,varName,valId,unit){var i=document.getElementById(inputId),v=document.getElementById(valId);
    i.addEventListener('input',function(){root.style.setProperty(varName,i.value+unit);v.textContent=i.value+unit;});}
  // 图片：输入框 → img.src   （直接 addEventListener 即可）
</script>
```

为每个控件调用对应 helper 绑定（`bindText('cKicker','elKicker')` 等）。

## 拖拽定位（元素可鼠标拖动）

给每个可移动元素加 `.drag` class + `data-name`，用 pointer 事件让用户直接把元素拖到想要的位置；偏移存在 `dataset.dx/dy`，一并写进复制参数。照抄：

```html
<style>
  .drag{cursor:move}
  .drag.dragging{outline:2px dashed rgba(232,115,74,.9);outline-offset:4px}
  body.dragmode .drag:hover{outline:1px dashed rgba(232,115,74,.5);outline-offset:4px}
</style>
<script>
  var scaleOf=function(){var c=document.getElementById('cover');return c.getBoundingClientRect().width/c.offsetWidth||1;};
  document.querySelectorAll('.cover .drag').forEach(function(el){
    el.dataset.dx=0;el.dataset.dy=0;
    el.addEventListener('pointerdown',function(e){
      if(!document.body.classList.contains('dragmode'))return;
      e.preventDefault();var s=scaleOf();
      var sx=e.clientX,sy=e.clientY,bx=parseFloat(el.dataset.dx),by=parseFloat(el.dataset.dy);
      el.classList.add('dragging');el.setPointerCapture(e.pointerId);
      function mv(ev){var nx=bx+(ev.clientX-sx)/s,ny=by+(ev.clientY-sy)/s;
        el.dataset.dx=nx;el.dataset.dy=ny;el.style.transform='translate('+nx+'px,'+ny+'px)';}
      function up(){el.classList.remove('dragging');
        el.removeEventListener('pointermove',mv);el.removeEventListener('pointerup',up);}
      el.addEventListener('pointermove',mv);el.addEventListener('pointerup',up);
    });
  });
</script>
```

- `body` 加 `dragmode` class 作为总开关；面板放「暂停拖拽」「重置所有位置」两个按钮（toggle class / 清 `dataset.dx,dy` + `style.transform=''`）。
- 用 `transform: translate` 做偏移（不动原布局，只叠加位移），0,0 = 原位。
- `scaleOf()` 除以缩放，保证封面被缩放预览时拖动距离仍对得上。
- 可拖元素通常是：大字、小标题、副标题、顶部 credits、底部块——即每个有独立视觉位置的文本组，各给一个 `id` 以便复制参数里输出偏移。

## 点击换图（选本地图替换背景/占位图）

让用户直接点封面里的图片区域，弹出文件选择器选本地图，`FileReader` 读成 data URL 实时预览。照抄：

```html
<input type="file" id="fileInput" accept="image/*" style="display:none">
<!-- 图片容器加 cursor:pointer 和 id，如 <div class="bg" id="bgArea"> -->
<script>
  var fileInput=document.getElementById('fileInput');
  document.getElementById('bgArea').addEventListener('click',function(){fileInput.click();});
  fileInput.addEventListener('change',function(){
    var f=this.files&&this.files[0];if(!f)return;
    var rd=new FileReader();
    rd.onload=function(){document.getElementById('photoImg').src=rd.result;
      window.__localImg=f.name;/* 面板里显示已选文件名 */};
    rd.readAsDataURL(f);
  });
  // URL 输入框改动时清掉本地图标记：this.oninput → window.__localImg=''
</script>
```

- 点击区域用图片容器（背景类封面点整块背景；前景图点该 `<img>`）。前景图上若压着可拖文本，文本 z-index 更高、各自处理，点空白图区换图、点文本拖动，互不冲突。
- **本地图只存在于用户的浏览器预览里，不在磁盘 HTML 文件中**（data URL 未落盘）。所以我要出正式 PNG 时**必须拿到用户那张原图**——复制参数里对本地图输出提示："图片: 本地图「<文件名>」——请把这张图一起发我"；用户把图发进对话或放进封面文件夹，我再 bake。
- 若用的是图片 URL（非本地图），参数里直接给 URL，我能自取，无需额外发图。

## 下载 PNG 按钮（浏览器内快速导出，零依赖兜底）

面板须有「下载 PNG」按钮：用户不装任何工具也能出图。按钮旁注明小字"快速导出，精细效果以正式导出为准"。照抄（W/H 换成画布尺寸）：

```html
<button id="btnDownload">下载 PNG</button> <small>快速导出，精细效果以正式导出为准</small>
<script src="https://cdn.jsdelivr.net/npm/html-to-image@1.11.13/dist/html-to-image.min.js"></script>
<script>
  document.getElementById('btnDownload').addEventListener('click', function(){
    var cover=document.getElementById('cover'), old=cover.style.transform;
    cover.style.transform='none';               // 临时取消预览缩放，按全尺寸导出
    htmlToImage.toPng(cover,{width:W,height:H,pixelRatio:1})
      .then(function(url){ cover.style.transform=old;
        var a=document.createElement('a'); a.download='封面_快速导出.png'; a.href=url; a.click(); })
      .catch(function(e){ cover.style.transform=old; alert('导出失败：'+e+'（可改用复制参数流程出正式图）'); });
  });
</script>
```

- 需要联网加载 CDN 与字体；导出失败（跨域图等）时引导走复制参数流程。
- 正式 PNG 仍以干净母版 + 导出降级链为准，此按钮只是自助兜底。

## 复制参数按钮

点击后把所有控件当前值拼成一段可读文本，写进剪贴板（`navigator.clipboard.writeText`），并显示在一个只读 `<textarea>` 里兜底（剪贴板失败时可手动选中复制）。格式固定，方便回填时解析：

```
【封面参数 · <主题>】
小标题: ...
大字: ...
副标题: ...
tagline: ...
日期: ...
署名: ...
图片URL: ...
背景色: #......
文字色: #......
大字色: #......
大字字号: ...px
大字字距: ...px
小标题字号: ...px
小标题字距: ...px
照片起始: ...%
—— 位置偏移（0,0=原位）——
大字位置: x=..px y=..px
小标题位置: x=..px y=..px
副标题位置: x=..px y=..px
署名位置: x=..px y=..px
...每个可拖元素一行
```

字段随封面实际变量/元素增减；每行 `键: 值`，键名用中文，和面板 label 对应。位置偏移用 `dataset.dx/dy`（四舍五入取整）。

## 回填出正式版（用户粘参数回来时 = 此刻才出 PNG）

1. 逐行读参数，覆盖到一份**干净母版**（不带面板/拖拽脚本的 HTML）的对应文本/`:root` 变量/`img src`/元素 `transform` 位置偏移。
2. 若参数里是"本地图「名字」"，需要用户发来的那张原图（放进封面同目录，`img src` 用相对路径）；若是 URL 直接用。
3. 另存为 v(n+1)，按 SKILL.md「导出与自查」出 PNG。
4. 不从可调版 HTML 截图（面板会进画面）——始终用干净母版出图。

## 实测注意

- 面板 HTML 是给用户在自己浏览器里手动用的，**不需要** agent-browser 的 `set viewport`；那条只用于我导出成品。
- 若要用 agent-browser 预览面板版，`open` 后**不要**紧跟 `set viewport` 再 eval（实测会把页面弄空白）——预览截图用独立 session、加 `wait` 后再截。
