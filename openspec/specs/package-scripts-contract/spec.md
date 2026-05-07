# package-scripts-contract Specification

## Purpose

锁定 CodePeek 仓库根 `package.json` 的脚本名集合、命令语义与最小 devDependencies 覆盖，为 AI agent skill、CI、后续任务提供稳定接口。脚本名重命名等同破坏性变更，必须经由新的 OpenSpec change。
## Requirements
### Requirement: 仓库根 package.json 存在

CodePeek 仓库根 SHALL 存在合法的 `package.json` 文件，满足以下字段：

- `name` 为 `codepeek`
- `private: true`
- `type: "module"`
- `engines.node` 包含 `>=22` 约束
- `main` 为 `out/main/main.js`

#### Scenario: 仓库根存在 package.json 且字段合规

- **WHEN** 在仓库根执行 `node -e "const p=require('./package.json'); console.log(p.name, p.private, p.type, p.main)"`
- **THEN** 输出 `codepeek true module out/main/main.js` 且退出码为 0

#### Scenario: engines.node 锁定 >=22

- **WHEN** 读取 `package.json.engines.node`
- **THEN** 值为 `>=22`（或语义等价的 SemVer 表达式，允许 Node 22 及以上）

### Requirement: 核心脚本契约

`package.json.scripts` SHALL 同时定义以下 9 条核心脚本，且名称与语义对后续任务保持稳定（重命名等同破坏性变更，必须经由新的 OpenSpec change）：

- `dev` — 拉起 electron-vite 开发服务
- `build` — 执行 electron-vite 生产构建
- `test` — 以非 watch 模式执行 Vitest
- `test:e2e` — 先 `build` 再执行 Playwright e2e
- `lint` — 在整个仓库上执行 ESLint
- `typecheck` — 以 project references 方式执行 TypeScript 类型检查
- `format` — 使用 Prettier 写入格式化
- `dist:mac` — 先 `build` 再用 electron-builder 出 `zip` + `dmg`
- `dist:mac:dir` — 先 `build` 再用 electron-builder 出未签名目录产物（便于本地验证）

#### Scenario: 所有 9 条核心脚本可被 npm 解析

- **WHEN** 在仓库根执行 `npm run`（npm 11 下 `-s/--silent` 会静默该列表输出，故 Scenario 统一使用不带 `-s` 的 `npm run`；静态等价命令为 `node -e "console.log(Object.keys(require('./package.json').scripts).join('\n'))"`）
- **THEN** 输出列表至少包含 `dev`、`build`、`test`、`test:e2e`、`lint`、`typecheck`、`format`、`dist:mac`、`dist:mac:dir`

#### Scenario: 脚本命令语义与契约一致

- **WHEN** 读取 `package.json.scripts`
- **THEN** 对应条目为：
  - `dev` = `electron-vite dev`
  - `build` = `electron-vite build`
  - `test` = `vitest run`
  - `test:e2e` = `npm run build && playwright test -c playwright.e2e.config.ts`
  - `lint` = `eslint .`
  - `typecheck` = `tsc -b`
  - `format` = `prettier --write .`
  - `dist:mac` = `npm run build && electron-builder --mac zip dmg --publish never`
  - `dist:mac:dir` = `npm run build && electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false`

### Requirement: 命令二进制可被 npm 解析

`package.json.devDependencies` SHALL 至少包含每条核心脚本所依赖的顶层二进制包，使 `npm run <script>` 能够找到对应可执行文件（即便该命令因缺少配置文件而失败）。

该要求覆盖以下二进制：`electron-vite`、`eslint`、`prettier`、`tsc`（来自 `typescript`）、`vitest`、`playwright`（来自 `@playwright/test`）、`electron-builder`、`electron`。

#### Scenario: devDependencies 覆盖所有核心脚本二进制

- **WHEN** 读取 `package.json.devDependencies`
- **THEN** 同时存在以下键：`electron`、`electron-vite`、`electron-builder`、`eslint`、`prettier`、`typescript`、`vitest`、`@playwright/test`

#### Scenario: 关键脚本的 "--help" / "--version" 不报 "missing script"

- **WHEN** 在仓库根执行 `npm run lint -- --version`（以及同构地对 `typecheck` / `test` / `build` 各跑一次版本 / 帮助探针）
- **THEN** npm 不输出 `npm ERR! missing script`；具体命令可能因缺 `eslint.config.js` / `tsconfig*.json` 而退出非 0，但该行为属后续任务范围，不违反本 spec

### Requirement: 辅助脚本

`package.json.scripts` SHALL 额外定义以下辅助脚本：`preview`、`format:check`、`test:watch`。这些脚本被视为非稳定契约：后续任务 MAY 在不开新 OpenSpec change 的情况下对其增删或重命名，但首版 `package.json` 必须包含这三者。

#### Scenario: 辅助脚本在首版 package.json 中存在

- **WHEN** 在仓库根执行 `npm run`（见上方同等效的静态命令说明）
- **THEN** 输出列表包含 `preview`、`format:check`、`test:watch`

### Requirement: package-lock.json 存在

仓库根 SHALL 存在 `package-lock.json`，以锁定 `devDependencies` 的可解析版本，供 CI / 其他开发机通过 `npm ci` 获得可重复的依赖树。

#### Scenario: package-lock.json 与 package.json 同步

- **WHEN** 在仓库根执行 `npm ls --depth=0 >/dev/null 2>&1`
- **THEN** 退出码为 0，即 lockfile 与 `package.json` 描述一致

