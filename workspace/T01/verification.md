# T01 · 验证证据（verifying 阶段产出）

> 环境：macOS arm64；Node v25.9.0；npm 11.12.1；worktree `../CodePeek-T01`；分支 `task/T01-bootstrap-package-scripts`；执行时间 2026-05-07。

## V0 · package.json 字段合规

```
$ node -e "const p=require('./package.json'); console.log(p.name,p.private,p.type,p.main,p.engines.node)"
codepeek true module out/main/main.js >=22
```

对应 spec `Requirement: 仓库根 package.json 存在` 与 `Requirement: engines.node 锁定 >=22`：**PASS**。

## V1 · npm run 列全部脚本（task 板首验手段）

```
$ npm run
Lifecycle scripts included in codepeek@0.0.0:
  test
    vitest run
available via `npm run`:
  dev             electron-vite dev
  build           electron-vite build
  preview         electron-vite preview
  lint            eslint .
  typecheck       tsc -b
  format          prettier --write .
  format:check    prettier --check .
  test:watch      vitest
  test:e2e        npm run build && playwright test -c playwright.e2e.config.ts
  dist:mac        npm run build && electron-builder --mac zip dmg --publish never
  dist:mac:dir    npm run build && electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false
```

- 9 条核心脚本（`dev/build/test/test:e2e/lint/typecheck/format/dist:mac/dist:mac:dir`）全部存在。
- 3 条辅助脚本（`preview/format:check/test:watch`）全部存在。

对应 spec `Requirement: 核心脚本契约` 与 `Requirement: 辅助脚本`：**PASS**。

> 注：`npm run -s` 在 npm 11 下静默是预期行为；task 板上的首验提法使用 `-s` 是习惯，核心语义等同 `npm run`。已显式用 `npm run` 获取完整清单。

## V2 · 脚本语义深等

对 9 条核心脚本做 `package.json.scripts[name] === expected` 逐项比对：

```
OK dev            = electron-vite dev
OK build          = electron-vite build
OK test           = vitest run
OK test:e2e       = npm run build && playwright test -c playwright.e2e.config.ts
OK lint           = eslint .
OK typecheck      = tsc -b
OK format         = prettier --write .
OK dist:mac       = npm run build && electron-builder --mac zip dmg --publish never
OK dist:mac:dir   = npm run build && electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false
```

对应 spec `Requirement: 核心脚本契约 · Scenario: 脚本命令语义与契约一致`：**PASS**。

## V3 · devDependencies 覆盖

脚本契约只要求 8 条脚本二进制必须被 devDependencies 覆盖；下面 8 条对应 spec 的「命令二进制可被 npm 解析」：

```
OK electron          ^41.1.0
OK electron-vite     ^5.0.0
OK electron-builder  ^26.0.0
OK eslint            ^9.0.0
OK prettier          ^3.0.0
OK typescript        ^5.6.0
OK vitest            ^3.0.0
OK @playwright/test  ^1.58.2
```

`package.json.devDependencies` 实际共声明 9 项：上表 8 项 + 冗余的 `playwright ^1.58.2`。后者是对 CodePal 参考实现的保守对齐（详见 `reference_impl.md` 与 `todo.md` 中 T07 收敛点），不在脚本契约要求内，故 V3 单独列出而非计入 8 项必集。

对应 spec `Requirement: 命令二进制可被 npm 解析 · Scenario: devDependencies 覆盖所有核心脚本二进制`：**PASS**。

## V4 · lockfile 与 package.json 自洽

```
$ npm ls --depth=0
codepeek@0.0.0 /Users/yousa/Documents/Github/CodePeek-T01
├── @playwright/test@1.59.1
├── electron-builder@26.8.1
├── electron-vite@5.0.0
├── electron@41.5.0
├── eslint@9.39.4
├── playwright@1.59.1
├── prettier@3.8.3
├── typescript@5.9.3
└── vitest@3.2.4
```

退出码 0。对应 spec `Requirement: package-lock.json 存在 · Scenario: package-lock.json 与 package.json 同步`：**PASS**。

> 注：`npm install` 以 `ELECTRON_SKIP_BINARY_DOWNLOAD=1 --ignore-scripts` 执行，以回避 Electron 二进制 postinstall 的 ECONNRESET 网络瑕疵。Electron 包体本身已装入 `node_modules/electron/`，只是 macOS 运行时二进制未下载。这对 T01 的"命令存在性"边界无影响；当 T03 需要真机 `npm run dev` 时会重新下载。已记录到 `workspace/T01/todo.md`。

## V5 · 4 个关键脚本命令存在性（Done When）

对每个脚本分别运行 `npm run <name>` 并检查输出中是否出现 `missing script`：

| script | `npm run` 观察到 `missing script`？ | 工具退出码 | 失败原因（非 T01 范围） |
| --- | --- | --- | --- |
| `lint` | 否 | 2 | `eslint.config.js` 不存在 → T04 |
| `typecheck` | 否 | 1 | `tsconfig.json` 不存在 → T02 |
| `test` | 否 | 1 | 无匹配测试文件 → T06 |
| `build` | 否 | 1 | `electron.vite.config.*` 不存在 → T03 |

对应 spec `Requirement: 命令二进制可被 npm 解析 · Scenario: 关键脚本的 "--help" / "--version" 不报 "missing script"`：**PASS**。

同样覆盖 task 板 `Done When`：_"`npm run lint` / `typecheck` / `test` / `build` 四个命令在 M1 完成后能被无错调用（命令存在性即可）"_。

## V6 · OpenSpec 自身校验

```
$ openspec validate bootstrap-package-scripts --strict
Change 'bootstrap-package-scripts' is valid
```

对应 `test_strategy.md` 中"OpenSpec change 自身合法"：**PASS**。

## V7 · 显式跳过项

| 项 | 状态 | 理由 |
| --- | --- | --- |
| Vitest 单测 | SKIP | T01 无运行时代码 |
| Playwright e2e | SKIP | T01 无 UI / 入口 |
| `npm run lint` 绿 | SKIP | T04 范围 |
| `npm run typecheck` 绿 | SKIP | T02 范围 |
| `npm run build` 绿 | SKIP | T03 / T18 范围 |
| `npm run dev` 真机冒烟 | SKIP | T03 Goal；T01 无 Electron 入口 |
| `npm run dist:mac` 绿 | SKIP | T18 / T19 范围 |

与 `test_strategy.md` 一致。

## 结论

- Spec 中 5 个 Requirement / 8 个 Scenario 全部 PASS。
- task 板 T01 `Done When` 两项全部满足：
  - ✅ 仓库根存在 `package.json`
  - ✅ `npm run lint/typecheck/test/build` 四个命令能被无错调用（命令存在性维度）
- 首验 `npm run` 列出全部目标脚本：通过（npm 11 下 `npm run -s` 会静默该列表，故实际使用不带 `-s` 的 `npm run`；等价静态命令为 `node -e "console.log(Object.keys(require('./package.json').scripts).join('\n'))"`）。

**验证结论**：T01 满足验收阈值，可以进入 review 阶段。
