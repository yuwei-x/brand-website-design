# 入场动画类型参考

科技炫酷风格建议使用 4+ 种动画类型交替出现，不连续重复同一种。
Apple 极简风格建议全部统一为 fade-up。

## 动画类型速查

| 类型 | 初始状态 | 持续时间 | 缓动 |
|------|---------|---------|------|
| fade-up | y:30-50, opacity:0 | 0.8-0.9s | power2.out / power3.out |
| slide-left | x:-80, opacity:0 | 0.9s | power3.out |
| slide-right | x:80, opacity:0 | 0.9s | power3.out |
| scale-up | scale:0.85, opacity:0 | 1.0s | power2.out |
| rotate-in | y:40, rotation:3, opacity:0 | 0.9s | power3.out |
| stagger-up | y:60, opacity:0 | 0.8s | power3.out |
| clip-reveal | clipPath:inset(100% 0 0 0) | 1.2s | power4.inOut |

## Section 动画系统实现

```javascript
function initSectionAnimations() {
  document.querySelectorAll(".scroll-section").forEach((section) => {
    const type = section.dataset.animation;
    const persist = section.dataset.persist === "true";
    const enter = parseFloat(section.dataset.enter) / 100;
    const leave = parseFloat(section.dataset.leave) / 100;

    // 定位在 enter/leave 中点
    section.style.top = ((enter + leave) / 2) * 100 + "%";
    section.style.transform = "translateY(-50%)";

    const children = section.querySelectorAll(
      ".section-label, .section-heading, .section-body, .section-note, .cta-button, .stat, .cta-inner > *"
    );

    const tl = gsap.timeline({ paused: true });

    switch (type) {
      case "fade-up":
        tl.from(children, { y: 50, opacity: 0, stagger: 0.12, duration: 0.9, ease: "power3.out" });
        break;
      case "slide-left":
        tl.from(children, { x: -80, opacity: 0, stagger: 0.14, duration: 0.9, ease: "power3.out" });
        break;
      case "slide-right":
        tl.from(children, { x: 80, opacity: 0, stagger: 0.14, duration: 0.9, ease: "power3.out" });
        break;
      case "scale-up":
        tl.from(children, { scale: 0.85, opacity: 0, stagger: 0.12, duration: 1.0, ease: "power2.out" });
        break;
      case "rotate-in":
        tl.from(children, { y: 40, rotation: 3, opacity: 0, stagger: 0.1, duration: 0.9, ease: "power3.out" });
        break;
      case "stagger-up":
        tl.from(children, { y: 60, opacity: 0, stagger: 0.15, duration: 0.8, ease: "power3.out" });
        break;
      case "clip-reveal":
        tl.from(children, { clipPath: "inset(100% 0 0 0)", opacity: 0, stagger: 0.15, duration: 1.2, ease: "power4.inOut" });
        break;
    }

    ScrollTrigger.create({
      trigger: scrollContainer,
      start: "top top",
      end: "bottom bottom",
      scrub: true,
      onUpdate: (self) => {
        const p = self.progress;
        if (p >= enter && p <= leave) {
          section.style.opacity = 1;
          section.style.pointerEvents = "auto";
          if (tl.progress() === 0) tl.play();
        } else if (persist && p > leave) {
          section.style.opacity = 1;
          section.style.pointerEvents = "auto";
        } else {
          section.style.opacity = 0;
          section.style.pointerEvents = "none";
          if (!persist) tl.progress(0).pause();
        }
      },
    });
  });
}
```

## 数字计数器

统计数字必须从 0 计数上升，不能直接显示最终值：

```javascript
document.querySelectorAll(".stat-number").forEach((el) => {
  const target = parseFloat(el.dataset.value);
  const decimals = parseInt(el.dataset.decimals || "0");

  ScrollTrigger.create({
    trigger: scrollContainer,
    start: "top top",
    end: "bottom bottom",
    onUpdate: (self) => {
      const p = self.progress;
      // 根据 stats section 的 enter/leave 范围调整
      if (p >= enterPct && p <= leavePct) {
        const t = (p - enterPct) / (leavePct - enterPct);
        const eased = 1 - Math.pow(1 - Math.min(t * 2, 1), 3);
        const val = target * eased;
        el.textContent = decimals > 0 ? val.toFixed(decimals) : Math.round(val);
      }
    },
  });
});
```

## Clip-Path 变体

| 效果 | from | to |
|------|------|-----|
| 圆形展开 | `circle(0% at 50% 50%)` | `circle(75% at 50% 50%)` |
| 左向擦除 | `inset(0 100% 0 0)` | `inset(0 0% 0 0)` |
| 底部擦除 | `inset(100% 0 0 0)` | `inset(0% 0 0 0)` |
| 菱形展开 | `polygon(50% 0%, 50% 0%, 50% 100%, 50% 100%)` | `polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)` |
