---
name: yuwei-video-to-website
description: |
  将产品视频转为滚动驱动品牌主页。自动抽帧、Canvas 渲染、GSAP 动效编排。
  输入：一个视频文件。输出：可部署的品牌主页。
  Turn a product video into a scroll-driven brand homepage with frame extraction,
  canvas rendering, and layered GSAP animation choreography.
  Trigger when: user provides a video file and wants a brand/product website.
---

# Video to Premium Scroll-Driven Website

将视频文件转为滚动驱动品牌主页，包含动画多样性和编排层次。

## 输入

用户提供：视频文件路径（MP4, MOV 等），可选：
- 品牌名 / 主题
- 文案内容和出现位置
- 配色偏好
- 风格方向（见 `references/style-directions.md`）

未指定的部分，简短询问或使用合理的创意默认值。

## Premium Checklist（非协商底线）

1. **Lenis 平滑滚动** — 原生滚动感觉像网页，Lenis 感觉像体验
2. **4+ 种动画类型** — 连续章节不重复同一入场动画
3. **交错 reveal** — label → heading → body → CTA，不同时出现
4. **不用 glassmorphism 卡片** — 文字在干净背景上，层次靠字号/字重/颜色
5. **方向多样性** — 章节从不同方向入场（左、右、上、缩放、clip）
6. **深色遮罩统计段** — 0.88-0.92 不透明度，数字从 0 计数，唯一允许居中文字的地方
7. **水平走马灯** — 至少一个 12vw+ 超大文字随滚动水平移动
8. **计数器动画** — 所有数字从 0 计数上升，不直接显示
9. **巨型排版** — Hero 12rem+，章节标题 4rem+，走马灯 10vw+
10. **CTA 持久** — `data-persist="true"` 保持最后章节可见
11. **Hero 独立 + 充足滚动** — Hero 占 20%+ 滚动范围，总高 800vh+（6 个章节）
12. **侧对齐文字** — 文字在外侧 40% 区域（`align-left`/`align-right`），不居中。例外：深色遮罩统计段
13. **Circle-wipe Hero 展开** — Hero 为独立 100vh，Canvas 通过 `clip-path: circle()` 展开
14. **帧速度 1.8-2.2** — 产品动画在 ~55% 滚动时完成。低于 1.8 拖沓

## 工作流

### Step 0: 前置检查

```bash
# 检测 ffmpeg 是否安装
which ffmpeg && which ffprobe
# 未安装则：
# brew install ffmpeg
```

如未安装，提示用户运行 `brew install ffmpeg`，等待安装完成后继续。

### Step 1: 视频分析

```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,duration,r_frame_rate,nb_frames -of csv=p=0 "<VIDEO_PATH>"
```

确定分辨率、时长、帧率、总帧数。决策：
- **目标帧数**：150-300 帧
  - 短视频（<10s）：原始帧率提取，上限 ~300
  - 中等（10-30s）：10-15fps
  - 长（30s+）：5-10fps
- **输出分辨率**：保持宽高比，宽度上限 1920px

### Step 2: 提取帧

```bash
mkdir -p frames
ffmpeg -i "<VIDEO_PATH>" -vf "fps=<CALCULATED_FPS>,scale=<WIDTH>:-1" -c:v libwebp -quality 80 "frames/frame_%04d.webp"
```

提取后验证：`ls frames/ | wc -l`

### Step 3: 脚手架

```
project-root/
  index.html
  css/style.css
  js/app.js
  frames/frame_0001.webp ...
```

无需构建工具。纯 HTML/CSS/JS + CDN 库。

### Step 4: 构建 index.html

必需结构（按此顺序）：

```html
<!-- 1. Loader: #loader > .loader-brand, #loader-bar, #loader-percent -->
<!-- 2. 固定 Header: .site-header > nav（logo + links） -->
<!-- 3. Hero: .hero-standalone（100vh，纯色背景，word-split 标题） -->
<!--    包含: .section-label, .hero-heading（words in spans）, .hero-tagline -->
<!--    滚动指示器 + 箭头 -->
<!-- 4. Canvas: .canvas-wrap > canvas#canvas（fixed，全视口） -->
<!-- 5. 深色遮罩: #dark-overlay（fixed，全视口，pointer-events:none） -->
<!-- 6. 走马灯: .marquee-wrap > .marquee-text（fixed，12vw 字号） -->
<!-- 7. 滚动容器: #scroll-container（800vh+） -->
<!--    内容章节: data-enter, data-leave, data-animation -->
<!--    统计段: .stat-number[data-value][data-decimals] -->
<!--    CTA 段: data-persist="true" -->
```

内容章节示例：
```html
<section class="scroll-section section-content align-left"
         data-enter="22" data-leave="38" data-animation="slide-left">
  <div class="section-inner">
    <span class="section-label">002 / Feature</span>
    <h2 class="section-heading">Feature Headline</h2>
    <p class="section-body">Description text here.</p>
  </div>
</section>
```

统计段示例：
```html
<section class="scroll-section section-stats"
         data-enter="54" data-leave="72" data-animation="stagger-up">
  <div class="stats-grid">
    <div class="stat">
      <span class="stat-number" data-value="24" data-decimals="0">0</span>
      <span class="stat-suffix">hrs</span>
      <span class="stat-label">Cold retention</span>
    </div>
  </div>
</section>
```

CDN 脚本（body 末尾，此顺序）：
```html
<script src="https://cdn.jsdelivr.net/npm/lenis@1/dist/lenis.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/ScrollTrigger.min.js"></script>
<script src="js/app.js"></script>
```

### Step 5: 构建 css/style.css

调用 **frontend-design skill** 获取创意方向。关键技术模式：

```css
:root {
  --bg-light: #f5f3f0;
  --bg-dark: #111111;
  --text-on-light: #1a1a1a;
  --text-on-dark: #f0ede8;
  --font-display: '[DISPLAY FONT]', sans-serif;
  --font-body: '[BODY FONT]', sans-serif;
}

/* 侧对齐文字区 — 产品占据中央 */
.align-left { padding-left: 5vw; padding-right: 55vw; }
.align-right { padding-left: 55vw; padding-right: 5vw; }
.align-left .section-inner,
.align-right .section-inner { max-width: 40vw; }
```

- **Hero-first 布局**：Hero 为独立 100vh 纯色背景。Canvas 隐藏，通过 circle-wipe 展开。
- **滚动章节**：`position: absolute`，定位在 enter/leave 范围中点，`transform: translateY(-50%)`。
- **移动端（<768px）**：侧对齐改为居中 + 深色背景遮罩。滚动高度降至 ~550vh。
- **文字对比度**：浅色背景正文不浅于 `#666`，深色背景文字不低于 `#ccc`。

风格方向的具体字体、配色、动画策略见 `references/style-directions.md`。

### Step 6: 构建 js/app.js

完整实现参考 `references/engineering.md`。核心模块：

1. **Lenis 平滑滚动** — 必须与 ScrollTrigger 连接
2. **两阶段帧加载** — 先 10 帧快速首屏，再后台加载剩余
3. **Canvas Padded Cover 渲染** — `IMAGE_SCALE` 0.82-0.90，背景色采样
4. **帧-滚动绑定** — `FRAME_SPEED` 1.8-2.2
5. **章节动画系统** — 读 `data-animation` 属性，4+ 种类型交替。详见 `references/animation-types.md`
6. **数字计数器** — 从 0 到 `data-value`
7. **水平走马灯** — 12vw+ 文字随滚动水平移动
8. **深色遮罩** — 统计段淡入/淡出
9. **Circle-wipe Hero 展开** — `clip-path: circle()` 从 0% 到 75%

可选视觉特效（科技炫酷风格）见 `references/visual-effects.md`。

### Step 7: 测试

1. 本地服务：`python3 -m http.server 3000` 或 `npx serve .`
2. 完整滚动测试 — 验证每个章节入场动画不同
3. 检查：平滑滚动、帧播放、交错 reveal、走马灯、计数器、深色遮罩、CTA 持久

## 反模式

- **循环特性卡片在固定段** — 每张卡片滚动时间太少。每个特性独立章节（8-10% 范围）
- **纯 cover 模式**（`Math.max` at 1.0）— 产品裁切到 header。用 `IMAGE_SCALE` 0.82-0.90
- **纯 contain 模式**（`Math.min`）— 露出不匹配边框
- **FRAME_SPEED < 1.8** — 产品动画拖沓
- **Hero < 20% 滚动范围** — 第一印象需要呼吸空间
- **连续章节相同动画** — 不重复同一入场类型
- **宽居中网格覆盖 canvas** — 改为侧区垂直列表
- **滚动高度 < 800vh**（6 个章节）— 一切太仓促

## 故障排查

- **帧不加载**：必须通过 HTTP 服务，不能 `file://`
- **滚动卡顿**：增大 `scrub` 值，减少帧数
- **白色闪烁**：确保所有帧加载完才隐藏 loader
- **Canvas 模糊**：应用 `devicePixelRatio` 缩放
- **Lenis 冲突**：确保 `lenis.on("scroll", ScrollTrigger.update)` 已连接
- **计数器不动**：检查 `data-value` 属性和 snap 设置
- **移动端内存问题**：帧数降至 <150，宽度降至 1280px
