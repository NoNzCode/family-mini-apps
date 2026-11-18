# AGENTS.md — 仓库记忆与开发规范

> 给 AI 编程助手（以及未来的自己）看的项目上下文：仓库约定、目录结构、数据格式与行为规则。修改项目结构或数据格式时请同步更新本文件。

## 仓库定位

一组面向家人的轻量前端作品：Canvas 小游戏、笔记/产出工作台、学龄前识字练习台，外加一个作品集与小工具门户站（portfolio-tools）。

- 每个项目都是**单文件 HTML（内联 CSS/JS）+ 少量资源**，无构建工具、无 npm 依赖、无 CDN 外链，可离线运行。
- 数据持久化在浏览器 localStorage，均提供 JSON 导出/导入。
- 浏览器直接打开 `index.html`，或 `python -m http.server` 起静态服务（手机访问时更方便）。

## 目录规范

```
family-mini-apps/
├── octopus-game/        # 海底大冒险（Canvas 游戏）
│   ├── index.html
│   ├── assets/          # 角色/背景原图 + seabed-clean.png 无水印背景 + anim/ 每角色 12 帧动画序列
│   └── css/ js/         # 预留空目录（.gitkeep）
├── workbench/           # 笔记情况台
│   ├── index.html
│   └── css/ js/ assets/ # 预留空目录（.gitkeep）
├── literacy-station/    # 识字启蒙台
│   ├── index.html
│   ├── __chk.js         # 题库检查点（被 .gitignore 忽略，不入库）
│   └── css/ assets/     # 预留空目录（.gitkeep）
├── portfolio-tools/     # 作品集 + 小工具（单页站）
│   └── index.html
├── AGENTS.md  CHANGELOG.md  README.md  LICENSE
```

- 项目目录名用小写英文 + 连字符，**不要用中文命名**。
- 每个项目入口统一叫 `index.html`；代码量小时保持单文件，不为了“规范”提前拆分。
- 静态资源放 `assets/`；`css/`、`js/` 目前只是占位，没有实际文件。

## 数据与存储

| 项目 | localStorage 键 | 内容 |
|---|---|---|
| workbench | `wb_notes_prod` | 产出记录 `{id,date,category,count,words,note}` |
| workbench | `wb_notes_review` | 复习项 `{id,title,category,nextReview,interval,mastery,status,lastReview}` |
| workbench | `wb_notes_material` | 素材 `{id,content,tag,source,rating,addedAt}` |
| workbench | `wb_notes_seeded` | 是否已写入示例数据（`seed()` 只执行一次） |
| literacy-station | `wb_literacy_db` | 完整状态，见下 |
| octopus-game | `octopus_best_score` | 最高分（整数字符串） |
| octopus-game | `octopus_settings` | `{muted:boolean,reducedMotion:boolean}`；读取时校验类型，损坏回退默认 |

`wb_literacy_db` 结构：

```js
{
  chars: [{ id, hanzi, pinyin, words: [], learned }],  // 字库 + 学习状态
  assignments: [{ id, date, charIds: [], done, flags }], // 每日任务记录
  rewards: [{ id, name, cost }],                        // 兑换商城
  exchanges: [{ id, rewardId, ts, status }],            // 兑换申请（pending/已确认）
  points: 0,                                            // 积分
  parentPwd: null                                        // 4 位家长密码（明文，仅防误触）
}
```

- 改动键名/字段前必须考虑老数据：先做兼容读取或迁移，再更新本文件。
- 不要为了“整洁”清空用户 localStorage；清空类操作必须二次确认。
- `.gitignore` 已忽略 `literacy-station/__chk.js`（本地数据快照），不要用 `git add -f` 强加。

## 各项目要点

### octopus-game/（海底大冒险）

- 玩家操控的是**鲨鱼**，不是章鱼。三阶段成长：幼鲨 radius 40 → 少年鲨 65 → 成年鲨 95；成长需求 6/10/14，普通猎物 +1 成长。可吃阈值：猎物 radius ≤ 玩家×0.9；危险阈值：≥ 玩家×1.15；中间为中立（不受伤也不得分）。
- 得分（`BASE_SCORE`）：小鱼 +2、章鱼/水母 +5；金色鱼（`gold`）得分×5。3 点生命，受伤保护 2s，出生保护 1s；危险来自体型较大的章鱼/水母/鱼，**无追击型大鲨鱼**。生命归零失败，成年鲨成长满通关。
- 道具泡：加速/磁铁 8s、护盾挡一次；场上最多 2 个。NPC 出生避开玩家中心安全区，防止开局/重开瞬间掉命或白送成长。
- **帧动画**：`assets/anim/` 下每角色 12 张透明 WebP 序列帧（`{type}_f01~f12.webp`），由**单张 AI 基础图经程序化网格形变派生**（脚本 `.trae/tools/build_anim.py`，输入 `assets/anim/src/{type}_base.jpg` 绿幕单帧图，抠绿幕 → bbox 裁剪 → pad 13% 最长边+16px → 网格位移形变 → 高 400px WebP）。由 `FRAME_ASSETS` 声明、`getFrames()` 校验（12 帧全成功才启用，否则回退 `assets/` 静态图）。节奏由 `ENTITY_TYPES.frameMs` 控制（fish 100 / shark 100 / octopus 140 / jellyfish 160）；鱼/鲨鱼摆尾速度随游速缩放，章鱼/水母恒定脉动；帧相位随机防止同类实体同步。
- 12 帧形变方案：鱼/鲨鱼尾部纵向位移波（摆幅 7.5%/5.5% 高），章鱼触手横向摆动+整体浮动，水母伞盖 ±8% 缩放+触手摆动；相位 `k/12` 首尾无缝循环。
- 帧的生成/切分流程见 `CHANGELOG.md` 2026-10-29 两条目；`assets/anim/src/{type}_base.jpg` 是单张基础图，保留以便重生成。
- 输入：键盘 WASD/方向键；Pointer Events 按住画面拖动跟随（鼠标/触控均可）。`P`/`Esc` 暂停，`M` 静音，`R` 重开；`visibilitychange` 后台自动暂停，`blur` 清空按键/指针。
- 渲染：秒制 dt（上限 0.033），DPR≤2，离屏背景/光束缓存；`resize` 重算画布但保留游戏状态。角色**不画落地阴影**；背景用 `assets/seabed-clean.png`（无水印），原 `seabed.png` 保留；`assets/` 静态图仍作回退，删除前确认 `anim/` 帧齐全。
- 持久化：`octopus_best_score`、`octopus_settings`（见数据表）；Web Audio 本地合成音效与**循环背景音乐**（`setMusicPlaying` 随开始/暂停/结算/静音联动），无外部音频文件。

### workbench/（笔记情况台）

- 顶部「今天要处理」：待复习（`nextReview <= 今天` 且未完成）+ 今日是否已记产出。
- 三个 tab：产出统计、复习计划、素材库。
- 产出统计：今日 / 本周（以周一为起点）/ 本月 / 连续打卡（从今天往前逐日连续计数），数据来自 `wb_notes_prod`；以下方 SVG 绘制近 14 天折线图。
- 复习：过期项标红；「完成」把 `status` 设为 `done`，`nextReview = 今天 + interval`；**interval 不会自动递增**，需要手动改间隔。
- 素材库：内容 / 标签 / 来源 / 评分，支持搜索。
- 首次运行 `seed()` 写入示例数据；「清空数据」会移除三个数据键并保留 `wb_notes_seeded`，避免示例数据被重新灌入。
- 数据量达到 30 条时顶部显示备份提醒。

### literacy-station/（识字启蒙台）

- **双数据源**：
  - 实时进度 DB：`wb_literacy_db`（上面的结构），记录 `learned`、积分、兑换、家长密码。
  - 题库检查点：`__chk.js` 定义 `window.BACKUP_STATE`（含 `chars` 等），是卡片/识字练习/组词练习/每日任务的题目来源。
  - `getQuizPool()` 优先用 `window.BACKUP_STATE.chars`，没有时才回退到 `state.chars`（内置 100 字默认题库）。
- 宝宝模式（默认）：
  - 每日任务：每次会话从题库随机抽 10 个字（`todayTask` 只缓存在内存，刷新会重抽），学会后打勾。
  - 3D 翻转卡片：仅当该字原来 `learned=false` 时首次标记学会并 +10 分（重复点不再加分）。
  - 识字练习、组词练习：答对 +5 分。
  - 积分商城：宝宝发起兑换申请（`status: pending`），家长在家长模式确认后才扣分。
- 家长模式：首次进入设置 4 位数字密码，之后校验；密码明文存在 localStorage，**只防误触，不是安全措施**。
- 「保存检查点」是**生成并下载**新的 `__chk.js`，需要手动覆盖仓库里的同名文件；浏览器不能直接写本地文件。刷新页面后 `BACKUP_STATE` 才更新。
- 「还原」把 `__chk.js` 的题库数据恢复到 `wb_literacy_db`；「清空」保留 `parentPwd` 但清掉其余数据。
- `__chk.js` 必须与 `index.html` 同级（`<script src="__chk.js">` 在页面里加载），**不能移到 `js/` 子目录**。

### portfolio-tools/（作品集 + 小工具）

- 编辑式极简风格的单页站：作品集区展示三个家庭应用卡片，小工具区提供三个浏览器内工具。
- 三个小工具均为**内存态、不写 localStorage**，刷新即重置：
  - 番茄钟：专注 25 / 休息 5 分钟切换，SVG 圆环进度，结束时 Web Audio 提示音，完成专注轮自动进入休息。
  - 字数统计：实时统计总字符、不含空格字符、中文汉字、英文单词、行数、预估阅读时长。
  - 调色板生成器：随机生成和谐五色（邻近 + 互补），点击色块复制十六进制色值。
- 纯单文件内联 CSS/JS，无外部资源、无 CDN，离线可跑；已通过部署插件公开上线（URL 见 CHANGELOG）。
- 不持久化任何用户数据，因此无 localStorage 键、无导入导出。

## 常见任务指引

- **新增小应用**：新建 kebab-case 目录 + `index.html`（自包含），按需建 `assets/`；同步更新 README 项目表、本文目录树和 CHANGELOG。
- **改题库**：优先改 `__chk.js` 的 `BACKUP_STATE`，或用家长模式保存后覆盖；改 `index.html` 里的 `defaultState()` 只影响“没有检查点文件”时的兜底题库。
- **改数据结构**：先确认老 localStorage 数据仍能读取（缺字段要补默认值），再改代码，最后更新本文数据表。
- **验收方式**：浏览器打开页面 → F12 Console 无报错；窄屏（手机尺寸）检查按钮与底部安全区；游戏另需确认背景、4 张静态角色图与 48 张动画帧加载成功，并走一遍成长/受伤/通关/重开主流程。
- 仓库没有自动化测试；改动后至少手动走一遍相关主流程。

## AI 助手行为规则

### 删除保护

- **禁止删除工作区以外的任何文件**，除非用户明确确认。工作区指本仓库目录 `family-mini-apps/`。
- 工作区内删除一律**删到回收站**（Windows 下用 PowerShell 的 `Microsoft.VisualBasic.FileSystem` 回收站方式），不要用 `rm` / `rm -rf` 永久删除。
- `git reset --hard`、`git push --force`、`git clean -fd` 等不可逆操作，执行前必须先向用户确认。

### 数据保护

- 不提交 `literacy-station/__chk.js`（含本地学习数据，`.gitignore` 已忽略）。
- 不擅自修改用户 localStorage 中已有的数据格式；兼容优先。
- 导入/导出、清空、覆盖检查点等操作上的二次确认提示不要删。

### 记录约定

- 完成一个功能或修复后，提醒用户可以提交；**仅在用户明确要求时执行 `git commit`**，且不要自动 `git push`。
- 提交信息写清楚改了什么（`git log` 即时间 + 简要记录，不单独维护逐次生成日志）。
- 重大更新、bug 修复与踩坑教训记入 `CHANGELOG.md`，格式：日期 + 改动 + 教训/注意事项。

## Git 约定

- `.gitignore` 已忽略编辑器/AI 工具目录（`.vscode/`、`.idea/`、`.kilo/`、`.mimosa/`、`.claude/`、`.zcode/`、`.trae/`、`.screenshots/`、`.playwright-cli/`）、`node_modules/`、系统临时文件、`literacy-station/__chk.js`。
- 提交信息用简短中文或英文描述改动即可。
- 中文内容只出现在 HTML/JS/文档内部，文件名和目录名一律英文。
