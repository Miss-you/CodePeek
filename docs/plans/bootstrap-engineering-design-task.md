# Bootstrap Engineering Design — Task Board

## Source Design

- 源设计：[`docs/plans/bootstrap-engineering-design.md`](./bootstrap-engineering-design.md)
- 范围：CodePeek 仓库从空脚手架走向 macOS arm64 可发版桌面应用的工程化基础设施落地。
- 任务总数：24（覆盖源设计 P0–P2，P3 不进入任务板）。
- 任务文档生命周期：本文档由 `deriving-task-board-from-design` skill 生成，由 `delivering-task-end-to-end` 推动状态流转。

## Status Legend

| 状态 | 含义 |
| --- | --- |
| `todo` | 已识别但未认领 |
| `claimed` | 已认领并完成 worktree / workspace 建立 |
| `research` | 正在产出 `current_state.md` / `reference_impl.md` |
| `spec` | 已产出 `final_impl.md`，OpenSpec change 与 `test_strategy.md` 在编写中 |
| `implementing` | 已批准 spec，按 TDD 写代码 |
| `verifying` | lint / typecheck / test / build / e2e 等验证执行中 |
| `review` | 在做 code review、问题分流，或在跟 PR 上 AI review |
| `blocked` | 被外部阻塞；解除后回到 `Notes.resume_to` 记录的状态 |
| `done` | 完成 final compare、OpenSpec change 已归档、task 文档闭环 |

`ready` = `status=todo` 且所有硬依赖已 `done`，由依赖关系推导得出，不作为独立状态。

## Dependency Rules

1. **硬依赖闭环**：只有当 `Depends On` 中所有任务为 `done`，本任务才能进入 `claimed`。
2. **里程碑边界**：里程碑 M1 → M2 → M3 → M4 → M5 → M6 之间存在隐式硬依赖（除非另有标注）。同一里程碑内的任务可按 `Parallel` 标注并行。
3. **冻结契约前置**：`T11–T13`（结构化日志、userData 约定 + Settings schema、持久化 schema 版本化、IPC 契约）必须在任意 adapter 实现（`T15–T17`）开始前 `done`。
4. **签名公证前置**：`T19`（macOS 签名 / 公证）必须在 `T20`（auto-update）和 `T21`（release.yml）之前 `done`，否则 release 产物不可信。
5. **OpenSpec change 一对一**：每个任务默认对应一个 OpenSpec change（除显式 `n/a`）。

## Task Table

> 列说明：`Workspace` 一律落在 `workspace/<task-id>/`；`Change` 列示 OpenSpec change 名（落在 `openspec/changes/<change-id>/`），`n/a` 表示该任务显式不开 change（仅限文档 / 仓库元数据级改动）。`Parallel` 标记同里程碑内可并行的组。

### M1 · 脚手架可跑（P0）

| ID | Title | Goal | Depends On | Parallel | Status | Owner | Claimed At | Workspace | Change | Done When | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T01 | Bootstrap package.json + scripts contract | 落地 `package.json`，定义 `dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:mac:dir` 脚本 | — | — | todo | — | — | `workspace/T01/` | `bootstrap-package-scripts` | 仓库根存在 `package.json`；`npm run lint` / `typecheck` / `test` / `build` 四个命令在 M1 完成后能被无错调用（命令存在性即可，本任务无需绿） | 首个验证：`npm run -s` 列出全部目标脚本 |
| T02 | Three-project tsconfig (main / preload / renderer + base) | 拆 `tsconfig.base.json` + `tsconfig.main.json` + `tsconfig.preload.json` + `tsconfig.renderer.json`，避免 typecheck 假阳性 | T01 | A | todo | — | — | `workspace/T02/` | `bootstrap-tsconfig-projects` | `tsc -b` 能成功跨项目构建空骨架；renderer DOM、main Node lib 不再相互泄漏 | 首个验证：`npm run typecheck` |
| T03 | electron-vite 三进程入口骨架 | 建立 `src/main/main.ts`、`src/preload/index.ts`、`src/renderer/main.tsx` + `index.html`，启用 `contextIsolation` / `sandbox` / `nodeIntegration: false` | T01, T02 | A | todo | — | — | `workspace/T03/` | `bootstrap-electron-vite-shell` | `npm run dev` 能拉起一个空白 BrowserWindow；preload 通过 `contextBridge` 暴露空白白名单 | 首个验证：`npm run dev` 真机冒烟 |
| T04 | ESLint flat config + Prettier 集成 | `eslint.config.js`（@eslint/js + typescript-eslint + react + react-hooks + import）+ `prettier` + `eslint-config-prettier` | T01, T02 | B | todo | — | — | `workspace/T04/` | `bootstrap-lint-format` | `npm run lint` 与 `npm run format` 互不冲突，且对最小骨架返回 0 | 首个验证：`npm run lint && npm run format -- --check` |
| T05 | Repo hygiene 文件 | `.editorconfig` / `.gitattributes` / `.gitignore`（含 `out/` `release/` `node_modules/` `playwright-report/` `.env*`）/ `.nvmrc`（node 22） | — | B | todo | — | — | `workspace/T05/` | n/a | 上述五个文件存在并被 git 跟踪；`.gitignore` 在 M5 触达 `release/` 时仍生效 | `n/a`：仓库元数据级，不开 OpenSpec change |
| T06 | Vitest 最小骨架 + smoke 单测 | `vitest.config.ts` 分 node / jsdom 环境；`tests/` 目录约定 + 一个 smoke 测试 | T01, T02 | C | todo | — | — | `workspace/T06/` | `bootstrap-vitest` | `npm run test` 通过；smoke 用例验证 `import.meta` / 模块解析正确 | 首个验证：`npm run test` |
| T07 | Playwright e2e 最小骨架 | `playwright.e2e.config.ts` + 一个 "应用启动并渲染空面板" 测试；CI 默认不跑，仅手动 / nightly | T03, T06 | C | todo | — | — | `workspace/T07/` | `bootstrap-playwright-e2e` | `npm run test:e2e` 在本机能跑通空白窗口渲染断言 | 首个验证：`npm run test:e2e -- --project=electron` |
| T08 | pre-commit：simple-git-hooks + lint-staged | 仅在改动文件上跑 lint / format（不跑 typecheck，避免提交慢） | T04 | D | todo | — | — | `workspace/T08/` | `bootstrap-git-hooks` | `git commit` 触发 lint-staged，对 lint 异常文件能阻止提交 | 首个验证：人为造一个 lint 错误后 `git commit` 验证拦截 |
| T09 | CI workflow（lint / typecheck / test / build） | `.github/workflows/ci.yml`：node 22 + cache npm + concurrency cancel-in-progress；不挂 e2e | T01–T07 | — | todo | — | — | `workspace/T09/` | `bootstrap-ci-workflow` | push / pull_request 触发后 4 个 step 全绿；同 branch 新 commit 自动取消旧 run | 首个验证：CI 上游一次绿色 run |

### M2 · 核心契约冻结（P1，前置于任意 adapter）

| ID | Title | Goal | Depends On | Parallel | Status | Owner | Claimed At | Workspace | Change | Done When | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T10 | 主进程结构化日志 | `src/main/logging/appLogger.ts`：JSON 行 + level / module / session-id；落地到 `userData/logs/` | T03 | E | todo | — | — | `workspace/T10/` | `core-app-logger` | 单测覆盖 4 类 level；运行时日志写入 `userData/logs/app.log`，按日轮转 | 首个验证：`npm run test -- appLogger` |
| T11 | userData 目录约定 + Settings schema v1 | `app.getPath('userData')` 下分 `history/` `settings.yaml` `logs/` `cache/`；`shared/appSettings.ts` 带 `version: 1` 与 i18n 字段 | T03, T10 | E | todo | — | — | `workspace/T11/` | `core-settings-schema-v1` | 启动时若文件不存在自动写默认；schema 通过 yaml 解析 + 类型检查；带迁移入口（即便 v1 仅 noop） | 首个验证：`npm run test -- settingsService` |
| T12 | 持久化 schema 版本化骨架（history / sessions / quota） | 抽出最小持久化 store 接口 + `version` 字段 + 迁移函数挂点 | T11 | F | todo | — | — | `workspace/T12/` | `core-history-schema-v1` | 单测验证：写入带 `version`，读取时若版本过低走迁移；版本不识别时 fail-closed | 首个验证：`npm run test -- historyStore` |
| T13 | IPC 契约集中文件 + preload contextBridge 白名单 | `src/shared/messageTypes.ts` 集中通道名 / payload / error code；preload 仅暴露白名单 | T03 | F | todo | — | — | `workspace/T13/` | `core-ipc-contract` | 单测覆盖类型推导；renderer 走 `window.codepeek.*` 调用 round-trip 通过 | 首个验证：`npm run test -- ipcHub` |
| T14 | i18n 骨架（en / zh-CN） | `renderer/i18n.tsx`：跟随系统 locale + override；最小 key/value，无外部依赖 | T03, T11 | E | todo | — | — | `workspace/T14/` | `core-i18n-shell` | 渲染端在 en / zh-CN 间切换，未命中 key 走显式 fallback | 首个验证：`npm run test -- i18n` |

### M3 · 单 adapter MVP（Claude）

| ID | Title | Goal | Depends On | Parallel | Status | Owner | Claimed At | Workspace | Change | Done When | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T15 | Claude session watcher + normalizer + 面板渲染端到端闭环 | 监听本地 Claude 会话日志 → normalizer → session store → IPC → 面板渲染一个 session 卡片 | T10, T11, T12, T13, T14 | — | todo | — | — | `workspace/T15/` | `claude-session-mvp` | e2e：以 fixture 替身写入 Claude 日志，面板内 1 秒内出现状态变化；持久化恢复后状态保留 | 首个验证：`npm run test:e2e -- claude-mvp` |

### M4 · 第二 adapter（Codex）

| ID | Title | Goal | Depends On | Parallel | Status | Owner | Claimed At | Workspace | Change | Done When | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T16 | Codex session watcher + normalizer 复用 M2 契约 | 接入 Codex 终端日志，证明边界设计可扩展 | T15 | — | todo | — | — | `workspace/T16/` | `codex-session-mvp` | 同 fixture 场景下 Claude 与 Codex 两类 session 同时显示；normalizer 单测通过 | 首个验证：`npm run test:e2e -- codex-mvp` |

### M5 · macOS 发版闭环（P2）

| ID | Title | Goal | Depends On | Parallel | Status | Owner | Claimed At | Workspace | Change | Done When | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T17 | macOS 权限引导 + 诊断日志导出按钮 | 首次启动检查辅助功能 / 文件访问；UI 上的 "导出诊断日志" 按钮打包 logs + settings + 平台信息 | T10, T11, T14 | G | todo | — | — | `workspace/T17/` | `macos-perm-and-diag-export` | 缺权限时面板显示引导；点击导出按钮生成 zip 并落到下载目录 | 首个验证：`npm run test -- supportDiagnostics` |
| T18 | electron-builder.yml + entitlements（macOS arm64 zip + dmg） | `electron-builder.yml`：`appId` / `productName` / `files` / `asar` / `directories.output: release`；`build/entitlements*.plist` | T01, T03 | G | todo | — | — | `workspace/T18/` | `dist-electron-builder-mac` | `npm run dist:mac` 产出 `release/*.dmg` + `*.zip` + `*.blockmap` + `latest-mac.yml`；本地 `dist:mac:dir` 也能跑 | 首个验证：`npm run dist:mac:dir` |
| T19 | macOS 代码签名 + 公证 | env：`CSC_LINK` / `CSC_KEY_PASSWORD` / `APPLE_ID` / `APPLE_APP_SPECIFIC_PASSWORD` / `APPLE_TEAM_ID` | T18 | — | todo | — | — | `workspace/T19/` | `dist-macos-signing` | 已签名 dmg 通过 `spctl --assess --verbose` 与 Apple notary 验证；Hardened Runtime 开启 | 首个验证：`xcrun notarytool history` 出现新条目 |
| T20 | electron-updater 接 GitHub Releases | 主进程接入 `electron-updater`；feed 走 GitHub Releases | T19 | H | todo | — | — | `workspace/T20/` | `dist-auto-update` | 启动时检测到更新会拉取 zip + blockmap 并提示；签名校验失败时拒绝安装 | 首个验证：本地 stub feed 验证更新流程 |
| T21 | release.yml（tag v* 触发，仅 macOS runner） | tag → install → lint → typecheck → test → dist:mac → release-asset 校验 → softprops/action-gh-release 创建 draft | T18, T19 | H | todo | — | — | `workspace/T21/` | `dist-release-workflow` | tag 推送后 GitHub Releases 出现 draft，含 dmg / zip / blockmap / latest-mac.yml；release notes 引用 `docs/release-notes-<version>.md` | 首个验证：在 fork 上一次 dry-run tag |
| T22 | dependabot.yml（npm，每周） | `.github/dependabot.yml`：仅 npm 生态、weekly、limit 5 | — | I | todo | — | — | `workspace/T22/` | n/a | 仓库 Dependabot 第一次 dispatch 后能看到 PR | `n/a`：仓库元数据级 |
| T23 | 仓库协作元数据 | `SECURITY.md`（极简）+ `.github/ISSUE_TEMPLATE/bug-report.yml` + `.github/PULL_REQUEST_TEMPLATE.md`（极简） | — | I | todo | — | — | `workspace/T23/` | n/a | 在 GitHub UI 上 New Issue 走出表单；新 PR 默认带模板 | `n/a` |

### M6 · 文档与诊断收尾（P2 收口）

| ID | Title | Goal | Depends On | Parallel | Status | Owner | Claimed At | Workspace | Change | Done When | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T24 | 发版文档与图标占位 | 应用图标 / Tray 图标占位资源；`docs/release-notes-<version>.md` 模板；最小 privacy 声明 + troubleshooting；`openspec/changes/` 与 `docs/plans/` 骨架 README | T18, T21 | — | todo | — | — | `workspace/T24/` | `release-docs-and-assets` | release.yml 引用 release notes 不再 404；首屏图标显示占位资源；privacy 与 troubleshooting 各为单文件 | 首个验证：本地一次 `npm run dist:mac` + 图标可见 |

## Claiming Rules

1. 任意任务在 `claimed` 之前，**必须**先按 `using-git-worktrees` 建立隔离 worktree，并完成基线校验。
2. 在该 worktree 中：
   - 把任务行 `Status` 从 `todo` 更新为 `claimed`，并填 `Owner`、`Claimed At`（ISO-8601，UTC）。
   - 创建 `workspace/<task-id>/` 目录。
   - 把 OpenSpec change 名（若有）写回 `Change` 列。
   - 在 `Change Log` 追加一条认领记录。
3. 选择 "下一个任务" 时，优先认领 `status=todo` 且 `Depends On` 全部 `done`、且解锁后续工作量最大的任务。
4. 一个任务只能对应一个 workspace 目录与一个 OpenSpec change（`n/a` 除外）。
5. `claimed` 后的任何工作（research / spec / implementing / verifying / review）必须继续在隔离 worktree 中进行；禁止回到共享主工作区推进同一任务。
6. 状态从 `done` 之外推进到下一格前，先在 `Change Log` 留痕，然后再修改 `Status` 列。
7. 如果证据冲突（如 task 文档显示 `done` 但 OpenSpec change 未归档），按源设计 → OpenSpec → workspace 产物 → 当前 task 文档文本的优先级回退到 "证据支持的最高状态"。

## Change Log

- `2026-05-07T00:00:00Z` · init · 由 `deriving-task-board-from-design` 从 `bootstrap-engineering-design.md` 推导生成 24 条任务（覆盖 M1–M6 / P0–P2），全部状态 `todo`。
- `2026-05-07T00:00:00Z` · scope · 显式将源设计 P3 项（Windows / Linux 打包、e2e PR gate、覆盖率阈值化、Sentry / 遥测、EV 签名、commitlint / husky / CONTRIBUTING / CODEOWNERS、第三方 notice 自动化、完整中文文档、a11y 高对比 / reduced-motion）排除出本任务板，待真实诉求触发后再单独立任务。
