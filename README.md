# TechVocab · 每日科技英语 (Premium Edition)

> 每天早上 9:00，从全球 5 大科技媒体自动精选 5 篇文章 → AI 摘要 + 英语学习角 → 推送到这个 App → 我在通勤路上用 3D 翻面卡片学新词 + 用间隔重复算法复习旧词。
>
> **零依赖、单文件 App、可离线、可分享、可面试。**

[![demo: techvocab](https://img.shields.io/badge/线上体验-TechVocab-5b5ff7?style=for-the-badge)](https://b598ae6761d5426a8fb85453136707c2.app.workbuddy.link)
![tech stack](https://img.shields.io/badge/HTML5-单文件-E34F26?style=for-the-badge&logo=html5)
![tech stack](https://img.shields.io/badge/CSS3-玻璃拟态-1572B6?style=for-the-badge&logo=css3)
![tech stack](https://img.shields.io/badge/JS-Vanilla-F7DF1E?style=for-the-badge&logo=javascript)
![license](https://img.shields.io/badge/license-MIT-success?style=for-the-badge)

---

## ✦ 为什么做这个

我是 AI 产品经理方向求职者，每天阅读英文科技新闻时手边需要一个轻量的「生词本 + 复习器」。市面上的"不背单词 / 多邻国"模式固定、单词库陈旧，且没有「**让我的工作内容自动成为学习材料**」的闭环。

所以我做了一个：

- 我每天读什么、App 第二天就学什么
- 不靠 SSR、不靠后端、不靠打包链，1 个 HTML 文件就是完整产品
- 视觉上配得上让面试官看到 30 秒就想点开的那种

---

## ✦ 数据管道架构

```
┌─────────────────────────────────────────────────────────────┐
│  automation-1788601298265 · 每日 09:00 (rrule=FREQ=DAILY)    │
└─────────────────────────────────────────────────────────────┘
        │
        ▼  5 大媒体
   ┌────────┬──────────┬──────────┬──────────┬──────────┐
   │ ZDNet  │ ITWorld  │ InfoWorld│Enterprise│ InfoWeek │
   │  .com  │  .com    │  .com    │  Innov.  │    .com  │
   └────────┴──────────┴──────────┴──────────┴──────────┘
        │
        ▼  5 篇文章 + 中文摘要 + 5-8 高频词
   ┌─────────────────────────────────────────────────────┐
   │  vocab-app/data.js   window.WORDS_DATA = {...}      │
   │  · id: dMMDD-NN                                     │
   │  · term / phonetic / meaning / sentence / sentenceCn│
   └─────────────────────────────────────────────────────┘
        │
        ▼  PATCH
   ┌─────────────────────────────────────────────────────┐
   │  GitHub Gist (fyf-tech/2e8aa28...)                  │
   │  raw → https://gist.githubusercontent.com/.../data.json │
   └─────────────────────────────────────────────────────┘
        │
        ▼  syncRemote() at app start
   ┌─────────────────────────────────────────────────────┐
   │  TechVocab App (你正在看的这个)                       │
   │  fetch → cache(localStorage) → 离线回退 data.js     │
   └─────────────────────────────────────────────────────┘
        │
        ▼  CloudStudio (workbuddy_cloudstudio_deploy)
   https://b598ae6761d5426a8fb85453136707c2.app.workbuddy.link
```

整个管道没有一台自己的服务器。Gist 当数据库，CloudStudio 当 CDN，App 在浏览器里跑。

---

## ✦ 核心算法：SRS 间隔重复

学新词 → 记住后 box=2（明天复习），模糊 box=1（今天再复习），没记住 box=1（重置）；每次答对 box+1：

```js
const INTERVALS = [0, 1, 2, 4, 8, 16, 32];   // 天
const MASTER_BOX = 5;                         // box≥5 = 已掌握
```

| 评分       | box 变化       | 下次到期    | 含义               |
| ---------- | -------------- | ----------- | ------------------ |
| 没记住 😵   | 1              | 1 天后      | 完全重置记忆曲线   |
| 模糊 🤔     | 当前 box（≤1） | 1 天后      | 不进位，再看一次   |
| 记住了 😀   | box + 1        | INTERVALS[box] 天后 | 推进记忆曲线 |

box=5 + 完成最后 1 次复习 → 标为"已掌握"，从今日任务队列移除。

这与 Anki / SuperMemo 的简化版同构，但少了算法复杂度，适合单文件部署。

---

## ✦ 视觉与交互亮点

### 1. 三主题（auto / light / dark）
- CSS 自定义属性分层：`html[data-theme="..."]` 切 `--bg-0 / --bg-1 / --panel / --ink / --line / --grad`
- auto 模式跟系统 `prefers-color-scheme`，深色背景配 `radial-gradient` 双 blob 模糊
- meta theme-color 也跟随，App 切到后台时状态栏颜色一致

### 2. 玻璃拟态 (Glassmorphism)
- 顶部 streak pill、底部 nav、bottom sheet 全部 `backdrop-filter: blur(...)`
- 渐变描边 + 半透白 + 投影，让卡片有悬浮质感

### 3. 圆环进度 (Conic-gradient Rings)
- 不依赖任何 SVG/canvas，纯 CSS `conic-gradient(var(--ringA) calc(var(--p)*1%), var(--track) 0)`
- 双环：今日新词（紫色 #5b5ff7）+ 待复习（青色 #19b9cc），完成时打勾变 fade-out

### 4. 3D 翻面学习卡
- `transform-style: preserve-3d` + `backface-visibility: hidden`
- 正面显示单词 + 音标 + 来源 + "点击翻面"
- 背面显示词性释义 + 双语例句 + 评分按钮

### 5. TTS（Web Speech API）
- 翻面前后都有喇叭图标，en-US 发音
- 词库 80 词全部自带音标

### 6. confetti 完成动画
- Canvas 实现，答完 8/8 触发，可视化奖励

---

## ✦ 文件结构

```
vocab-app/
├── index.html      # Premium Edition：HTML+CSS+JS 全在这一份
├── data.js         # 兜底词库（80 词），网络失败时使用
├── README.md       # 你正在读
└── qa-*.png        # Playwright QA 截图（交付前自测）
```

总共 < 100 KB，gzip 后约 25 KB。

---

## ✦ 本地预览

```bash
cd vocab-app
python -m http.server 8899
# 浏览器打开 http://127.0.0.1:8899/index.html
```

或者直接双击 `index.html`（部分浏览器本地 fetch 会被拦截，推荐 http server）。

---

## ✦ 部署

**当前生产环境**：CloudStudio 静态托管 → `workbuddy_cloudstudio_deploy` 工具一键发布。
**词库数据源**：GitHub Gist（同步 + PATCH 自动化）。
**未来规划**：GitHub Pages（公开仓库后自动启用）。

---

## ✦ 我在用什么"vibe coding"它

| 工具             | 用途                                                   |
| ---------------- | ------------------------------------------------------ |
| WorkBuddy 主对话 | PM 视角拆需求、设计 UX、决定技术取舍                   |
| Claude / GPT     | 重构复杂 CSS 段、生成 Boilderplate、补充交互细节       |
| Playwright MCP   | 每改一版都用真实浏览器跑一遍学习流程 + 截图验收        |
| PowerShell + git | 文件操作、版本控制                                     |
| Local HTTP server| 解决浏览器 `file://` 协议拒绝 fetch 的问题             |

整个 App 没装 `npm install`，没装 webpack/vite，连 `package.json` 都没有。
**这是 vibe coding 的核心约束：让代码服务于 PM 的产品判断，而不是反过来。**

---

## ✦ Roadmap

- [ ] PWA（service worker 离线缓存，让 App 真正"装"到手机）
- [ ] 例句音频缓存（当前每次 TTS 都从浏览器拉一次）
- [ ] 学习日历视图（年度热力图）
- [ ] 多用户 / 班级模式（一个 Gist 多人共享词库）
- [ ] iOS Shortcuts 集成（每日 9:00 推送学习提醒）

---

## ✦ License

MIT — 拿去改造，标注一下原作者更佳。

---

<sub>Built with 💜 by fyf-tech · 数据每天 9:00 自动更新 · 邮箱：fyf.tech@proton.me</sub>
