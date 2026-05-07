# T01 · final_impl_v1（待评审）

## 目标重述

落地 CodePeek 仓库根 `package.json`，定义 9 个脚本：`dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:mac:dir`，并使其在 M1 完成后能被无错调用（命令存在性即可）。

## 设计

### 1. package.json 字段

```jsonc
{
  "name": "codepeek",
  "version": "0.0.0",
  "private": true,
  "type": "module",
  "description": "Unified monitoring dashboard for IDEs and AI coding agents.",
  "license": "Apache-2.0",
  "main": "out/main/main.js",
  "engines": { "node": ">=22" },
  "scripts": {
    "dev": "electron-vite dev",
    "build": "electron-vite build",
    "preview": "electron-vite preview",
    "lint": "eslint .",
    "typecheck": "tsc -b",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:e2e": "npm run build && playwright test -c playwright.e2e.config.ts",
    "dist:mac": "npm run build && electron-builder --mac zip dmg --publish never",
    "dist:mac:dir": "npm run build && electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false"
  },
  "devDependencies": {
    "@playwright/test": "^1.58.2",
    "electron": "^41.1.0",
    "electron-builder": "^26.0.0",
    "electron-vite": "^5.0.0",
    "eslint": "^9.0.0",
    "playwright": "^1.58.2",
    "prettier": "^3.0.0",
    "typescript": "^5.6.0",
    "vitest": "^3.0.0"
  }
}
```

> 注：CodePal 用 typescript 6.0.2 与 electron 41 这种激进版本。为降低 CodePeek 早期风险，我选 typescript ^5.6（社区稳定主流），electron 41（与参考一致）。`npm install` 时 lockfile 解析具体 patch 版本。

### 2. 关键决策

| 决策 | 选项 | 选择 | 理由 |
| --- | --- | --- | --- |
| 模块格式 | CommonJS / ESM | ESM (`"type": "module"`) | electron-vite + vite + eslint flat config 全是 ESM 优先；与 CodePal 对齐。 |
| 入口 `main` | 立即写 / 留空 | `out/main/main.js` | electron-vite build 默认输出路径；T03 实现入口后 `electron .` 不会因找不到 `main` 而报错。 |
| `version` | `0.1.0` / `0.0.0` | `0.0.0` | 尚未发版；遵循 SemVer "尚未首版" 惯例。 |
| `private` | true / 省略 | `true` | 防止误发到 npm registry。 |
| `engines` | 留空 / `>=22` | `>=22` | 源设计 §1 明确锁 Node 22；与 CI / `.nvmrc`（T05）对齐前置。 |
| `lint` 脚本 | `eslint .` / `eslint . --ext .ts,.tsx` | `eslint .` | flat config 时 `--ext` 已废弃；扩展名由 `eslint.config.js` (T04) 内部 `files` 声明。 |
| `typecheck` 脚本 | `tsc --noEmit` / `tsc -b` | `tsc -b` | T02 将拆三套 tsconfig project；`tsc -b` 是 project references 唯一正确入口。 |
| `format` 脚本 | `prettier --write .` 单一 / 同时给 `format:check` | 同时 | 源设计 T04 验证步骤 `npm run lint && npm run format -- --check` 直接需要 `format:check`；现在落更省后续返工。 |
| `test:e2e` 是否 build 前置 | 是 / 否 | 是（CodePal 模式） | playwright 起 Electron 实例需 `out/` 已生成。 |
| `dist:mac:dir` 是否带 `-c.mac.identity=null` | 是 / 否 | 是 | 不签名快速本地验证；与源设计 §5.2 "本地 dist:mac:dir / dist:mac:unsigned:test 不签名跑一遍" 思路一致。 |
| 是否落 `preview` | 是 / 否 | 是 | electron-vite 自带；维护成本 = 0；后续手工调试很有用。 |
| 是否落 `test:watch` | 是 / 否 | 是 | vitest 工作流标准命令；维护成本 = 0。 |
| 是否落 `release:mac` / `seed:history` / `dist:mac:unsigned:test` | 落 / 不落 | 不落 | 越出 T01 Goal；分别由 T21 / Claude MVP / 后续打包任务负责。 |
| 是否在 T01 落 `simple-git-hooks` 配置 | 落 / 不落 | 不落 | T08 范围。 |
| 是否在 T01 落 `dependencies` | 落 / 留空 | 留空 | T03 / T15 才会触达 React / electron-updater 等运行时依赖。 |

### 3. 用户可见契约（即"脚本契约"）

下游消费者：

- AI agent skills（compatibility-first-planning / delivering-task-end-to-end / monitoring-pr-ai-reviews）会调用 `npm run lint / typecheck / test / build` 等。
- AGENTS.md 的提交前硬约束依赖这些脚本。
- 后续 T02–T09 都把这些脚本名作为既定事实直接使用。

T01 一旦合并，**这些脚本名即为 SemVer 等级稳定契约**——后续重命名等同破坏性变更，需要走 OpenSpec change。

### 4. 边界与"不过度设计"

不在 T01 做的事：

- 不写 Electron / TS / ESLint / Vitest / Playwright / electron-builder 任意配置文件。
- 不引入运行时 `dependencies`。
- 不实现 git hook / lint-staged。
- 不写 README 的脚本表（README 已是入口文档，不重复）。

只做契约 + 安装最小 dev 工具占位 + 跑通 `npm install` 让契约可被调用。

### 5. 风险与缓解

| 风险 | 缓解 |
| --- | --- |
| `npm install` 拉取 electron 41 体积大 / 网络慢 | 在 worktree 内执行；失败时记录到 todo.md，命令存在性不依赖 install 成功。 |
| TypeScript 6 vs 5.6 版本分歧后续踩坑 | 选 5.6 主流稳定版；T02 / T04 视生态适配再升级。 |
| CodePal 用 `eslint` v9 + `typescript-eslint` v8，二者匹配 | T04 任务中再校准；T01 仅装 `eslint` 占位。 |
| Lockfile 跨开发机不一致 | 与 `.nvmrc`（T05）+ CI 缓存（T09）联动；T01 先生成 lockfile 即可。 |

### 6. 验证策略（提前规划）

- 静态：`cat package.json | jq '.scripts | keys'` 包含 9 个目标脚本（含 `format:check` 等额外脚本）。
- 动态：`npm run -s` 列出全部脚本（task 板首个验证手段）。
- 命令存在性：对每个脚本调用 `npm run <name> -- --help` 或 `npm run <name> -- --version` 不返回 "missing script"。注意：本任务不要求脚本逻辑通过，只要求脚本存在 + 工具二进制可被解析。
- `npm install` 完整跑一遍并产出 `package-lock.json`。

### 7. 验收对照（Done When）

| 指标 | 计划证据 |
| --- | --- |
| 仓库根存在 `package.json` | `ls package.json` |
| `npm run lint/typecheck/test/build` 命令存在 | `npm run -s` 输出包含这 4 个脚本（M1 内其他任务负责让它们绿） |
| 首个验证：`npm run -s` 列出全部目标脚本 | terminal 输出粘贴到 verification.md |
