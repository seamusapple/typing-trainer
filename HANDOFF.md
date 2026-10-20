# 指法训练 · 部署交接

给负责部署到 GitHub Pages 的 AI / 同学。本文自包含，不需要会话历史。

## 线上地址

已部署：https://seamusapple.github.io/typing-trainer/ （仓库 github.com/seamusapple/typing-trainer，Pages 从 main 根目录发布；提交人用 seamusapple 的 noreply 邮箱）。改完 `index.html` 直接 push 到 `main`，约 1 分钟生效。

## 这是什么

- 一个单文件网页：`index.html`，零依赖、零构建。打开即用；除「AI 周计划」外完全离线可用。
- 练习数据存浏览器 `localStorage`（key `typing-trainer.v1`），页面内有「导出 / 导入」。
- 字体走一条 Google Fonts 链接（Inter / Noto Sans SC 用于界面，IBM Plex Mono 用于打字文本和键帽），断网时自动回退系统字体，不影响功能。
- 视觉：浅鼠尾草底 `#f4f6f0`、白卡片、深绿 `#2f4f3a`、陶土色 `#d98e73` 为唯一暖色强调；页眉右上的太阳（夜间为月牙）按本机时间移动。深色模式跟随系统，也可在 `<html>` 上写 `data-theme="light"|"dark"` 强制。

## 目录

```
typing-trainer/
├── index.html   # 全部代码（样式 + 标记 + 脚本）
└── HANDOFF.md   # 本文
```

## 部署步骤（GitHub Pages，branch 模式）

1. 在 GitHub 新建**公开**仓库，建议名 `typing-trainer`（私有仓的 Pages 需要付费计划）。
2. 本地初始化并推送。提交人邮箱用**个人邮箱**，不要用公司邮箱：

```bash
cd ~/Codes/Personal/Projests/typing-trainer
git init -b main
git config user.email "<个人邮箱>"
git add index.html HANDOFF.md
git commit -m "typing trainer: single-file touch-typing coach"
git remote add origin git@github.com:<用户名>/typing-trainer.git
git push -u origin main
```

3. 开启 Pages：仓库 Settings → Pages → Build and deployment → Source 选 **Deploy from a branch**，Branch 选 `main` / `/ (root)`，保存。
   用 gh CLI 等价于：

```bash
gh api -X POST repos/<用户名>/typing-trainer/pages -f 'source[branch]=main' -f 'source[path]=/'
```

4. 约 1 分钟后访问 `https://<用户名>.github.io/typing-trainer/`。

## 部署后验收（必做）

- 页面打开后先按 F 再按 J，能开始打字；打错一个键会标红且不前进。
- 刷新页面，页眉「今日 N / 15 分钟」的分钟数没有归零（说明 localStorage 可用）。
- 点「复制摘要给 AI」，按钮短暂变成「已复制」（https 下剪贴板可用）。
- 手机宽度打开不出现横向滚动条（键盘按容器宽度整体缩小，不需要横向滚动）。
- （可选，需要真实密钥）「AI 周计划」选服务、填密钥、点「生成本周计划」，出现建议和 7 行计划表；刷新后计划仍在、密钥框为空；点「清除计划」后计划消失。

## 注意事项

- **数据跟域名走**：之前在本机 `file://` 下练的记录不会自动出现在 Pages 上。迁移方式：本地页面点「导出数据」得到 JSON，线上页面点「导入数据」。
- 不要引入构建工具、框架或打包步骤；保持单文件。
- 不要加 Jekyll 主题或 README 之外的 `.md` 渲染需求；仓库根目录只需 `index.html`。
- 不要改 `localStorage` 的 key 名（`typing-trainer.v1`），改了老数据就读不到了。
- 不要把 API 密钥写进 localStorage、URL、日志或导出数据；当前实现只放在页面内存的闭包变量里。
- 后续若加 PWA（离线安装）需要 `manifest.json` + service worker，当前未做。

## 功能速览（方便验收时对照）

- 5 种模式：英文单词 / 弱键专练 / 高频词 / 代码符号（Swift + shell）/ 自定义文本
- 打错不前进 + 错后锁 0.35 秒 + 可选提示音；每轮先按 F→J 锚定食指（可关）
- 每个键按「最近 30 次」统计准确率与时延；键盘按手指分区用同色系深浅着色，可切准确率热图
- 诊断面板：按 4 条几何规则归类错键对（食指串列 / 同指串行 / 邻指借用 / 左右手镜像）+ 习惯性错按，一键生成专项文本
- 两阶段：近 5 轮准确率 ≥97% 自动进「速度阶段」，弱键模式改练最慢的键，给目标 WPM，可开节拍器
- 每日 15 分钟目标、今日计划打勾、连续达标天数、稳定的键 N/26、12 周练习热力格 + 历史走势图
- AI 周计划（见下）

## AI 周计划

- 在诊断卡之后。选服务（Claude / OpenAI / DeepSeek），模型名随服务预填（`claude-opus-5-5` / `gpt-6-astra` / `deepseek-chat`，可改），填 API 密钥，点「生成本周计划」。只有点按钮时才发请求。
- 浏览器直连三个端点：`https://api.anthropic.com/v1/messages`、`https://api.openai.com/v1/responses`（400/404 时退回 `/v1/chat/completions`）、`https://api.deepseek.com/chat/completions`。用户消息就是「复制摘要给 AI」的同一段摘要；返回按固定 JSON schema 校验后才显示。60 秒超时。
- **密钥永不落盘**：只存在页面内存的闭包变量里，不进 localStorage、URL、日志、导出文件；关页即丢。
- 计划（不含密钥）存在 localStorage 的 **`typing-trainer.plan.v1`**，形如 `{createdAt, provider, model, plan}`，刷新后还在；「清除计划」删掉它。它不在「导出数据」里。
- 「按今天练」：取今天对应的那一行；计划带专项文本 drill 时，走与诊断卡相同的自定义文本流程（载入文本、切到「自定义文本」、开一轮）；没有 drill 时切到该行的模式。
- 某服务若拒绝浏览器直连（CORS）或网络不通，状态行会提示改用「复制摘要给 AI」。
