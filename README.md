# brand-website-design

一套将产品视频转为滚动驱动品牌主页的 Claude Code 工作规范，包含前端设计准则与两个可复用的 Skill。

## 包含内容

```
.
├── CLAUDE.md                                    # 前端编码通用规范
└── .claude/skills/
    ├── frontend-design/                         # 官方 Skill（Anthropic，Apache 2.0）
    └── yuwei-video-to-website/                  # 原创 Skill（yuwei-x，CC BY-NC 4.0）
```

### `CLAUDE.md`

项目级编码规范，约定了：

- 写前端前必须调用哪个 Skill
- 参考图片的精确匹配流程（截图对比 ≥2 轮）
- 本地服务器启动与截图工作流
- 输出默认值（文件结构、技术栈、占位图）
- Anti-Generic 规则：配色、阴影、排版、动效、交互状态、间距

### `frontend-design` Skill

由 Anthropic 官方维护，创建有辨识度、生产级别的前端界面，避免千篇一律的 AI 风格。

源仓库：[anthropics/skills/skills/frontend-design](https://github.com/anthropics/skills/tree/main/skills/frontend-design)

### `yuwei-video-to-website` Skill

将产品视频自动转换为滚动驱动的品牌主页。

功能：
- ffmpeg 自动抽帧（输出 WebP）
- Canvas API 逐帧渲染 + 滚动绑定
- GSAP + ScrollTrigger 动效编排（走马灯、数字计数器、交错 reveal）
- Lenis 平滑滚动
- 14 项 Premium Checklist 底线保障

## 安装

把下方这段话直接发给你的 Claude Code，它会帮你完成所有操作：

> 请帮我把以下 GitHub 仓库克隆到桌面，文件夹命名为 BrandWebsiteDesign。如果 Git 没有安装，请先帮我安装。克隆完成后，告诉我文件夹在哪里、包含哪些文件。
>
> 仓库地址：https://github.com/yuwei-x/brand-website-design.git
> 目标路径：~/Desktop/BrandWebsiteDesign

## 使用方式

在新项目里使用这套规范：

1. 把 `CLAUDE.md` 复制到你的项目根目录
2. 把 `.claude/skills/` 目录复制到你的项目 `.claude/` 目录下
3. 用 Claude Code 打开项目，规范自动生效

触发方式：
- **视频转网站**：「把这个视频做成一个滚动品牌主页」→ 触发 `yuwei-video-to-website`
- **前端设计**：「做一个产品落地页」→ 触发 `frontend-design`

## 许可证

| 文件 | 许可证 |
|---|---|
| `CLAUDE.md`、`yuwei-video-to-website/` | [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)（© 2026 yuwei-x） |
| `frontend-design/` | [Apache 2.0](https://github.com/anthropics/skills/blob/main/LICENSE)（© Anthropic） |
