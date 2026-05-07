# Proposal · bootstrap-package-scripts

## Why

CodePeek 仓库目前没有 `package.json`，但所有 AI agent skill（`compatibility-first-planning`、`delivering-task-end-to-end`、`monitoring-pr-ai-reviews` 等）和 `AGENTS.md` 的提交前硬约束都把 `npm run lint / typecheck / test / test:e2e / build / dist:mac` 当成既定命令调用。脚本契约缺失等于让所有自动化工作流在第一步失败，必须先把脚本名作为稳定接口落地，后续 T02–T24 才有锚点。

## What Changes

- 在仓库根新增 `package.json`，字段含 `name=codepeek` / `private: true` / `type: module` / `engines.node>=22` / `main=out/main/main.js`。
- 定义脚本契约 9 项（核心）+ 4 项（辅助）：
  - 核心：`dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:mac:dir`
  - 辅助：`preview` / `format:check` / `test:watch`（不在 task 板硬要求内，零维护成本，便于后续任务）
- 引入立即可跑的最小 `devDependencies`：`electron` / `electron-vite` / `electron-builder` / `eslint` / `prettier` / `typescript` / `vitest` / `@playwright/test` / `playwright`。
- 通过 `npm install` 生成 `package-lock.json` 与 `node_modules/`，让脚本命令可被无错调用（命令存在性维度）。
- **不**新增任何源代码、tsconfig、eslint config、vitest config、playwright config、electron-builder config、git hook、`.editorconfig` 等：这些属于 T02–T08 / T18 范围。

## Capabilities

### New Capabilities

- `package-scripts-contract`: 锁定 CodePeek 仓库根 `package.json` 的脚本名集合与语义；下游所有 skill / CI / 任务都以此为稳定接口。

### Modified Capabilities

（无；CodePeek 此前不存在任何 capability。）

## Impact

- **新增文件**：`package.json`、`package-lock.json`。
- **新增目录**：`node_modules/`（已被 `.gitignore` 忽略）。
- **影响 skill**：所有调用 `npm run *` 的 skill 现在能进入第一步而不报 "missing script"。
- **影响 task 板**：T02–T09 解锁；T18 / T20 在 M5 触达时无须再改 9 个核心脚本名。
- **未触达**：源码、Electron 入口、CI workflow、签名 / 公证、自动更新、文档（属于后续任务）。
- **下游契约**：脚本名一旦合并即视为 SemVer 等级稳定，重命名等同破坏性变更，需要新的 OpenSpec change。
