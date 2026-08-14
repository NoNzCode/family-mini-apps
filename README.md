# family-mini-apps

这是作者为个人与孩子制作的一组轻量级前端静态页面集合，包含学习、识字与小游戏等示例与工具，便于本地打开体验或作为教学参考。无需构建或服务器，浏览器直接打开即可。

## 📂 项目目录

| 项目 | 入口 | 简介 |
|---|---|---|
| 🐙 章鱼大战 | [octopus-game/index.html](octopus-game/index.html) | Canvas 小游戏：控制章鱼躲避鲨鱼、捕食鱼群，支持 WASD / 方向键 |
| 📝 工作台 | [workbench/index.html](workbench/index.html) | 笔记与产出追踪面板：打卡统计、SVG 折线图、间隔重复复习计划、素材库、数据导入/导出 |
| 🔤 识字启蒙台 | [literacy-station/index.html](literacy-station/index.html) | 学龄前识字练习：翻卡/识字/组词练习、积分兑换商城，家长模式管理题库（PIN 保护） |

## 🚀 运行方式

直接双击对应项目的 `index.html` 在浏览器打开即可：

- [octopus-game/index.html](octopus-game/index.html) — 章鱼大战（需 `assets/` 下图片存在）
- [workbench/index.html](workbench/index.html) — 工作台（手机或桌面浏览器均可）
- [literacy-station/index.html](literacy-station/index.html) — 识字启蒙台（手机或桌面浏览器均可，宝宝/家长双模式）

## 📁 仓库结构

```
family-mini-apps/
├── octopus-game/        # 章鱼大战（Canvas 游戏）
│   ├── index.html
│   └── assets/
├── workbench/           # 工作台（笔记与复习面板）
│   ├── index.html
│   └── css/ js/ assets/
└── literacy-station/    # 识字启蒙台（识字练习与积分兑换）
    ├── index.html
    ├── __chk.js         # 题库检查点数据（家长模式导出/还原）
    └── css/ js/ assets/
```

## 🛠 开发要点

- 所有页面为单文件静态实现（HTML + 内联 CSS/JS），无外部依赖，可离线使用。
- 数据以 localStorage 保存，各项目使用独立 key（工作台前缀 `wb_notes_`，识字台 `wb_literacy_db`），均提供导出/导入 JSON 备份。
- UI 侧重移动端体验：较大按钮、底部安全区、SVG 内联图形。
- 目录规范与开发约定详见 [AGENTS.md](AGENTS.md)，重大更新与踩坑记录见 [CHANGELOG.md](CHANGELOG.md)。

## 📄 许可

详见 [LICENSE](LICENSE)。欢迎改进、演示或将这些页面作为教学素材使用。
