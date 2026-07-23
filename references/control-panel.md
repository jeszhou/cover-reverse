# 可调版控制面板（自助调参 → 复制参数回填）

**交付只出这一个"可调版 HTML"，先不出 PNG。** 左边封面实时预览，右边控制面板。用户自己拖滑块/改文字/拖位置/点背景换图，满意后点「复制参数」，把参数块（+本地图，如用了）粘回对话，我据此在**干净母版**上出**正式 PNG**——即参数确定后才出图。面板只是用户的调试台，不进成品。

（干净母版 = 不带面板的 HTML；出 PNG 时现从可调版剥离面板/拖拽脚本生成，或直接按参数新建，二选一。）

## 结构约定

- 页面 `body` 用 flex 横向排列：左 `.cover`（1086×1448，导出对象），右 `.panel`（`position:sticky`，不导出）。
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
      var availW=window.innerWidth-300-72, availH=window.innerHeight-48;   // 300=面板宽
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

## 控件覆盖（按封面实际有的元素生成，不要多不要少）

- **文字**：每个文本槽一个 `<input type=text>`（小标题/大字/副标题/tagline/日期等——有几个配几个）。
- **图片**：**点击换图（主推）** + 图片链接输入框（备选），见下方「点击换图」。
- **颜色**：每个 `--*-color` 变量一个 `<input type=color>` + 显示当前 hex。
- **数值**：每个字号/字距/位置类变量一个 `<input type=range>` + 显示"值+单位"。范围给合理区间（字号 120–360、字距 -12–20、位置 20–70% 等，按变量调整）。

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

## 复制参数按钮

点击后把所有控件当前值拼成一段可读文本，写进剪贴板（`navigator.clipboard.writeText`），并显示在一个只读 `<textarea>` 里兜底（剪贴板失败时可手动选中复制）。格式固定，方便回填时解析：

```
【封面参数 · <主题>】
小标题: ...
大字: ...
副标题: ...
tagline: ...
日期: ...
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
