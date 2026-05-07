# T01 · CodePeek 现状

## 仓库结构实况

```
CodePeek/
├── .claude/ .codex/ .trae/    # 三套 AI agent skill 目录，仅含 SKILL.md
├── docs/plans/                # 已有 bootstrap-engineering-design{,-task}.md
├── openspec/config.yaml       # spec-driven 配置已就位，尚无 changes/ 目录
├── AGENTS.md / CLAUDE.md / LICENSE / README.md / .gitignore
```

## 工程化事实清单

- `package.json`：**不存在**。本任务即落地首个版本。
- `package-lock.json`：不存在。
- `node_modules/`：不存在。
- `tsconfig*.json`：不存在（T02 负责）。
- `eslint.config.js` / `.prettierrc*`：不存在（T04 负责）。
- `vitest.config.ts` / `playwright.e2e.config.ts`：不存在（T06 / T07 负责）。
- `electron-builder.yml`：不存在（T18 负责）。
- 源码目录 `src/`：不存在（T03 负责）。
- `openspec` CLI：已安装（`/opt/homebrew/bin/openspec`, v1.3.0）。
- Git：`main` 分支干净，已基于 main 开出隔离 worktree `../CodePeek-T01`，分支 `task/T01-bootstrap-package-scripts`。

## 约束与硬要求

源设计（`docs/plans/bootstrap-engineering-design.md` §1、§第二部分 P0）+ task 板 T01 行要求：

1. 脚本名必须精确覆盖：`dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:mac:dir`。
2. `"Done When"`：
   - 仓库根存在 `package.json`。
   - M1 结束后 `npm run lint/typecheck/test/build` 可被无错调用（命令存在性即可；T01 本身无需绿）。
3. 首个验证手段：`npm run -s`（仅列出脚本，不执行）。
4. 源设计 §1 明确要求锁 Node 22（`engines.node >=22`）+ 后续 T05 再落 `.nvmrc`。
5. `type: module`（CodePal 对齐），`private: true`（未发布 npm）。
6. AGENTS.md 硬约束："提交前 lint / typecheck / 测试" —— 脚本存在 = 硬约束能落地的物质基础。
7. 用户指令：允许同时落 devDependencies 最小可跑集合，并执行 `npm install` 生成 lockfile。

## 与后续任务的边界

- T02 会新建 `tsconfig.base/main/preload/renderer.json`，`typecheck` 脚本应指向 `tsc -b`（tsconfig 项目组）。T01 的 `typecheck` 指向 `tsc -b`，T02 完成时无需再改 `package.json`。
- T03 会新建 `src/main/main.ts` 等与 `electron-vite.config.ts`；T01 的 `dev` / `build` 指向 `electron-vite dev` / `electron-vite build`。
- T04 会新建 `eslint.config.js`；T01 的 `lint` 指向 `eslint .`，`format` 指向 `prettier --write .`，`format:check` 指向 `prettier --check .`。
- T06 会新建 `vitest.config.ts`；T01 的 `test` 指向 `vitest run`。
- T07 会新建 `playwright.e2e.config.ts`；T01 的 `test:e2e` 在构建后调用 `playwright test -c playwright.e2e.config.ts`。
- T18 会新建 `electron-builder.yml`；T01 的 `dist:mac` 与 `dist:mac:dir` 在构建后调用 `electron-builder`。

T01 只负责脚本契约 + 最小立即可跑依赖；不新建配置文件（除非脚本自身必需）。

## 非目标

- 不实现 Electron 入口代码（T03）。
- 不写 tsconfig / eslint / vitest / playwright 配置（T02/T04/T06/T07）。
- 不引入 `simple-git-hooks` / `lint-staged`（T08）。
- 不落 `.editorconfig` / `.gitattributes` / `.nvmrc`（T05）。
- 不落 electron-builder.yml（T18）。

这些都是后续任务的范围，T01 只保证脚本名与可被调用。
