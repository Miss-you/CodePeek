# CodePeek Bootstrap Engineering Design

> 适用范围：CodePeek 仓库从 **空脚手架** 走向 **可发布桌面应用** 的工程化基础设施清单。
>
> 本文档分两部分：
>
> - **第一部分**：完整工程化清单（参考 CodePal 现有工程实践 + AI 代理工作流契约推导）。
> - **第二部分**：从 **个人项目 + 长期可维护性 + ROI** 视角做的审核与删减结论，以及最终保留的最小集和分期落地建议。

---

## 0. 现状速查

- 仓库目前只含 `README.md`、`AGENTS.md`、`CLAUDE.md`、`LICENSE`，以及 `.claude/.codex/.trae/openspec` 下的 skill / 配置骨架。
- `package.json`、`tsconfig.json`、Electron 入口、构建配置、CI、打包脚本均尚未落盘。
- `openspec/config.yaml` 已就位，意味着变更流程默认走 spec-driven + OpenSpec change。
- `AGENTS.md` 已经把 "提交前确认 lint / type-check / 相关测试" 写成硬约束。
- 各 agent 目录下已经存在 `compatibility-first-planning` / `deriving-task-board-from-design` / `delivering-task-end-to-end` / `monitoring-pr-ai-reviews` 等 skill，但这些 skill 在第一步就会调用 `npm run lint`、`npm run test:e2e` 等命令——目前仓库尚未提供这些命令，是工程化补齐的第一个 trigger。
- 参考实现 [shamcleren/CodePal](https://github.com/shamcleren/CodePal) 已经具备 Electron + electron-vite + electron-builder + Vitest + Playwright + GitHub Actions CI/Release 完整链路，其工程结构是本清单的主要参照。

---

## 第一部分 · 完整工程化清单

### 1. 基础工程脚手架（先于一切的"最小可复现"）

| 条目 | 说明 | 为什么现在就要 |
|---|---|---|
| `package.json` + `npm scripts` 契约 | 对齐 `dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:win` | skill 已经把这些命令当成"事实"。脚本名漂移会让所有 AI 执行链路在第一步失败。 |
| `electron-vite` 主/渲染/preload 三进程入口 | `src/main/main.ts`、`src/preload/index.ts`、`src/renderer/main.tsx` | Electron 三进程模型是 IPC / 持久化 / 面板渲染边界的物理体现。不先定，"冻结 IPC 契约"无处落点。 |
| `tsconfig.json` 分 project（main / preload / renderer / shared） | 三套 lib / target 不一样（Node vs DOM vs sandbox） | 不分 project 会让 `tsc --noEmit` 把 renderer 当 Node 编译，CI typecheck 不可信。 |
| `.editorconfig` | 行尾、缩进、字符集统一 | macOS / Windows 换行差异在 Electron 项目中常见，签名后的 diff 会被污染。 |
| `.nvmrc` 或 `"engines": { "node": ">=22" }` | 锁 Node 版本（与 CodePal 对齐） | 本地与 CI 的 Node / npm 版本不一致是 electron-builder 失败最常见的原因。 |

### 2. 代码质量：Lint / Format / Typecheck

| 条目 | 说明 | 为什么 |
|---|---|---|
| `eslint.config.js`（flat config） | `@eslint/js` + `typescript-eslint` + `eslint-plugin-react` + `eslint-plugin-react-hooks` + `eslint-plugin-import` | AGENTS.md 默认 lint 必跑；规则模板与 CodePal 对齐避免漂移。 |
| `prettier` + `eslint-config-prettier` | format 与 lint 冲突的避让 | 否则 `npm run lint` 与 `npm run format` 会互相踩脚。 |
| `npm run typecheck` 独立命令 | `tsc -b` 对三套 tsconfig 分别跑 | skill 把 typecheck 当独立 gate，不能仅靠 build 间接检查。 |
| `lint-staged` + `husky` 或 `simple-git-hooks` | pre-commit 跑改动文件的 lint / typecheck | AI 代理工作流最容易漏 "提交前检查"，hook 可仓库级强制。 |
| `commitlint` | 配合 AGENTS.md 的 "祈使句、英文小写动词开头" | 已写约束就要机械化执行。 |

### 3. 测试策略

| 条目 | 说明 | 为什么 |
|---|---|---|
| `vitest.config.ts` + `tests/` 目录 | 单元测试环境（node vs jsdom）按包分开 | 主进程 store 与 React 组件混跑会污染 `window` / `fs`。 |
| Playwright e2e 驱动 Electron | 面板 UI / IPC / hook 端到端 | 监控 Claude / Codex 终端会话的核心链路只有 e2e 能证明"面板渲染正常"。 |
| `tests/fixtures/{claude,codex}/` 真实载荷 | adapter 正确性靠 replay 真实 log / hook payload，而不是 mock | CodePal 已证明这是 normalizer 不退化的唯一办法。 |
| 覆盖率阈值（可选） | session/history store 等 "单源真相" 模块设阈值 | 防回归，UI 层不强求。 |
| e2e CI 稳定性策略 | retry、artifact 上传、trace 保留 | Electron e2e 在 CI 极易 flake。 |

### 4. CI / CD

| 条目 | 说明 | 为什么 |
|---|---|---|
| `.github/workflows/ci.yml` | install → lint → typecheck → test → build | CodePal 模板可直接对齐；建议显式加 typecheck step。 |
| `.github/workflows/e2e.yml` | 单独工作流，nightly 或 label 触发 | 主 CI 要求 "绿得快"，避免被 Electron e2e 拖慢。 |
| `.github/workflows/release.yml` | tag `v*` 触发 mac + win 双 runner 打包并发布 draft | 见第 5 节。 |
| `concurrency: cancel-in-progress` | 同 branch 新 commit 取消旧 run | 节省 CI 配额、减少 flake 干扰。 |
| 依赖缓存 | `actions/setup-node@v4` 的 `cache: npm` | 启动时间从 3min → 40s。 |
| Fork PR 的 secret 隔离 | release secrets 仅在 tag push 提供 | 防签名私钥泄露。 |
| `dependabot.yml` 或 `renovate.json` | Electron 与 electron-builder 升级节奏快 | security patch 是 release 门槛。 |

### 5. 桌面打包与发布

#### 5.1 通用
| 条目 | 说明 | 为什么 |
|---|---|---|
| `electron-builder.yml` | `appId` / `productName` / `files` / `asar` / `publish: github` | 默认值几乎都不对，必须显式配。 |
| `directories.output: release` | 输出目录固定 | release.yml 的产物断言依赖此路径。 |
| `afterAllArtifactBuild` 脚本 | 产物校验、命名标准化、生成 checksum | release verify 的锚点。 |
| Auto-update 通道 | `electron-updater` + GitHub Releases | 没有 updater = 用户手动重装。 |

#### 5.2 macOS
| 条目 | 说明 | 为什么 |
|---|---|---|
| `build/entitlements.mac.plist` + `entitlements.mac.inherit.plist` | Hardened Runtime / JIT / network / files | Apple Silicon 不加会直接崩。 |
| 签名 & 公证 | `CSC_LINK` / `APPLE_ID` / `APPLE_APP_SPECIFIC_PASSWORD` / `APPLE_TEAM_ID` | 不公证用户打开会看 "无法验证开发者"。 |
| `arm64` 与 `x64` 目标策略 | v1 先 arm64 | CodePal 也是只做 arm64。 |
| `zip` + `dmg` 双产物 | updater 需要 zip；分发用 dmg | 只做 dmg 会让 updater 失效。 |
| 本地 `dist:mac:dir` / `dist:mac:unsigned:test` | 不签名跑一遍 | 排除签名干扰用于排错。 |

#### 5.3 Windows
| 条目 | 说明 | 为什么 |
|---|---|---|
| `nsis` target + per-user / per-machine 策略 | 默认 per-user 更安全 | per-machine 会被企业杀软拦。 |
| EV 代码签名（可延后但留 env） | `WIN_CSC_LINK` / `WIN_CSC_KEY_PASSWORD` | 没签名会触发 SmartScreen 红屏。 |
| `appx` / `portable` 不做 | v1 缩范围 | — |
| Windows runner 不能公证，仅签名 | release.yml 拆 mac / win 两 runner | 构建环境不可合并。 |
| 路径与行尾兼容 | watcher / hook 处理 `\r\n`、`C:\...` | 监控 Claude / Codex 日志会踩。 |

#### 5.4 Linux
- v1 显式不做。

### 6. 运行时工程化（与"监控 Claude / Codex 终端会话"强相关）

| 条目 | 说明 | 为什么 |
|---|---|---|
| 主进程结构化日志（`appLogger.ts`） | 带 level / module / session-id 的 JSON 行日志 | 99% 的 watcher / hook 问题靠主进程日志定位。 |
| 用户数据目录约定 | `app.getPath('userData')` 下分 `history/`、`settings.yaml`、`logs/`、`cache/` | v1 发出去后 schema 再动就要写迁移。 |
| Settings schema + 版本号 + 迁移入口 | `shared/appSettings.ts` 带 `version: 1` | 没有 version，老用户数据只能猜。 |
| 持久化 schema 版本化与迁移 | history / sessions / quota | `compatibility-first-planning` 列为必冻结契约。 |
| IPC 契约文件 | 通道名 / payload / error code 集中在 `src/shared/messageTypes.ts` | 散在各处就是 adapter 漏到 core。 |
| Crash reporter | Electron `crashReporter` 或 Sentry | 桌面用户反馈极少，崩溃要能自抓。 |
| 首次启动 / 权限申请流程（macOS） | 文档 + UI 提示 + 诊断按钮 | 监控会话需要读 `~/.claude/…` / `~/.codex/…`，权限缺失要能引导。 |
| i18n 骨架（en / zh-CN） | `renderer/i18n.tsx` | README 已承诺双语，不先搭就要重做。 |
| 面板可访问性基线 | 键盘、焦点、reduced-motion、高对比 | 企业环境会因 a11y 缺失禁用。 |

### 7. 安全与合规

| 条目 | 为什么 |
|---|---|
| `contextIsolation: true` / `sandbox: true` / `nodeIntegration: false` | Electron 默认安全姿势。 |
| `contextBridge` 白名单 API | 防 "为方便" 直接挂 `window` 的调试代码外泄。 |
| `electron-updater` 走 HTTPS + 签名校验 | 否则 update feed 等于给中间人 RCE。 |
| 数据最小化 + 明确不上传策略 | 监控类产品不写明白用户不敢装。 |
| `LICENSE` / `THIRD_PARTY_NOTICES.md` | Apache-2.0 已在，依赖 notice 要补。 |
| Privacy 文档（中英） | 桌面监控产品几乎必备。 |

### 8. 仓库级协作基础设施

| 条目 | 为什么 |
|---|---|
| `.github/ISSUE_TEMPLATE/{bug,feature}.yml` + `config.yml` | 没模板的 issue 全是 "打不开了" 一句话。 |
| `.github/PULL_REQUEST_TEMPLATE.md` | 对齐 AGENTS.md 提交规范。 |
| `CODEOWNERS` | 即使一人项目，AI 代理开 PR 也需要默认 reviewer。 |
| `CONTRIBUTING.md` | 把 "先 openspec-propose 再实现" 工作流外化。 |
| `SECURITY.md` | 桌面监控产品安全披露通道必须有。 |
| `CHANGELOG.md` 或 `docs/release-notes-*.md` | release.yml 引用为 release body。 |
| `.gitattributes` | 锁定二进制资产行尾、防 `icon.icns` 被当文本 merge。 |
| `.gitignore` | `out/` / `release/` / `node_modules/` / 用户数据 / `.env*` / `playwright-report/`。 |

### 9. 可观测性与用户反馈

| 条目 | 为什么 |
|---|---|
| 产品内 "诊断日志导出" 按钮 | 用户自助上报比远程猜强 100 倍（CodePal 有 `supportDiagnostics.ts`）。 |
| 版本 / Electron / OS 信息展示 | 收 bug 第一件事就问这个。 |
| 可选遥测（opt-in，匿名） | 启动 / 崩溃 / 活跃 adapter；视隐私承诺决定是否做。 |

### 10. 文档工程

| 条目 | 为什么 |
|---|---|
| `docs/plans/` 目录 + design / design-task 模板 | skill 已假设其存在，需先落空骨架。 |
| `openspec/changes/` 骨架 | 同理。 |
| `docs/release-notes-<version>.md` 模板 | release.yml `body_path` 引用此路径。 |
| `docs/support-scope.md`（中英）+ `docs/troubleshooting.md`（中英） | 桌面产品必备："能用 / 不能用 / 不会修" 三类划线。 |
| 各 agent 入口文档（CLAUDE.md / CODEX.md / TRAE.md） | 三个 skill 目录已存在，入口文档要对齐。 |

---

## 第二部分 · 个人项目 / 长期可维护性 / ROI 视角的审核与删减

### 审核原则

1. **可维护性优先于完整度**：个人项目最大的成本是"心智负担"，每多一项基础设施就多一个未来要维护的东西。
2. **延后到有真实需求再做**：贡献者模板、CODEOWNERS、commitlint 这类协作设施在没有协作者前就是空配置文件。
3. **让 AI 代理可执行 = 必做**：`lint` / `typecheck` / `test` / `build` 这四个命令不到位，所有 skill 都跑不起来，ROI = 无穷大。
4. **能用钱 / 后续补的延后**：EV 代码签名、遥测、Sentry 这种花钱或第三方服务的事项，先留口子但不做。
5. **冻结类（schema / IPC / userData）必须前置**：否则 v1 一发出去就是数据迁移噩梦。
6. **跨平台诱惑要克制**：Windows 一旦做就要一直维护。个人项目先 macOS arm64，再决定是否扩 Windows。

### 删减明细

| 原条目 | 决策 | 理由 |
|---|---|---|
| `commitlint` | **去掉** | 一人项目，AGENTS.md 的祈使句规范靠自觉即可，hook 失败更常烦人。 |
| `husky` 全套 | **降级**：用 `simple-git-hooks` 起一个最小 pre-commit 跑 `lint-staged` | husky 自身就是另一套要维护的工具。 |
| `lint-staged` | **保留**：只跑 lint / format on changed files | typecheck 放 CI，pre-commit 不跑 typecheck，避免提交时等几十秒。 |
| `dependabot.yml` / Renovate | **保留 dependabot.yml 最小配置（仅 npm，每周一次）** | 个人项目跟不上 Electron security patch 是真实风险。Renovate 太重不要。 |
| Crash reporter（Sentry） | **去掉**，改为本地写崩溃 dump 到 `userData/logs/` | 不引入外部 SaaS，"诊断日志导出" 已能覆盖个人项目场景。 |
| 可选遥测 | **去掉**，写在 privacy 里 "v1 不收集任何数据" | 简化合规叙事，节省维护成本。 |
| EV 代码签名（Windows） | **延后**，仅留 env 占位 | 证书贵 + 个人难买；先发未签名，文档里写 SmartScreen 提示即可。 |
| Windows 全套打包 | **降级到 P2 阶段**，v1 先只发 macOS arm64 | 打包矩阵每多一个平台都是长期成本。明确 v1 = macOS-only 是个人项目最 ROI 的取舍。 |
| `appx` / `portable` Windows 目标 | 已不做 | — |
| Linux 打包 | 已不做 | — |
| `THIRD_PARTY_NOTICES.md` 自动生成 | **保留为脚本但延后跑**：v1 前手写一份简表 | 自动生成工具（如 license-checker）后续再接。 |
| `CONTRIBUTING.md` | **去掉** | 一人项目，AGENTS.md + CLAUDE.md 已经覆盖。 |
| `CODEOWNERS` | **去掉** | 一人项目无意义。 |
| `.github/ISSUE_TEMPLATE/*` | **保留 1 个 bug-report.yml** | feature-request 个人项目可以省。 |
| PR 模板 | **保留极简版（< 10 行）** | 帮助 AI 代理生成结构化 PR 描述就够了。 |
| `SECURITY.md` | **保留极简版** | 任何"监控 + 桌面" 产品都需要披露通道，5 行即可。 |
| 双语文档（privacy / support-scope / troubleshooting 全套中英） | **降级**：v1 只产出一份双语 README + 单语 privacy + 单文件 troubleshooting | CodePal 那一套是 v1.x 累积下来的，现在做太重。i18n 仍然在 UI 内做，文档先英为主中文翻译延后。 |
| 覆盖率阈值 | **去掉**作为 CI gate | 个人项目下 coverage 阈值往往逼自己写无意义测试。保留 vitest --coverage 命令但不阻塞。 |
| `e2e.yml` 单独 workflow | **保留但仅 nightly + 手动触发** | 主 CI 不挂 e2e，避免 PR 阻塞。 |
| Auto-update | **保留**，但 v1 先打通 GitHub Releases 通道 | 桌面应用缺 updater 必须手动重装，长期维护损失最大。 |
| Settings / 持久化 schema 版本化 | **保留**（必须前置） | 一旦有真实用户，schema 不带 version 就是技术债无法回头。 |
| IPC 契约集中文件 | **保留**（必须前置） | 散落式 IPC 是 CodePeek 这类多 adapter 项目的最大坑。 |
| 主进程结构化日志 | **保留** | watcher / hook 排错离不开。 |
| 诊断日志导出按钮 | **保留** | 替代 Sentry，零成本且对个人项目最实用。 |
| 面板 a11y 基线 | **降级到 P2**：仅守键盘可达 + 焦点可见 | 高对比 / reduced-motion 等可在有真实反馈后再做。 |
| 权限引导 UX | **保留**（macOS 必须） | 不引导=用户装完打不开是 release blocker。 |
| `tsconfig` 分 project | **保留** | 不分一定踩 typecheck 假阳性。 |
| `.editorconfig` / `.gitattributes` | **保留** | 极低维护成本，规避难调试问题。 |
| `.nvmrc` | **保留** | 与 CI 对齐成本 = 0。 |
| Tray 图标 / 应用图标 / hero 图 | **保留占位**，v1 用临时素材 | 不做就 dist 不出来；不必精修。 |

### 最终保留清单（个人项目 ROI 优先版）

按 **必须先做（P0）** / **核心闭环前补齐（P1）** / **发版前补齐（P2）** / **延后到真实诉求触发（P3）** 分层。

#### P0 · 让 AI 代理与本人都能跑通的最小可执行集
1. `package.json` + 脚本契约：`dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:mac:dir`。
2. 三套 `tsconfig`（main / preload / renderer + 共享 base）。
3. `electron-vite` 主/渲染/preload 入口（含 `contextIsolation` / `sandbox` / `nodeIntegration: false`）。
4. ESLint flat config + Prettier + `eslint-config-prettier`。
5. `.editorconfig` / `.gitattributes` / `.gitignore` / `.nvmrc`。
6. Vitest 最小骨架 + 一个 smoke 单测。
7. Playwright e2e 最小骨架 + 一个 "应用能启动并渲染空面板" 测试（CI 中默认不跑，仅 nightly / 手动）。
8. `simple-git-hooks` + `lint-staged`（pre-commit 仅跑 changed file lint / format）。
9. `.github/workflows/ci.yml`：install → lint → typecheck → test → build。

#### P1 · 核心链路（监控 Claude / Codex 终端会话）落地前必须冻结
10. 主进程结构化日志（`src/main/logging/appLogger.ts`）。
11. userData 目录约定 + Settings schema v1（带 `version` 字段）。
12. 持久化 schema 版本化骨架（history / session / quota），含迁移入口约定。
13. IPC 契约集中文件（`src/shared/messageTypes.ts`）+ preload `contextBridge` 白名单。
14. i18n 骨架（en / zh-CN，简单的 key/value，无外部库依赖也行）。
15. macOS 权限引导（辅助功能 / 文件访问）+ 产品内 "导出诊断日志" 按钮。

#### P2 · 第一次发版前补齐
16. `electron-builder.yml` + `build/entitlements*.plist`（macOS arm64 only：zip + dmg）。
17. macOS 签名 + 公证（`CSC_LINK` / `APPLE_*` env）。
18. `electron-updater` 接 GitHub Releases。
19. `.github/workflows/release.yml`（tag 触发，仅 macOS runner）。
20. 极简版 `SECURITY.md` + 一份 privacy 声明 + `docs/release-notes-<version>.md` 模板。
21. `.github/ISSUE_TEMPLATE/bug-report.yml` + 极简 PR 模板。
22. `dependabot.yml`（仅 npm，每周）。
23. 应用图标 / Tray 图标占位资源。
24. `docs/plans/` 与 `openspec/changes/` 目录骨架（让 skill 工作流有落点）。

#### P3 · 延后到真实诉求触发再做
- Windows 打包（NSIS、签名、windows-latest runner）。
- Linux 打包。
- `e2e.yml` 单独 workflow 接 PR。
- 覆盖率阈值化。
- `THIRD_PARTY_NOTICES.md` 自动化。
- 中文版完整文档（troubleshooting / support-scope / privacy zh-CN）。
- a11y 高对比 / reduced-motion。
- Sentry / 遥测。
- EV 代码签名 / Windows SmartScreen 处理。
- `commitlint` / `husky` 完整套件 / `CONTRIBUTING.md` / `CODEOWNERS`（仅在出现协作者时再做）。

### 工程化推进路径建议

把上面的 P0–P2 拆成 6 个里程碑，便于 `deriving-task-board-from-design` 后续推导任务板：

1. **M1 · 脚手架可跑**：P0 第 1–9 项；标志 = `npm run lint && npm run typecheck && npm run test && npm run build` 全绿。
2. **M2 · 核心契约冻结**：P1 第 10–13 项；标志 = IPC / settings / 持久化 schema 文件存在 + 单测覆盖。
3. **M3 · 单 adapter MVP（Claude）**：在 M2 之上完成 Claude 终端会话监控的端到端闭环。
4. **M4 · 第二 adapter（Codex）**：复用 M2 契约，证明边界设计可扩展。
5. **M5 · macOS 发版闭环**：P2 第 16–22 项；标志 = 一次成功的 tag 触发 release，可装、可自动更新。
6. **M6 · 文档与诊断收尾**：P2 第 20、23、24 项；标志 = 用户拿到 dmg + 看到 release notes + 能自助导出诊断包。

P3 项不进入里程碑，作为"未来可能"放在文档尾部备忘即可。

---

## 附录 · 与现有 skill / OpenSpec 流程的衔接

- 本文档承担的是 `compatibility-first-planning` 的 **设计输出**，对应 `docs/plans/<topic>-design.md` 角色。
- 下一步：使用 `deriving-task-board-from-design` 把 P0–P2 的最终保留清单落成 `docs/plans/bootstrap-engineering-design-task.md`，并用稳定 ID（如 `T01–T24`）锁定。
- 单个里程碑下的 OpenSpec change 通过 `openspec-propose` 创建；执行用 `delivering-task-end-to-end`；归档用 `openspec-archive-change`。
- AGENTS.md 中关于 "提交前 lint / typecheck / 测试" 的硬约束会在 M1 完成的瞬间自动生效，不再是空头承诺。
