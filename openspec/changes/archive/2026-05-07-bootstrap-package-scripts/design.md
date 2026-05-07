# Design · bootstrap-package-scripts

## Context

- CodePeek 仓库当前只有 `README.md` / `AGENTS.md` / `CLAUDE.md` / `LICENSE` / `.gitignore` / 三套 agent skill 目录 / 一个 `openspec/config.yaml` / `docs/plans/` 下两份 plan 文件；不存在 `package.json`、`tsconfig*.json`、源码、构建配置。
- 参考实现 [shamcleren/CodePal](https://github.com/shamcleren/CodePal) 已经在 v1.1.4 落地了 Electron + electron-vite + electron-builder + Vitest + Playwright 完整链路，其 `package.json` 是本设计的同构对象。
- 任务板 `docs/plans/bootstrap-engineering-design-task.md` 行 T01 列出了 9 条核心脚本与"命令存在性即可"验收阈值。
- 用户在认领阶段确认：T01 同时落"立即可跑的最小依赖集"并执行 `npm install`。
- workspace/T01 已产出 `current_state.md` / `reference_impl.md` / `final_impl.md`（rubric 88.6 分通过）/ `todo.md`。

## Goals / Non-Goals

**Goals:**

- 在 `/Users/yousa/Documents/Github/CodePeek` 仓库根落地 `package.json`，含 9 条核心脚本（`dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:mac:dir`）+ 3 条辅助脚本（`preview` / `format:check` / `test:watch`）。
- 装入立即可跑的最小 `devDependencies`，使每个脚本绑定的二进制（`electron-vite` / `eslint` / `prettier` / `tsc` / `vitest` / `playwright` / `electron-builder`）能被 npm 解析。
- 通过 `npm install` 生成 `package-lock.json`，保证脚本契约的可重复消费。
- 锁 `engines.node>=22`，与源设计 §1 与未来 `.nvmrc`（T05）/ CI（T09）对齐。

**Non-Goals:**

- 不实现任何 Electron / React / IPC / 持久化代码。
- 不写 `tsconfig*.json` / `eslint.config.js` / `vitest.config.ts` / `playwright.e2e.config.ts` / `electron-builder.yml`：脚本可以 *被调用*，但执行后会因缺配置而报"找不到配置"——这正符合 T01 的"命令存在性即可"边界。
- 不引入 `dependencies`：M3 / T15 / T20 才会触达运行时依赖。
- 不引入 git hook / `lint-staged`：T08 范围。
- 不写 README 的 Scripts 段落：M1 末期单独处理。

## Decisions

### D1 · `type: module` vs CommonJS

**选 ESM (`"type": "module"`)**。
理由：electron-vite 5 / vite 7 / vitest 3 / eslint 9 flat config 全部 ESM 优先；CodePal v1.1.4 也是 ESM。
备选：CommonJS——会让 eslint flat config / vitest config 写法多一层兼容样板，长期 ROI 更低。

### D2 · `main` 指向 `out/main/main.js`

理由：electron-vite build 默认输出路径；T03 实现入口后 `electron .` / `electron-builder` 直接命中。
备选：先省略 `main` 字段——会让 `electron .` 报错；与 CodePal 不一致；徒增差异。

### D3 · `typecheck` 用 `tsc -b` 而非 `tsc --noEmit`

理由：T02 将拆三套 tsconfig project（main / preload / renderer + 共享 base）。`tsc -b`（build mode）是 project references 的唯一入口，能跨 project 增量编译；`--noEmit` 在 project references 下会假阳性 / 不工作。
备选：先用 `tsc --noEmit` 等 T02 时再改——会触发不必要的脚本契约变更，违背"脚本一旦合并即稳定"承诺。

### D4 · `lint` 用 `eslint .`，不带 `--ext`

理由：eslint 9 flat config 中 `--ext` 已废弃（被 `eslint.config.js` 内 `files: ["**/*.ts", "**/*.tsx"]` 取代）。CodePal 仍用 `--ext .ts,.tsx` 是 v8 遗留。
备选：保留 `--ext`——eslint 9 会发警告或被未来版本移除支持。

### D5 · `format` + `format:check` 双脚本

理由：源设计 T04 验证步骤显式调用 `npm run format -- --check`（即 `format:check`）；现在落比后续返工省事；维护成本 = 0。
备选：仅 `format`，`format:check` 等 T04 再加——在 T04 任务里强行改 `package.json` 等于让 T04 同时承担两件事。

### D6 · `test:e2e` 在 e2e 之前 `npm run build`

理由：Playwright 起 Electron 实例需要 `out/` 已生成；CodePal 也是这个模式。
备选：跳过 build——首次跑 e2e 一定失败；`test:e2e` 不应假设 `out/` 已存在。

### D7 · `dist:mac:dir` 带 `-c.mac.identity=null -c.mac.notarize=false`

理由：本地非签名快速验证；T18 落 `electron-builder.yml` 之前必须有这两个 flag 才不会要求开发者本机有 Apple 证书。
备选：等 T18 配置 yml 后再写——T18 在 M5，与 T01 之间有数月跨度，期间不能本地验证打包。
计划：T18 落 yml 后，把这两个 flag 移入 yml，`dist:mac:dir` 简化（已写入 `workspace/T01/todo.md`）。

### D8 · 引入"立即可跑的最小依赖集"

依赖清单（major 版本对齐 CodePal v1.1.4）：

| 包 | 版本 | 用途 | 谁会真正消费 |
| --- | --- | --- | --- |
| `electron` | `^41.1.0` | 提供 `electron` 二进制 / `dev` 与 `build` | T03 / T15 |
| `electron-vite` | `^5.0.0` | `electron-vite` CLI | T03 |
| `electron-builder` | `^26.0.0` | `electron-builder` CLI | T18 |
| `eslint` | `^9.0.0` | `eslint .` | T04 |
| `prettier` | `^3.0.0` | `prettier --write/--check` | T04 |
| `typescript` | `^5.6.0` | `tsc -b` | T02 |
| `vitest` | `^3.0.0` | `vitest run` | T06 |
| `@playwright/test` | `^1.58.2` | `playwright test` | T07 |
| `playwright` | `^1.58.2` | 与 CodePal 对齐双装；T07 决定是否精简 | T07 |

理由：
- 每条脚本对应的二进制都能被 `npm exec` / `npm run` 解析。
- 不依赖 plugin 生态（@eslint/js、typescript-eslint、eslint-plugin-react 等）：留给 T04，避免 T01 强行做 T04 的事。
- `typescript ^5.6` 而非 CodePal 的 `^6.0.2`：5.6 是当前主流 LTS-like 稳定版；6.0 在 2025 后期才稳；T02 决定是否升级。
备选：完全空 devDependencies——脚本可被 `npm run` 看到，但执行立刻 "command not found"，让任何想本地试跑的人受挫；用户指令明确选了"脚本 + 立即可跑的最小依赖集"。

### D9 · 跑 `npm install` 生成 `package-lock.json`

理由：用户指令明确选了。lockfile 让脚本契约真正可重复消费；CI（T09）将依赖 `actions/setup-node@v4` 的 `cache: npm` —— 没有 lockfile 会失败。
备选：不跑 install——lockfile 缺失时 `npm ci` 不工作；T09 落 CI 时还得回 T01 补。

### D10 · 不写 README Scripts 段

理由：M1 末期 / 后续任务统一处理；T01 不应顺手扩范围。

## Risks / Trade-offs

| 风险 | 缓解 |
| --- | --- |
| `npm install` 在网络受限环境失败 → 无 lockfile | 在 worktree 内执行；如失败则在 `workspace/T01/todo.md` 记录并降级到"仅 package.json"，命令存在性靠 `npm run -s` 验证（task 板首验手段）。 |
| `electron@^41` 体积大 / 平台特定二进制 | 仅本任务一次性下载；`.gitignore` 已排除 `node_modules/`。 |
| `typescript@^5.6` 与 `eslint@^9` / `vitest@^3` peer 兼容 | T02 / T04 / T06 在自身任务中会做兼容验证；T01 仅装顶层包，无 plugin。 |
| `dist:mac:dir` 本地跑现在仍会失败（缺 `electron-builder.yml`） | 这是 T01 边界内的预期；命令存在性即可，T18 才让它绿。 |
| 9 个核心脚本之外的辅助脚本（preview / format:check / test:watch）将来被某个任务移除 | 这些是非契约脚本，可以无破坏性增删；不在 T01 的稳定接口承诺范围。 |
| 子 skill 缺失（用 todo.md 记录） | 不影响 T01 实质交付，但需在用户具备完整 skill 集时回归正式 review 流程。 |

## Migration Plan

- 部署：合并 `task/T01-bootstrap-package-scripts` 到 `main`；`package.json` 与 `package-lock.json` 即为该任务最终产物。
- 回滚：`git revert` 单 commit；其他文件未改动。
- 兼容性：仓库此前无 `package.json`，新增不会破坏既有调用。

## Open Questions

无未决问题。审核中未出现高严重度反馈；中低反馈已纳入 `workspace/T01/todo.md`。
