# PWE-Music-Player（草稿）

> ⚠️ 本文件为 AI 草稿：内容从该项目历史会话中**原文摘录**，未经無涘加工与确认，不代表已定愿景。無涘确认后请删除本横幅，并按模板分节填写（模板见 Director/templates/ProjectVision.md）。
> 提取于 2026-08-18 · 记录者 dsh（DeepSeek Harness）

- 项目要解决常见“轻音乐电台”依赖固定歌单、听几天就开始重复的问题。自动搜索又会混入实地录音、有声书、盗版或错误授权内容，因此曲库改用人工核验白名单，只收录权利人自行开放或授权明确的录音。

- 这是一个免费、无广告、长时间不重复播放的公版与开放授权古典音乐收听站。它是纯静态 PWA，可直接通过 GitHub Pages 使用，并将收听历史保存在本地。

## 目标与验收
- （待無涘确认）

- 不做账号系统、后端服务、推荐算法、广告、捐赠或商业变现，也不在仓库托管音频文件。不采用未经人工核验的自动搜索结果，不引入框架、npm 依赖或构建流程。

## 项目参考
- （可删本节）

## 开场交代（原文摘录，AI 未加工）

> 来源：codex 会话 019fa990-2370-74c1-8168-6a81d8942b65（2026-07-29 00:30，cwd /Users/haodong/Documents/GitHub/PWE-Music-Player）
>
> 读取并完整、正确地实现给定方案。严格遵守方案第 0 节的无头运行纪律：不要提问，不要提议交互功能，最终消息只能是完成回报。
>
> <stdin>
> # PLAN — 公版古典轻音乐 PWA 播放器（PWE-Music-Player）
>
> ## 0. 无头运行纪律（最高优先级，先读这段）
>
> 你正在**无头（headless）模式**下运行，没有人会回答你的问题。
>
> - **禁止提问。** 不要在最终消息里问"你希望……吗"、"要不要我……"。任何需要决策的地方，按本文档的规定执行；本文档没规定的，选最简单可行的方案并在最终回报里用一行说明你选了什么。
> - **禁止提议交互式功能**（不要提议启动浏览器、不要提议让用户预览确认）。
> - **禁止把任务转成建议书。** 你的交付物是**代码文件**，不是方案描述。
> - **最终消息只能是完成回报**：列出你创建/修改的文件、你实际跑过的自测命令及其输出结果、以及未完成项（如有）。
> - 如果某一步卡住，跳过它继续完成其余部分，并在最终回报里明确标注"未完成：X，原因：Y"。绝不能因为一个子项卡住就整体退出。
>
> ## 1. 背景与目标
>
> 用户要一个**长时间不重复**的轻音乐（公版古典钢琴）播放器。现有流媒体的"24小时轻音乐频道"本质是固定歌单循环，听几天就重样。
>
> 目标：一个**纯静态 PWA**，从 archive.org 流式播放公版/CC 授权的古典钢琴录音，**本地记录已播曲目，保证不重复**，可在 iOS / macOS / Windows 上装成应用。
>
> **技术选型（已定，不要改）**：纯 vanilla JS + HTML + CSS，**无构建步骤、无 npm 依赖、无框架**。理由：要能直接丢到 GitHub Pages 或本地 `python3 -m http.server` 跑起来，长期零维护。
>
> ## 2. 数据源（我已实测验证，照抄即可，不要自己另找）
>
> archive.org 公开 API，**无需 API key**。
>
> ### 2.1 已验证的事实
>
> - 搜索：`https://archive.org/advancedsearch.php?q=<urlencoded>&fl[]=identifier&fl[]=title&rows=N&output=json`
> - 元数据：`https://archive.org/metadata/<identifier>` → `.files[]`，音频取 `format == "VBR MP3"`
> - 播放 URL：`https://archive.org/download/<identifier>/<encodeURIComponent(filename)>`
> - **实测过**：`https://archive.org/download/musopen-chopin/Ballade%20no.%201%20-%20Op.%2023.mp3`
>   → `HTTP 200` / `content-type: audio/mpeg` / **`access-control-allow-origin: *`**
> - **CORS 全开**，浏览器可直接流式播放，**不需要任何后端或代理**。
>
> ### 2.2 授权过滤规则（关键，必须严格实现）
>
> 按下载量排序会把**商业厂牌盗版排在最前面**。我实测过：下载量第一的 `polka-Frederic-chopin-complete-edition`（258 首）是 Deutsche Grammo
> …（截断，全文见来源会话）

> 来源：codex 会话 019fa9a5-3256-79f0-8968-acb1aaca141e（2026-07-29 00:53，cwd /Users/haodong/Documents/GitHub/PWE-Music-Player）
>
> 读取并完整实现给定的修复方案。严格遵守第 0 节无头运行纪律。注意：你的沙箱很可能没有外网访问，node scripts/build-catalog.mjs 会 fetch failed —— 这是预期的，不要因此停工，也绝对不要伪造 catalog.json 数据。你的职责是把代码改对、把单元测试补上并跑通（单元测试不需要联网）；联网重建曲库由我在沙箱外执行。若无法联网，在完成回报里如实写明哪几条完成标准未能自测。
>
> <stdin>
> # PLAN R2 — 修复曲库策展质量（PWE-Music-Player）
>
> ## 0. 无头运行纪律（最高优先级，先读这段）
>
> 你在**无头模式**运行，没有人会回答你的问题。
>
> - **禁止提问**、禁止提议交互功能、禁止把任务转成建议书。
> - **最终消息只能是完成回报**：改了哪些文件、实跑过的命令及真实输出、未完成项。
> - 卡住就跳过继续做其余部分，并标注"未完成：X，原因：Y"。绝不整体退出。
>
> ## 1. 上一轮的成果与问题
>
> 上一轮（`quietpiano-v1`）的代码质量是好的，**播放器、去重逻辑、授权过滤、测试全部通过验证**，不要重写它们。本轮**只修曲库策展**。
>
> 已验证通过、**不要动**的部分：
> - `app.js` 的去重逻辑（50 次选曲无重复，测试通过）
> - `isAllowedItem` 的授权过滤（实测 0 空 license / 0 librivox / 0 盗版）
> - URL 构造（随机 5 条实测 200 + audio/mpeg）
> - PWA / service worker / 视觉
>
> ## 2. 本轮要修的问题（实测数据）
>
> 我跑通了 `node scripts/build-catalog.mjs`，生成 681 首。实测发现：
>
> ### 问题 A：混入大量非古典音乐（严重）
>
> 681 首里 **302 首（44%）** 的 `composer` 是 `"Unknown"`，抽样内容：
>
> ```
> [Ariu_Kara]        Территория ненависти / The Territory of Hatred   ← 俄语摇滚/后朋
> [Ariu_Kara]        Крысиный лаз / Rat Hole
> [CheapWindowsLeak] 02 Yelling At Inanimate Objects                  ← indie/noise
> [ALaFlaca]         A la flaca                                        ← 拉丁
> [jamendo-153087]   Life Goes On / True Dream / Newfound Love        ← 通用库存音乐
> [casfom_00009]     casfom 00009 edcopy                              ← 垃圾文件名
> ```
>
> **根因**：`SEARCH_QUERIES` 第 3 条 `subject:(piano OR "classical piano" OR "classical music")` 太松——任何在 subject 标签里提到 "piano" 的摇滚乐队都会命中，而它们确实是 CC 授权，所以授权过滤放行了。**这是上一轮方案的规格错误，不是你的实现错误。**
>
> ### 问题 B：作曲家严重失衡（严重）
>
> ```
> 3
> …（截断，全文见来源会话）

> 来源：codex 会话 019fa9e7-351e-70a0-8ff3-739d7adea705（2026-07-29 02:05，cwd /Users/haodong/Documents/GitHub/PWE-Music-Player）
>
> 读取并完整实现给定方案。严格遵守第 0 节无头运行纪律。你的沙箱很可能没有外网，build-catalog.mjs 会 fetch failed —— 这是预期的，如实报告，绝对不要伪造 catalog.json。你的职责是把代码改对、把单元测试补上并跑通（单测不需要联网），联网重建曲库由我在沙箱外执行。
>
> <stdin>
> # PLAN R4 — 改用人工白名单 + 体裁/乐器标签筛选
>
> ## 0. 无头运行纪律（最高优先级）
>
> 无头模式，没有人会回答你。**禁止提问**、禁止提议交互功能、禁止把任务转成建议书。
> **最终消息只能是完成回报**：改了哪些文件、实跑命令的真实输出、未完成项。
> 卡住就跳过继续做其余部分并标注"未完成：X，原因：Y"，绝不整体退出。
> **绝对不要伪造 catalog.json 数据。** 你的沙箱可能没有外网；联网重建曲库由我在沙箱外跑。
>
> ## 1. 本轮背景
>
> 前三轮证明了一件事：**archive.org 的免费古典长尾无法用元数据启发式筛选。**
> 每加一道闸门就被下一类垃圾绕过——先是 Jamendo 库存音乐，然后是 aporee 实地录音（火车声、泳池声）和 audiocite 法语有声书，再然后是「Sacred Harp Singing」（那是无伴奏合唱，harp 指人声，不是竖琴）。R2 那版 1350 首里有 35% 的曲目其标注作曲家名根本不出现在曲名里。
>
> **所以本轮彻底改策略：放弃自动搜索，改用人工核验过的白名单。**
>
> 用户还提了两个新需求：
> - 管弦乐动态范围大，做背景音会突然变吵 → 需要能筛掉
> - 不限钢琴，吉他等也接受 → 乐器也要打标签
>
> ## 2. 白名单（我已逐个核验授权与内容，直接照抄，不要增删）
>
> ```js
> const CURATED = [
>   // identifier, kind, instrument, historical
>   ["musopen-chopin-complete-works-flac",          "solo",       "piano",  false],
>   ["musopen-chopin",                              "solo",       "piano",  false],
>   ["bach-well-tempered-clavier-book-1",           "solo",       "piano",  false],
>   ["master-tracks-the-open-well-tempered-clavier","solo",       "piano",  false],
>   ["Complete_Chopin_Nocturnes",                   "solo",       "piano",  false],
>   ["The_Open_Goldberg_Variations-11823",          "solo",       "piano",  false],
>   ["GRIEGPeerGyntcomplete",                       "orchestral", "orchestra", false],
>   ["TheInfiniteLuteCompilationOfZeruTaLur",       "solo",       "lute",   false],
>   ["CHOPINEtudes-Cortot
> …（截断，全文见来源会话）


## AI 访谈原文（codex）

> 访谈：codex（2026-08-18 00:28）｜ 整理：dsh ｜ 待無涘确认

我会只读核对当前项目入口、最新进展档案和工作区状态，不修改文件。




1. 这是一个免费、无广告、长时间不重复播放的公版与开放授权古典音乐收听站。它是纯静态 PWA，可直接通过 GitHub Pages 使用，并将收听历史保存在本地。

2. 项目要解决常见“轻音乐电台”依赖固定歌单、听几天就开始重复的问题。自动搜索又会混入实地录音、有声书、盗版或错误授权内容，因此曲库改用人工核验白名单，只收录权利人自行开放或授权明确的录音。

3. 项目已经公开部署，具备中英双语、已听去重、编制与乐器筛选、睡眠定时、逐曲署名、授权清单及 PWA 安装能力。当前 `catalog.json` 实际包含 1775 首、约 126.5 小时、49 位作曲家和 119 个来源，其中 381 首来自 Pandora Records；旧摘要中的 1778 首是更早一次构建快照。技术实现没有已知未完成项，但 `ProjectInfo/ProjectVision.md` 与 `ProjectProgress.md` 仍是 dsh 草稿，等待無涘复核和正式定稿。

4. 不做账号系统、后端服务、推荐算法、广告、捐赠或商业变现，也不在仓库托管音频文件。不采用未经人工核验的自动搜索结果，不引入框架、npm 依赖或构建流程。

5. 主要风险是播放依赖 Internet Archive 与 ibiblio.org，外部服务波动、文件迁移或跨域策略变化会直接影响可播性。曲目授权需要持续保守核验，并保留权利人异议后的快速移除机制；联网重建时来源数量和去重结果也可能随上游变化。当前治理阻塞是项目愿景与进展档案尚未经無涘确认，而不是代码实现阻塞。

