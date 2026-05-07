# T01 · 遗留与备忘

## 流程偏离记录

1. **子 skill 缺失，使用替代评审**
   - `delivering-task-end-to-end` 要求 `requesting-code-review` / `receiving-code-review` / `verification-before-completion` / `using-git-worktrees` / `test-driven-development` 等子 skill。
   - 当前 `.trae/skills/` 仅部署 8 个 OpenSpec / planning skill，**未部署上述子 skill**。
   - 替代做法：`final_impl.md` 内做"自测三视角 rubric"；worktree 直接用原生 git；无单测 TDD（T01 不写运行时代码，无测试可写）。
   - 后续若部署完整 skill 集，应回归原工作流。

2. **TDD 不适用**
   - T01 落地 `package.json` 与 lockfile，**不存在可单测的运行时代码**。验证靠 `npm run` + 命令存在性 dry run（npm 11 下 `npm run -s` 会静默该列表，故统一使用 `npm run`；等价静态命令为 `node -e "console.log(Object.keys(require('./package.json').scripts).join('\n'))"`）。
   - `test_strategy.md` 会显式记录这一点。

## 后续任务清理点

1. **T18（electron-builder.yml）落地后**：将 `dist:mac:dir` 的 `-c.mac.identity=null -c.mac.notarize=false` CLI 传参改为 `electron-builder.yml` 中的 mac block 配置（per-target 或 environment-aware），同时 `dist:mac:dir` 简化为 `electron-builder --mac dir --publish never`。
2. **T07（Playwright）落地后**：评估是否可删除冗余的 `playwright` devDependency（仅保留 `@playwright/test`）。
3. **T03（electron-vite shell）落地前**：重跑 `npm install`（不带 `ELECTRON_SKIP_BINARY_DOWNLOAD=1 / --ignore-scripts`）以下载 Electron macOS arm64 运行时二进制；T01 因网络 ECONNRESET 跳过了这一步，但未违反 T01 边界（命令存在性 ≠ 运行时二进制就绪）。
4. **T20（electron-updater）落地后**：补 `dependencies.electron-updater`。
5. **README** 在 M1 末期加"Scripts" 表格章节（不在 T01 范围）。
6. **T05** 负责扩充 `.gitignore`（加 `out/` / `release/` / `playwright-report/` / `.env*`），T01 只在文件中加了必须的 `node_modules/`。

## 已知风险（需在 final compare 阶段复核）

- 网络受限环境下 `npm install` 可能失败；脚本契约本身不依赖 install 成功，但 lockfile 不生成会推到 todo。
- `electron@^41` 与 `typescript@^5.6` / `vitest@^3` 的兼容性需 T02–T07 各自验证；T01 仅装占位，如各任务发现版本冲突需在该任务内升降版。
- `engines.node >=22` 在 CI（actions/setup-node@v4）上由 `.nvmrc`（T05）确定具体小版本。
