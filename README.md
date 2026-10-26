# 科技晨报 · Tech Morning Paper

每天自动生成一份中文科技新闻报纸：按影响力和加权挑选至少 8 条新闻，每条附正反观点和来源，配一个丹麦冷知识，并按当地天气切换晴、阴、雨三套版式。

A daily Chinese-language tech newspaper: at least 8 stories picked by impact plus weights, each with sourced pro and con views, one Danish fun fact, and a sunny, cloudy or rainy look chosen by the local weather.

---

## 目录 · Contents

- [功能 · Features](#功能--features)
- [文件结构 · Files](#文件结构--files)
- [工作流程 · How it works](#工作流程--how-it-works)
- [推送机制 · Delivery](#推送机制--delivery)
- [可配置项 · Configuration](#可配置项--configuration)
- [手动运行 · Manual run](#手动运行--manual-run)
- [数据格式 · Edition format](#数据格式--edition-format)
- [已知限制 · Known limits](#已知限制--known-limits)

---

## 功能 · Features

| 中文 | English |
|---|---|
| 选题：AI 技术、AI 监管与中美欧竞争、全球芯片、美国科技大厂、单独成条的学术论文 | Topics: AI technology, AI regulation and US/China/Europe competition, global chips, US big tech, papers as stand-alone stories |
| 安全、隐私、反垄断只收重大法案；不收消费电子 | Security, privacy and antitrust only for major laws; no consumer electronics |
| 每期至少 8 条，只按“影响力 + 加权”排序，不平衡领域 | At least 8 stories, ranked purely by impact plus weights, no topic balancing |
| 四个同等加权项：欧洲（含丹麦）、具身智能、大模型、顶会精选论文 | Four equal weights: Europe (incl. Denmark), embodied AI, large models, conference-selected papers |
| 每条：发生了什么 / 为什么重要 / 正面观点 / 负面或质疑 / 来源比例 | Each story: what happened / why it matters / pro views / critical views / source mix |
| 中文来源不超过一半 | Chinese-language sources capped at 50% |
| 每期一个丹麦冷知识，不计入条数 | One Danish fun fact per issue, not counted as a story |
| 晴、阴、雨三套版式，按天气自动选用 | Sunny, cloudy and rainy looks, picked by the weather |
| 今日要闻目录、影响力条、正反观点条、来源比例图 | Headline index, impact bars, pro/con bars, source-mix charts |

## 文件结构 · Files

```
tech-morning-paper/                
├── README.md                 # 本文件 · This file
└── tech-morning-paper.zip    # 完整包，含以下全部文件 · Full package, contains all files below
    ├── SKILL.md              # 技能说明（给 Claude 的完整规则）· Skill instructions for Claude
    ├── template.html         # 报纸页面模板（三套天气版式）· Page template with three weather looks
    ├── build.py              # 把当期数据注入模板，生成 newspaper.html · Builds newspaper.html from an edition
    ├── config.json           # 天气地点与版式规则 · Weather location and look rule
    └── example-edition.json  # 一期完整数据样例 · A complete sample edition
```

运行时的工作目录是 `/mnt/project-files/tech-daily/`，每期数据存在 `editions/<YYYY-MM-DD>.json`。其他提到的文件都在zip包中

At run time the working folder is `/mnt/project-files/tech-daily/`, and each issue is saved as `editions/<YYYY-MM-DD>.json`. Other files are inside of the zip file I have uploaded. 

## 工作流程 · How it works

1. **读取规则** · Read the rules from project memory (`tech-news-scope`) and `config.json`.
2. **查天气** · Fetch the day's forecast from yr.no and pick the look:
   - 降水 ≥ 1 mm 或白天有雨 → `rainy` · ≥ 1 mm rain or daytime rain → `rainy`
   - 无降水且白天晴或少云 → `sunny` · dry and clear or fair → `sunny`
   - 其余 → `cloudy` · otherwise → `cloudy`
3. **选新闻** · Search the past 24 hours, keep 8+ stories, rank by impact plus weights.
4. **写观点** · Write pro and critical views with sources; check the Chinese-source cap.
5. **加冷知识** · Add a new, sourced Danish fun fact that hasn't appeared before.
6. **生成页面** · Write `editions/<date>.json` and run `build.py`.
7. **发布** · Publish `newspaper.html` to the same artifact link.
8. **通知** · Post a short message in the project thread.

## 推送机制 · Delivery

| 项目 · Item | 设置 · Setting |
|---|---|
| 触发方式 · Trigger | 定时任务 · Scheduled task |
| 时间 · Time | **每天丹麦时间 `8:52` · Daily at `8:52` Denmark time** (`CRON_TZ=Europe/Copenhagen 52 8 * * *`) |
| 报纸链接 · Paper link | 固定不变，每天更新 · Same link, updated daily: https://claude.ai/artifact/EpfeyC65HdPDJJDHYQiN12 |
| 通知 · Notification | 在项目线程发消息（天气和版式、头条三条、冷知识标题、链接），Claude App 推送 · A thread message (weather and look, top 3 headlines, fun-fact title, link) that triggers a Claude app notification |

## 可配置项 · Configuration

| 变量 · Variable | 当前值 · Current | 在哪改 · Where to change |
|---|---|---|
| 天气地点 · Weather location | 丹麦埃斯比约 · Esbjerg, Denmark | `config.json`（换成新城市的 yr.no 链接 · swap in the city's yr.no link） |
| 版式判定规则 · Look rule | 降水 ≥ 1 mm 为雨天 · ≥ 1 mm = rainy | `config.json`、`SKILL.md` |
| 推送时间 · Delivery time | 8:52（丹麦时间 · Denmark time） | 定时任务 · Scheduled task |
| 最少条数 · Minimum stories | 8 | 项目记忆、`SKILL.md` · Project memory, `SKILL.md` |
| 选题范围 · Topic scope | 见“功能” · See Features | 项目记忆、`SKILL.md` |
| 加权项 · Weighted topics | 欧洲、具身智能、大模型、顶会精选论文（同等）· Europe, embodied AI, large models, selected papers (equal) | 项目记忆、`SKILL.md` |
| 会议优先级 · Conference priority | 具身智能和机器学习 > 视觉和语言 · Robotics & ML > vision & language | 项目记忆、`SKILL.md` |
| 中文来源上限 · Chinese-source cap | 50% | 项目记忆、`SKILL.md` |
| 冷知识主题 · Fun-fact theme | 丹麦 · Denmark | `SKILL.md`、定时任务 · Scheduled task |
| 报纸名称、配色、版式 · Name, colours, layout | 科技晨报，三套天气配色 · 科技晨报, three weather palettes | `template.html` |

改规则时直接告诉 Claude 即可，它会同步更新记忆、文件和定时任务。

To change a rule, just tell Claude; it updates the memory, files and scheduled task together.


## 数据格式 · Edition format

```json
{
  "date": "2026-10-26", "dateZh": "2026年10月26日", "weekday": "星期一", "edition": 2,
  "weather": { "condition": "rainy", "label": "雨", "location": "Esbjerg", "locationZh": "埃斯比约",
               "summary": "…", "high": 13, "low": 11, "precip": 1.2, "wind": 12,
               "source": "yr.no", "sourceUrl": "https://www.yr.no/…" },
  "stories": [
    { "type": "news | paper", "category": "大模型", "boost": ["大模型加权"], "impact": 4,
      "short": "目录短名", "headline": "标题", "dek": "一句话说明",
      "what": "发生了什么", "why": "为什么重要", "venue": "（仅论文）会议信息",
      "refs": [{ "name": "来源", "url": "https://…", "lang": "en | zh" }],
      "pro":  [{ "text": "正面观点", "name": "来源", "url": "https://…", "lang": "en" }],
      "con":  [{ "text": "质疑观点", "name": "来源", "url": "https://…", "lang": "en" }] }
  ],
  "fact": { "title": "冷知识标题", "text": "内容", "source": "来源", "url": "https://…" }
}
```
完整样例见 `example-edition.json`。· See `example-edition.json` for a full example.

## 内容限制 · Content limits
- 内容在新闻之外加入了学术内容，关注 AI 相关期刊和顶级会议的最新发布，例如会议精选的 Highlight、Oral 和最佳论文会增加额外权重。严格来说，这部分并不属于新闻，而是反映最新科技发展的研究方向。论文条目会标注“论文”类别和发表状态（会议、预印本等），与普通新闻区分开，但仍和其他条目一起按影响力排序。

- The content adds academic part alongside the news, following the latest releases from AI journals and top conferences, such as conference-selected highlights, orals and best papers(extra weights are added). Strictly, this section is not news; it reflects the latest research directions in AI technology. Paper entries are labelled with a “paper” category and their publication status (conference, preprint and so on) to set them apart from regular news, while still being ranked by impact alongside the other stories.

- 影响力评分是编辑判断，不是量化指标。
- Impact scores are editorial judgement, not a measured metric.

- 本技能刻意采用多语言来源，不局限于某一种语言的媒体。由于报纸的默认输出语言是中文，为了避免信息只来自中文互联网，这里设置了一条硬性限制：简体中文（普通话）来源在每期全部来源中的占比必须低于 50%。英文及其他语言的来源不受此限制。每条新闻底部和右侧栏都会单独列出中文来源的数量和比例，便于核对。

- This skill deliberately draws on sources in multiple languages rather than any single language's media. Because the paper's default output language is Chinese, a hard limit keeps coverage from leaning on the Chinese-language internet: Simplified Chinese (Mandarin) sources must stay below 50% of all sources in each issue. English and other-language sources are not capped. The number and share of Chinese sources are listed separately under every story and in the side column, so the limit is easy to check.

- 本技能对每条新闻都强制设置了正反两面的讨论：无论支持一方还是质疑一方的声音有多大，读者总能同时看到正面观点和负面或质疑的观点，而且每个观点都单独标明来源。每条新闻底部还会用小图显示正反观点各有几条。我们认为这一点是本报保持客观、不偏向任何一方的基础。

- This skill requires every story to present both sides. However loud the supporting or the critical voices are, readers always see positive views alongside negative or skeptical ones, and each view cites its own source. A small chart under every story shows how many views sit on each side. We see this as the foundation of the paper’s claim to be unbiased.

## 外在限制 · External limits
- 天气只能通过 WebFetch 从 yr.no 获取；命令行访问外网会被代理拦截。· Weather comes only from yr.no via WebFetch; command-line network access is blocked by the proxy.

- 新闻条数为偶数时，两栏布局最后一行只有一条。· With an even number of stories, the last row of the two-column layout holds a single story.