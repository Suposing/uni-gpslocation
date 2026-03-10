# AGENTS

## 文档目的
本文件用于说明当前仓库中 AI 助手与人类协作者的分工、协作边界和执行约束，确保在 `uni-app x` 定位测试工程内开展修改时，流程清晰、上下文准确、风险可控。

## 当前项目概览
- 当前仓库是一个 `uni-app x` 示例工程，主要用于验证 `uni_modules/sup-gpslocation` 插件的前台定位、后台定位、最近位置获取与系统设置跳转能力。
- 入口文件为 `App.uvue`、`main.uts`，页面入口为 `pages/index/index.uvue`。
- 应用配置集中在 `manifest.json`、`pages.json`、`uni.scss`。
- 插件主体位于 `uni_modules/sup-gpslocation`，包含 Android、iOS、Harmony、Web 的 UTS 实现与说明文档。
- `unpackage/`、`.history/`、`.hbuilderx/` 主要为构建产物、历史记录和 IDE 配置，默认不作为业务逻辑修改目标。

## Agent 清单

### Codex Coding Agent
- 在当前仓库根目录内执行代码阅读、问题定位、代码编辑、文档补充和变更说明整理等工作。
- 默认以中文沟通；引用命令、路径、API 名称时可保留英文原文。
- 会优先依据仓库现状执行任务，不默认假设存在 `package.json`、`npm`、`pnpm` 或自定义构建脚本。
- 不会主动提交代码、发布插件或安装/配置 HBuilderX；涉及真机运行、云打包、证书签名等步骤由人工执行。

### 人类协作者
- 负责明确目标、补充业务背景、提供复现步骤与验收标准。
- 对代码、配置、权限声明、发布方式等关键修改进行审核与确认。
- 当任务涉及真机权限、系统设置、签名证书、平台差异或发布流程时，需要提前说明限制条件。

## 变更确认机制（User Approval）
1. 当我向你提问以澄清需求或范围后，在你明确回复“确认 / 同意 / 按这个做”之前，我只能进行代码阅读、分析定位、方案说明和影响面梳理，不能直接修改代码或配置。
2. 在准备修改前，我必须先说明计划改动的文件、函数或配置项，说明预期效果与可能影响。你确认后我再执行实际修改。
3. 删除或移除任何代码、文件、目录、页面、资源、路由配置、插件实现，必须单独征得你的明确同意；未获得同意不得执行删除操作。
4. **禁止丢弃未提交改动**：除非你明确说“可以丢弃 / 恢复 / 撤销某文件改动”，否则不得执行任何会清空工作区改动的 Git 或 IDE 操作，包括但不限于 `git checkout -- <file>`、`git restore`、`git reset --hard`、`git clean`、`git stash pop/drop` 及 IDE 中等价的 Discard/Undo 操作。
5. **清理无关文件需二次确认**：若发现本次需求之外的文件存在改动，只能先列出拟处理文件和原因；获得你明确同意后，才允许对这些文件执行还原或清理。

## 当前工程的协作边界

### 主要修改目标
- 页面交互与展示：`pages/index/index.uvue`
- 应用生命周期与全局样式：`App.uvue`
- 应用入口：`main.uts`
- 页面路由与窗口配置：`pages.json`
- 应用 ID、版本、平台配置与权限相关声明：`manifest.json`
- 定位插件实现与文档：`uni_modules/sup-gpslocation/**`

### 默认谨慎处理的目录
- `unpackage/`：构建输出目录，通常不手改，除非你明确要求检查构建产物。
- `.history/`：编辑器历史快照，不作为源代码修改目标。
- `.hbuilderx/`：IDE 本地配置，仅在你明确要求调整运行配置时才修改。

### 平台与能力现状
- 插件当前面向 Android / iOS 的定位测试场景，Android 最低版本要求见 `uni_modules/sup-gpslocation/package.json`。
- Harmony / Web 目录存在占位实现时，默认按“未完整实现”处理，不擅自承诺可用能力。
- 与定位相关的系统权限、后台运行、通知、系统设置跳转等行为，必须结合真实平台差异说明，不得只按单一平台假设。

## 验证与运行方式
- 当前仓库未发现 `package.json`，不要默认使用 `npm run dev`、`pnpm build`、`pnpm preview` 之类命令。
- 本项目通常通过 HBuilderX 运行到 Android / iOS 真机或模拟器进行验证；Codex 可以协助检查配置、代码和构建产物，但不替代人工完成 IDE 内运行与签名操作。
- 如需核对构建结果，优先查看 `unpackage/dist/dev/` 或 `unpackage/dist/build/` 下对应平台产物。
- 如需终端排查，优先使用只读命令，例如 `git status --short`、`rg --files`、`Get-Content -Raw manifest.json`。

## 协作流程建议
1. 明确任务目标：说明是“页面测试台调整”“插件 API 修复”“权限配置核查”还是“文档同步”。
2. 补充约束条件：说明涉及平台、是否允许新增依赖、是否允许改 `manifest.json`、是否需要兼容历史接口命名。
3. Codex 执行：先阅读相关文件，给出定位结果、改动范围和影响面；经你确认后再修改。
4. 结果验证：优先检查页面逻辑、插件接口、配置项和构建产物目录；需要真机验证的部分由人工补充回归结果。
5. 最终交付：Codex 输出变更说明、验证情况、风险点和需要人工继续执行的步骤。

## 常见任务模板
- **页面调整**：提供页面目标、交互预期、截图或文案要求 → Codex 修改 `pages/index/index.uvue` → 人工在 HBuilderX 真机预览验证。
- **插件修复**：提供复现步骤、平台、权限状态、日志现象 → Codex 检查 `uni_modules/sup-gpslocation` 实现 → 人工回归前后台定位场景。
- **配置修改**：说明要变更的版本号、权限、页面标题或路由 → Codex 修改 `manifest.json` / `pages.json` → 人工重新运行构建。
- **文档同步**：说明目标读者与需覆盖内容 → Codex 更新说明文档或注释 → 人工审核措辞与准确性。

## 最佳实践
- 在开始任务前先说明当前验证平台，例如 Android 真机、iOS 真机、模拟器或仅静态代码检查。
- 若问题与权限、后台服务、通知、系统设置页面有关，应同时提供系统版本、授权状态和复现路径。
- 修改定位插件时，要同步检查页面调用侧与文档说明是否统一使用当前命名，例如 `background`、`LocationData`。
- 若仅需调整业务源码，应避免改动 `unpackage/` 下生成文件。
- 涉及敏感配置、签名证书、AppID、密钥或账号信息时，应使用占位符并由人工在本地环境补齐。

## Git 提交信息规范（参考 git-commit-plugin）

团队提交信息格式参考 VS Code 扩展 `git-commit-plugin` 与 Conventional Commits；Codex 的职责是**输出提交信息文本**，不负责执行 `git commit`。

### Codex 输出
- 当你说“给我 commit”时，我只输出可直接复制的提交信息文本。

### 统一格式
```text
<type>(<scope>): <subject>

<body>

<footer>
```

### 可选图标
```text
<icon> <type>(<scope>): <subject>
```

默认图标：
- `🎉 init:` 项目初始化
- `✨ feat:` 添加新特性
- `🐞 fix:` 修复 bug
- `📃 docs:` 仅修改文档
- `🌈 style:` 仅修改格式，不改逻辑
- `🦄 refactor:` 代码重构
- `🎈 perf:` 性能或体验优化
- `🧪 test:` 增加或修改测试
- `🔧 build:` 构建或配置相关
- `🐎 ci:` CI 配置相关
- `🐳 chore:` 其他杂项
- `↩ revert:` 回滚变更

### 规则
- `type` 必填，优先使用 `init` `feat` `fix` `docs` `style` `refactor` `perf` `test` `build` `ci` `chore` `revert`。
- `scope` 选填，优先使用中文，例如 `定位页`、`定位插件`、`配置`、`文档`。
- `subject` 必填，中文、动词开头、不要句号，建议不超过 20 字。
- `body` 选填，用于补充背景、影响面、平台范围。
- `footer` 选填，如有关联需求、问题编号或破坏性说明，写在此处。

### 示例
- `✨ feat(定位页): 增加状态面板`
- `🐞 fix(定位插件): 修复后台权限回调`
- `🔧 build(配置): 调整 Android 权限声明`
- `📃 docs(文档): 更新协作说明`

<!-- OPENSPEC:START -->
## OpenSpec 指南

当前仓库未发现 `openspec/AGENTS.md` 或完整的 OpenSpec 目录，因此：

- 不要虚构 OpenSpec 提案、变更单或规格文件流程。
- 当后续仓库正式接入 OpenSpec 后，凡是涉及任务分解、方案设计、规格变更、架构调整或重大能力扩展，必须先读取 `openspec/AGENTS.md` 再执行。
- 若用户明确要求引入 OpenSpec 规范，应先说明仓库当前缺失相关目录与文件，再由人工决定是否补建。

请保留本管理区块的 `<!-- OPENSPEC:START -->` 与 `<!-- OPENSPEC:END -->` 标记，便于后续接入 OpenSpec 时自动更新。
<!-- OPENSPEC:END -->

## JSDoc 注释规范要求
- 所有新增或修改的 `.uvue`、`.uts`、`.ts`、`.js` 等文件中的关键函数、导出方法、复杂业务逻辑和重要状态计算，默认应遵守根目录 `JSDOC_STYLE.md` 的规范。
- 注释语言统一使用中文，优先使用 `@description`、`@property`、`@param`、`@returns`、`@event` 等标签。
- 对页面核心方法、插件导出 API、重要配置构造函数、权限处理逻辑和复杂回调，应补充完整注释；简单中间变量可按复杂度酌情省略。

## 回复语言规范
- 所有面向用户的自然语言回复必须使用中文。
- 在引用代码、文件名、命令、API、类型名和目录时，可保留其原始英文形式，但周围说明仍应使用中文。
