# 高级视觉效果库

以下效果按需使用，由品牌风格和用户 prompt 决定。科技炫酷风格可选 2-3 个，Apple 极简风格通常不使用。

## 霓虹发光（Neon Glow）

适用：Hero 标题、品牌名、关键数据。深色背景效果最佳。

```css
.neon-text {
  color: var(--color-primary-light);
  text-shadow:
    0 0 7px var(--color-primary),
    0 0 10px var(--color-primary),
    0 0 21px var(--color-primary),
    0 0 42px var(--color-primary);
}

@keyframes neon-breathe {
  0%, 100% {
    text-shadow:
      0 0 7px var(--color-primary),
      0 0 10px var(--color-primary),
      0 0 21px var(--color-primary);
  }
  50% {
    text-shadow:
      0 0 10px var(--color-primary),
      0 0 20px var(--color-primary),
      0 0 42px var(--color-primary),
      0 0 82px var(--color-primary);
  }
}

.neon-text { animation: neon-breathe 3s ease-in-out infinite; }
```

## 故障特效（Glitch Text）

适用：科技风章节标题。通过 ScrollTrigger 控制激活/取消。

```css
.glitch-text {
  position: relative;
}

.glitch-text.active::before,
.glitch-text.active::after {
  content: attr(data-text);
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
}

.glitch-text.active::before {
  color: #ff00ff;
  clip-path: inset(0 0 60% 0);
  animation: glitch-top 2.5s infinite linear alternate-reverse;
}

.glitch-text.active::after {
  color: #00ffff;
  clip-path: inset(40% 0 0 0);
  animation: glitch-bottom 3s infinite linear alternate-reverse;
}

@keyframes glitch-top {
  0% { transform: translate(0); }
  20% { transform: translate(-3px, 3px); }
  40% { transform: translate(3px, -2px); }
  60% { transform: translate(-2px, 1px); }
  80% { transform: translate(1px, -3px); }
  100% { transform: translate(0); }
}

@keyframes glitch-bottom {
  0% { transform: translate(0); }
  20% { transform: translate(3px, -2px); }
  40% { transform: translate(-3px, 3px); }
  60% { transform: translate(2px, 1px); }
  80% { transform: translate(-1px, -2px); }
  100% { transform: translate(0); }
}
```

HTML 需要 `data-text` 属性：
```html
<span class="glitch-text" data-text="钛金属表壳">钛金属表壳</span>
```

## 微粒闪烁（Sparkle Particles）

适用：CTA 按钮、品牌 Logo 周围的精致点缀。控制数量 ≤ 12。

```javascript
function createSparkles(container, count = 8) {
  container.style.position = "relative";
  container.style.overflow = "visible";

  for (let i = 0; i < count; i++) {
    const sparkle = document.createElement("span");
    sparkle.style.cssText = `
      position: absolute;
      width: ${3 + Math.random() * 3}px;
      height: ${3 + Math.random() * 3}px;
      background: var(--color-primary-light);
      border-radius: 50%;
      pointer-events: none;
      filter: blur(0.5px);
      opacity: 0;
    `;
    container.appendChild(sparkle);

    gsap.set(sparkle, {
      x: Math.random() * 300 - 150,
      y: Math.random() * 200 - 100,
    });

    gsap.to(sparkle, {
      opacity: gsap.utils.random(0.2, 0.8),
      duration: gsap.utils.random(0.8, 1.5),
      repeat: -1, yoyo: true,
      delay: gsap.utils.random(0, 2),
      ease: "power1.inOut",
    });

    gsap.to(sparkle, {
      y: "-=25",
      x: `+=${gsap.utils.random(-20, 20)}`,
      duration: gsap.utils.random(2, 4),
      repeat: -1, yoyo: true,
      ease: "sine.inOut",
    });
  }
}
```

## 全页色变过渡（Gradient Shift）

适用：章节间过渡，滚动时背景色渐变。通过 CSS 变量驱动。

```javascript
ScrollTrigger.create({
  trigger: scrollContainer,
  start: "top top",
  end: "bottom bottom",
  scrub: true,
  onUpdate: (self) => {
    const p = self.progress;
    document.documentElement.style.setProperty("--bg-hue", 20 + p * 10);
    document.documentElement.style.setProperty("--bg-saturation", p * 15);
    document.documentElement.style.setProperty("--bg-lightness", 4 + p * 2);
  },
});
```

```css
:root {
  --bg-hue: 20;
  --bg-saturation: 0;
  --bg-lightness: 4;
}
```

## 水平滚动走马灯（Marquee）

适用：科技/品牌风格，超大文字（12vw+）随滚动水平移动。

```javascript
document.querySelectorAll(".marquee-wrap").forEach((el) => {
  const speed = parseFloat(el.dataset.scrollSpeed) || -25;
  gsap.to(el.querySelector(".marquee-text"), {
    xPercent: speed,
    ease: "none",
    scrollTrigger: {
      trigger: scrollContainer,
      start: "top top",
      end: "bottom bottom",
      scrub: true,
    },
  });
});
```

```css
.marquee-wrap {
  position: fixed;
  z-index: 3;
  width: 100%;
  overflow: hidden;
  pointer-events: none;
}

.marquee-text {
  font-family: var(--font-display);
  font-size: clamp(4rem, 12vw, 10rem);
  font-weight: 900;
  color: rgba(255, 255, 255, 0.06);
  white-space: nowrap;
  will-change: transform;
}
```

## 斜切分屏画廊（Clip-Path Gallery）

适用：作品展示、产品系列。

```css
.split-gallery {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0;
  height: 100vh;
  overflow: hidden;
}

.split-item {
  overflow: hidden;
  clip-path: polygon(0 0, 100% 0, 85% 100%, 0 100%);
}

.split-item:nth-child(even) {
  clip-path: polygon(15% 0, 100% 0, 100% 100%, 0 100%);
}

.split-item img {
  width: 100%; height: 100%;
  object-fit: cover;
  transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1);
}

.split-item:hover img {
  transform: scale(1.05);
}
```

配套 GSAP ScrollTrigger 展开动画：

```javascript
gsap.utils.toArray(".split-item").forEach((item, i) => {
  gsap.from(item, {
    clipPath: i % 2 === 0
      ? "polygon(0 0, 0 0, 0 100%, 0 100%)"
      : "polygon(100% 0, 100% 0, 100% 100%, 100% 100%)",
    duration: 1.2,
    ease: "power4.inOut",
    scrollTrigger: {
      trigger: item,
      start: "top 80%",
      toggleActions: "play none none reverse",
    },
  });
});
```
