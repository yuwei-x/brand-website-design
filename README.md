# brand-website-design

一套将产品视频转为滚动驱动品牌主页的 Claude Code 工作规范，包含前端设计准则与可复用的 Skill 模板。

## 包含内容

```
.
├── CLAUDE.md                          # 前端编码通用规范
└── .claude/skills/
    ├── frontend-design/               # 高质量前端界面生成 Skill
    └── yuwei-video-to-website/        # 视频→滚动品牌主页 Skill
```

### `CLAUDE.md`

项目级编码规范，约定了：

- 写前端前必须调用哪个 Skill
- 参考图片的精确匹配流程（截图对比 ≥2 轮）
- 本地服务器启动与截图工作流
- 输出默认值（文件结构、技术栈、占位图）
- Anti-Generic 规则：配色、阴影、排版、动效、交互状态、间距

### `frontend-design` Skill

创建有辨识度、生产级别的前端界面，避免千篇一律的 AI 风格。

功能：
- 在编码前确定大胆的美学方向（极简、最大化、复古未来、编辑风等）
- 生成可运行的 HTML/CSS/JS 或 React/Vue 代码
- 排版、颜色、间距、动效全面精细化

### `yuwei-video-to-website` Skill

将产品视频自动转换为滚动驱动的品牌主页。

功能：
- ffmpeg 自动抽帧（输出 WebP）
- Canvas API 逐帧渲染 + 滚动绑定
- GSAP + ScrollTrigger 动效编排（走马灯、数字计数器、交错 reveal）
- Lenis 平滑滚动
- 14 项 Premium Checklist 底线保障

## 使用方式

### 在新项目中使用这套规范

1. 将 `CLAUDE.md` 复制到你的项目根目录
2. 将 `.claude/skills/` 目录复制到你的项目 `.claude/` 目录
3. 在 Claude Code 中打开项目，规范会自动生效

### 触发 Skill

在 Claude Code 中，直接描述需求即可自动触发对应 Skill：

- **视频转网站**：「把这个视频做成一个滚动品牌主页」→ 触发 `yuwei-video-to-website`
- **前端设计**：「做一个产品落地页」→ 触发 `frontend-design`

## 许可证

本仓库内容（CLAUDE.md、Skill 文档、参考文档）采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) 许可证。

- 可自由使用、修改和分享
- 须署名原作者
- 禁止用于商业目的
