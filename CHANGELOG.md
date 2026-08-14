# CHANGELOG — 更新日志

> 记录重大更新、bug 修复与踩坑教训。新条目加在最上面，格式：日期 + 改动 + 教训。

## 2026-08-14 仓库整理与目录规范化

**改动：**

- 删除 VSCode/AI 工具缓存目录（`.vscode/`、`.kilo/`、`.mimosa/`），并加入 `.gitignore`。
- 中文目录改名：`工作台` → `workbench`，`识字启蒙台` → `literacy-station`。
- 入口统一为各项目的 `index.html`（原 `笔记情况台.html`、`识字启蒙台.html` 改名）。
- 各项目补齐 `css/`、`js/`、`assets/` 目录（`.gitkeep` 占位）。
- 新建 `AGENTS.md`（仓库规范与 AI 助手行为规则，合并原 `REPO_MEMORY.md`）、本文件 `CHANGELOG.md`，重写 `README.md` 加项目目录。

**教训：**

- `rm -rf` 是永久删除、不进回收站——本次误删 `.vscode` 后确立规则：工作区内删除一律走回收站。
- `literacy-station/__chk.js` 是题库检查点，被 `index.html` 以同级相对路径 `<script src="__chk.js">` 引用，且家长模式「保存检查点」依赖覆盖该文件，**不能移到 `js/` 子目录**。
- `.mimosa/` 等插件缓存目录会被工具自动重建，忽略即可，不要反复手动删。
