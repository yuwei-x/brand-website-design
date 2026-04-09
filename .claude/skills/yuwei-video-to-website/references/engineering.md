# 工程基础参考

## Lenis + ScrollTrigger 连接

每次都使用这段样板代码，不需要修改：

```javascript
const lenis = new Lenis({
  duration: 1.2,
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
  smoothWheel: true
});
lenis.on("scroll", ScrollTrigger.update);
gsap.ticker.add((time) => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);
```

## CDN 引用（必须按此顺序）

```html
<script src="https://cdn.jsdelivr.net/npm/lenis@1/dist/lenis.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/ScrollTrigger.min.js"></script>
<script src="js/app.js"></script>
```

## Canvas 帧渲染完整实现

### Padded Cover 绘制

```javascript
const IMAGE_SCALE = 0.85; // 0.82-0.90 sweet spot
function drawFrame(index) {
  const img = frames[index];
  if (!img) return;
  const dpr = window.devicePixelRatio || 1;
  const cw = canvas.width / dpr;
  const ch = canvas.height / dpr;
  const iw = img.naturalWidth, ih = img.naturalHeight;
  const scale = Math.max(cw / iw, ch / ih) * IMAGE_SCALE;
  const dw = iw * scale, dh = ih * scale;
  const dx = (cw - dw) / 2, dy = (ch - dh) / 2;
  ctx.fillStyle = bgColor; // 采样自帧角落
  ctx.fillRect(0, 0, cw, ch);
  ctx.drawImage(img, dx, dy, dw, dh);
}
```

为什么用 padded cover 而非纯 cover 或 contain：
- 纯 cover (`Math.max` at 1.0)：产品会被裁切到 header 区域
- 纯 contain (`Math.min`)：canvas 边缘露出不匹配的背景色
- Padded cover 在两者之间取平衡，`IMAGE_SCALE` 控制留白比例

### 背景色采样

从帧四角取平均色，填充 canvas 边缘，让产品和背景无缝融合：

```javascript
function sampleBgColor(img) {
  const tmp = document.createElement("canvas");
  tmp.width = img.naturalWidth;
  tmp.height = img.naturalHeight;
  const c = tmp.getContext("2d");
  c.drawImage(img, 0, 0);
  const corners = [
    [2, 2], [img.naturalWidth - 3, 2],
    [2, img.naturalHeight - 3], [img.naturalWidth - 3, img.naturalHeight - 3],
  ];
  let r = 0, g = 0, b = 0;
  corners.forEach(([x, y]) => {
    const d = c.getImageData(x, y, 1, 1).data;
    r += d[0]; g += d[1]; b += d[2];
  });
  return `rgb(${Math.round(r/4)},${Math.round(g/4)},${Math.round(b/4)})`;
}
```

调用频率：每 ~20 帧采样一次即可，不需要每帧都算。

### 两阶段帧加载

```javascript
async function loadAllFrames() {
  let loaded = 0;
  // Phase 1: 先加载前 10 帧，实现快速首屏
  for (let i = 0; i < Math.min(10, FRAME_COUNT); i++) {
    await loadFrame(i);
    loaded++;
    updateLoader(loaded);
  }
  if (frames[0]) drawFrame(0);
  // Phase 2: 后台并行加载剩余帧
  const remaining = [];
  for (let i = 10; i < FRAME_COUNT; i++) {
    remaining.push(loadFrame(i).then(() => { loaded++; updateLoader(loaded); }));
  }
  await Promise.all(remaining);
  hideLoader();
}
```

### Canvas Retina 缩放

```javascript
function resizeCanvas() {
  const dpr = window.devicePixelRatio || 1;
  canvas.width = window.innerWidth * dpr;
  canvas.height = window.innerHeight * dpr;
  canvas.style.width = window.innerWidth + "px";
  canvas.style.height = window.innerHeight + "px";
  ctx.scale(dpr, dpr);
}
```

不做这一步，canvas 在 Retina 屏上会模糊到无法使用。

## 帧播放绑定

### 线性播放（标准）

```javascript
const FRAME_SPEED = 2.0; // 1.8-2.2，控制产品动画完成时间点
ScrollTrigger.create({
  trigger: scrollContainer,
  start: "top top",
  end: "bottom bottom",
  scrub: true,
  onUpdate: (self) => {
    const accelerated = Math.min(self.progress * FRAME_SPEED, 1);
    const index = Math.min(Math.floor(accelerated * FRAME_COUNT), FRAME_COUNT - 1);
    if (index !== currentFrame) {
      currentFrame = index;
      requestAnimationFrame(() => drawFrame(currentFrame));
    }
  }
});
```

`FRAME_SPEED` 的含义：值为 2.0 表示产品动画在滚动到 50% 时就已播放完毕，后续滚动停留在最后一帧。低于 1.8 会感觉拖沓。

### V 型播放（拆解→合体→拆解）

适用于产品先展示完整外观，滚动拆解后再合体：

```javascript
function scrollToFrame(progress) {
  const EXPLODE_END = 0.45;
  const HOLD_END = 0.55;
  if (progress <= EXPLODE_END) {
    return Math.floor((progress / EXPLODE_END) * (FRAME_COUNT - 1));
  } else if (progress <= HOLD_END) {
    return FRAME_COUNT - 1;
  } else {
    const t = (progress - HOLD_END) / (1 - HOLD_END);
    return Math.floor((1 - t) * (FRAME_COUNT - 1));
  }
}
```

## Hero → Canvas Circle-Wipe

```javascript
ScrollTrigger.create({
  trigger: scrollContainer,
  start: "top top",
  end: "bottom bottom",
  scrub: true,
  onUpdate: (self) => {
    const p = self.progress;
    hero.style.opacity = Math.max(0, 1 - p * 15);
    const wipeProgress = Math.min(1, Math.max(0, (p - 0.01) / 0.06));
    canvasWrap.style.clipPath = `circle(${wipeProgress * 75}% at 50% 50%)`;
  }
});
```

## Dark Overlay（统计数据段）

```javascript
function initDarkOverlay(enter, leave) {
  const fade = 0.03;
  ScrollTrigger.create({
    trigger: scrollContainer,
    start: "top top",
    end: "bottom bottom",
    scrub: true,
    onUpdate: (self) => {
      const p = self.progress;
      let opacity = 0;
      if (p >= enter - fade && p <= enter) opacity = 0.85 * ((p - (enter - fade)) / fade);
      else if (p > enter && p < leave) opacity = 0.85;
      else if (p >= leave && p <= leave + fade) opacity = 0.85 * (1 - (p - leave) / fade);
      darkOverlay.style.opacity = opacity;
    }
  });
}
```

## 项目结构

```
project-root/
  index.html
  css/style.css
  js/app.js
  frames/frame_0001.webp ...
```

无需构建工具。纯 HTML/CSS/JS + CDN 库。
