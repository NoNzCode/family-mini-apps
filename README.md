# family-mini-apps

一组给家人（尤其是孩子）用的轻量前端小应用：一个 Canvas 小游戏、一个笔记/产出工作台、一个学龄前识字练习台。

全部是单文件 HTML + 内联 CSS/JS，无构建工具、无外部依赖，克隆后双击 `index.html` 即可运行，数据保存在浏览器 localStorage。

## 项目一览

| 项目 | 入口 | 说明 |
|---|---|---|
| 海底大冒险（octopus-game） | [octopus-game/index.html](octopus-game/index.html) | Canvas 成长小游戏：操控**鲨鱼**从幼鲨一路长成成年鲨，吃较小猎物得分/成长，躲开较大危险生物；3 点生命、加速/磁铁/护盾道具、金色鱼高分、最高分记录、循环背景音乐；键盘 WASD/方向键或按住画面拖动，`P`/`Esc` 暂停、`M` 静音、`R` 重开 |
| 笔记情况台（workbench） | [workbench/index.html](workbench/index.html) | 产出统计（今日/本周/本月/连续打卡）、近 14 天 SVG 折线图、复习计划（逾期标红、一键完成）、素材库（标签/来源/评分/搜索）、JSON 导入导出 |
| 识字启蒙台（literacy-station） | [literacy-station/index.html](literacy-station/index.html) | 学龄前识字：每日 10 字 3D 翻卡（首次学会 +10 分）、识字练习/组词练习（答对 +5 分）、积分商城；家长模式管理题库与奖励（4 位数字 PIN） |

## 运行方式

直接双击对应项目的 `index.html`，或在仓库根目录起一个静态服务器（可选，便于手机访问）：

```powershell
# 任选其一，然后访问 http://localhost:8080/
python -m http.server 8080
# 或
npx serve .
```

- **海底大冒险**：桌面/手机浏览器均可；键盘或按住画面拖动操作。需 `octopus-game/assets/` 下角色/背景图与 `anim/` 48 张动画帧存在（背景使用无水印版 `seabed-clean.png`，原 `seabed.png` 保留）。
- **笔记情况台**：桌面/手机浏览器均可。
- **识字启蒙台**：桌面/手机浏览器均可；宝宝模式默认进入，家长模式首次进入需设置 4 位密码。

## 数据与备份

| 应用 | localStorage 键 | 说明 |
|---|---|---|
| workbench | `wb_notes_prod`、`wb_notes_review`、`wb_notes_material`、`wb_notes_seeded` | 产出记录、复习计划、素材、是否已写入示例数据 |
| literacy-station | `wb_literacy_db` | 字库与 `learned` 状态、积分、兑换记录、家长 PIN、奖励配置 |
| octopus-game | `octopus_best_score`、`octopus_settings` | 最高分；静音与减少动效设置（`{muted,reducedMotion}`），损坏时自动回退默认值 |

- 工作台和识字台都提供 JSON 导出/导入，换浏览器或清缓存前请先导出备份。
- 识字台的题库检查点保存在 [literacy-station/__chk.js](literacy-station/__chk.js)：家长模式点「保存」会生成并下载新的 `__chk.js`（需手动覆盖仓库里的同名文件），点「还原」从该文件恢复。该文件是**本地数据快照**，已在 `.gitignore` 中忽略，不会被提交；刚克隆的仓库没有它时，页面使用内置默认题库。

## 仓库结构

```
family-mini-apps/
├── octopus-game/        # 海底大冒险（Canvas 游戏）
│   ├── index.html
│   ├── assets/          # 角色/背景原图 + seabed-clean.png 无水印背景 + anim/ 每角色 12 帧动画序列
│   └── css/ js/         # 预留占位（.gitkeep）
├── workbench/           # 笔记情况台
│   ├── index.html
│   └── css/ js/ assets/ # 预留占位（.gitkeep）
└── literacy-station/    # 识字启蒙台
    ├── index.html
    ├── __chk.js         # 题库检查点（git 忽略）
    └── css/ assets/     # 预留占位（.gitkeep）
```

## 开发要点

- 保持单文件、可离线：不引入框架、构建步骤和外部 CDN 资源。
- 数据格式改动前先考虑已有 localStorage 数据的兼容性，保留导入/导出与二次确认。
- 移动端优先：按钮够大、输入框不局促、考虑底部安全区。
- 图形优先用内联 SVG / Canvas，避免外链图片。
- 目录规范、数据结构和 AI 助手行为规则见 [AGENTS.md](AGENTS.md)；重大更新与踩坑记录见 [CHANGELOG.md](CHANGELOG.md)。

## 许可

详见 [LICENSE](LICENSE)。欢迎改进、演示或作为教学素材使用。
