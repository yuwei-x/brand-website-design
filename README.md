# brand-website-design

一套将产品视频转为滚动驱动品牌主页的 Claude Code 工作规范，包含前端设计准则与可复用的 Skill 模板。

## 包含内容

```
.
├── CLAUDE.md                          # 前端编码通用规范
└── .claude/skills/
    └── yuwei-video-to-website/        # 视频→滚动品牌主页 Skill（原创）
```

此外还依赖一个**官方 Skill**（不包含在本仓库中，见下方安装说明）：

| Skill | 来源 | 许可证 |
|---|---|---|
| `yuwei-video-to-website` | 本仓库 | CC BY-NC 4.0 |
| `frontend-design` | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/frontend-design) | Apache 2.0 |

### `CLAUDE.md`

项目级编码规范，约定了：

- 写前端前必须调用哪个 Skill
- 参考图片的精确匹配流程（截图对比 ≥2 轮）
- 本地服务器启动与截图工作流
- 输出默认值（文件结构、技术栈、占位图）
- Anti-Generic 规则：配色、阴影、排版、动效、交互状态、间距

### `yuwei-video-to-website` Skill

将产品视频自动转换为滚动驱动的品牌主页。

功能：
- ffmpeg 自动抽帧（输出 WebP）
- Canvas API 逐帧渲染 + 滚动绑定
- GSAP + ScrollTrigger 动效编排（走马灯、数字计数器、交错 reveal）
- Lenis 平滑滚动
- 14 项 Premium Checklist 底线保障

### `frontend-design` Skill（官方）

由 Anthropic 官方维护，创建有辨识度、生产级别的前端界面，避免千篇一律的 AI 风格。

源仓库：[anthropics/skills/skills/frontend-design](https://github.com/anthropics/skills/tree/main/skills/frontend-design)

## 安装与使用

### 第一步：克隆本仓库

```bash
git clone https://github.com/yuwei-x/brand-website-design.git
cd brand-website-design
```

### 第二步：安装官方 `frontend-design` Skill

从 Anthropic 官方仓库单独下载（确保始终使用最新版本）：

```bash
# 方式一：使用 gstack（如已安装）
gstack skills install frontend-design

# 方式二：手动克隆官方 skill
git clone --depth 1 --filter=blob:none --sparse \
  https://github.com/anthropics/skills.git /tmp/anthropic-skills
cd /tmp/anthropic-skills
git sparse-checkout set skills/frontend-design
cp -r skills/frontend-design ~/.claude/skills/frontend-design
```

### 第三步：在项目中使用

```bash
# 将规范文件复制到你的项目
cp CLAUDE.md /your-project/
mkdir -p /your-project/.claude/skills

# 链接原创 skill
ln -s ~/.claude/skills/yuwei-video-to-website /your-project/.claude/skills/
# 链接官方 skill
ln -s ~/.claude/skills/frontend-design /your-project/.claude/skills/
```

在 Claude Code 中打开项目后，直接描述需求即可触发对应 Skill：

- **视频转网站**：「把这个视频做成一个滚动品牌主页」→ 触发 `yuwei-video-to-website`
- **前端设计**：「做一个产品落地页」→ 触发 `frontend-design`

## 许可证

- `CLAUDE.md`、`.claude/skills/yuwei-video-to-website/` 及本仓库其余原创内容：[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)（© 2026 yuwei-x）
- `frontend-design` Skill 由 Anthropic 官方维护，遵循 [Apache 2.0](https://github.com/anthropics/skills/blob/main/LICENSE)，**不包含在本仓库中**
