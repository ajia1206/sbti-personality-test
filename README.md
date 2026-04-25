# SBTI Personality Test

一个静态版 SBTI 人格测试页面，基于开源项目进行二次改造，并结合课程附件中的本地资产整理和重构而来。

线上地址：

- https://sbti-personality-test-sepia.vercel.app

GitHub 仓库：

- https://github.com/ajia1206/sbti-personality-test

## 功能

- 纯前端静态页面，无后端服务。
- 30 道常规题 + 2 道酒鬼触发题。
- 15 个维度评分，按 L / M / H 分级。
- 25 种常规人格 + HHHH 兜底人格 + DRUNK 隐藏人格。
- 深色赛博朋克视觉风格。
- 内置 UI 风格切换：Cyberpunk、Minimalist Monochrome、Swiss International、Playful Geometric。
- 单题卡片堆叠式答题流程。
- 支持整卡点选、A/B/C/D 快捷选择和 Enter 快速进入下一题。
- 结果页展示人格配图、匹配度和维度解释。

## 本地运行

在项目根目录执行：

```bash
python3 -u -m http.server 4173 --bind 127.0.0.1
```

然后打开：

```text
http://127.0.0.1:4173/
```

也可以直接打开 `index.html`，但用本地 HTTP 服务更接近线上静态站点环境。

## 目录结构

```text
.
├── index.html
├── images/
│   ├── CTRL.png
│   ├── DEAD.png
│   └── ...
├── 01_questions.md
├── 02_results.md
├── 03_scoring_logic.md
└── README.md
```

核心页面是 `index.html`。题库、结果文案和算分逻辑已经内嵌到页面脚本中，三个 Markdown 文件保留为资产来源和复查材料。

## 算分逻辑

每个常规维度由 2 道题组成，选项值为 1 / 2 / 3。两题相加后转成等级：

- `2-3`: `L`
- `4`: `M`
- `5-6`: `H`

用户的 15 维向量会与 25 种人格 pattern 计算 Manhattan 距离，距离最小者作为常规最优结果。

特殊规则：

- 如果酒鬼触发题命中，直接返回 `DRUNK`。
- 如果常规最高匹配度低于 60%，返回 `HHHH`。

## 部署

该项目可以作为静态站点直接部署到 Vercel、Cloudflare Pages 或任意静态文件服务。

当前 Vercel 部署使用的公开地址是：

```text
https://sbti-personality-test-sepia.vercel.app
```

如果重新部署，只需要发布 `index.html` 和 `images/` 目录即可。

## UI 模式

页面顶部提供 4 种 UI 模式切换，选择会写入浏览器 `localStorage`：

- `Cyber`: 默认深色赛博朋克风格。
- `Mono`: 高对比黑白、 serif 标题、锐利边框的编辑风格。
- `Swiss`: 国际主义网格、粗边框、Swiss Red 功能强调。
- `Play`: Playful Geometric，使用高饱和色块、硬阴影和圆形控件。

## 说明

本项目仅用于课程实战、前端复刻和 Prompt 工作流演示。测试结果仅供娱乐，不应作为心理诊断、职业评估或任何严肃决策依据。
