# T01 · Self-Review 记录

因 `.trae/skills/` 未部署 `requesting-code-review` / `receiving-code-review`，本阶段采用自查 + 分流的方式，扮演 3 个独立审查视角逐一通读仓库差异。

## 待审文件集合（本次变更）

```
modified:  .gitignore                                      (+1 行: node_modules/)
modified:  docs/plans/bootstrap-engineering-design-task.md (T01 状态轮转 + Change Log)
new:       package.json
new:       package-lock.json                                (npm install 生成)
new:       openspec/changes/bootstrap-package-scripts/
            ├── proposal.md
            ├── design.md
            ├── tasks.md
            └── specs/package-scripts-contract/spec.md
new:       workspace/T01/
            ├── current_state.md
            ├── reference_impl.md
            ├── final_impl_v1.md
            ├── final_impl.md
            ├── test_strategy.md
            ├── verification.md
            └── todo.md
```

`node_modules/` 已被 `.gitignore` 忽略，不会被 commit。

## 视角 A · 契约消费者视角（AI skill / 未来任务）

- ✅ 9 条核心脚本按任务板精确定义，命令语义已通过深等比对。
- ✅ `typecheck` 用 `tsc -b`，与 T02 的 project references 直接对齐；无需后续改名。
- ✅ `test:e2e` 前置 `npm run build`，Playwright 起 Electron 时 `out/` 可用。
- ✅ `engines.node>=22` 与 CI / `.nvmrc`（T05）对齐前置。
- 🟡 `dist:mac:dir` 在 CLI 带 `-c.mac.identity=null -c.mac.notarize=false`，T18 落 yml 后可简化。已入 `todo.md`。
- ❌ 无发现 bug / regression 级问题。

## 视角 B · 可维护性 / 边界视角

- ✅ T01 未越界：没有实现 Electron / tsconfig / eslint 配置；`src/` 仍然不存在。
- ✅ `.gitignore` 仅加 `node_modules/`（T01 本任务 `npm install` 必然产生），剩余条目 T05 负责。最小侵入。
- ✅ `package-lock.json` 随 package.json 自洽（`npm ls --depth=0` 通过）。
- 🟡 `playwright` 与 `@playwright/test` 同装是 CodePal 对齐的保守选择；T07 可再收敛。已入 `todo.md`。
- 🟡 Electron 平台二进制未下载（`ELECTRON_SKIP_BINARY_DOWNLOAD=1`）。对 T01 边界无影响；T03 真机冒烟前需重新 install。已入 `todo.md`。
- ❌ 无发现 bug / regression 级问题。

## 视角 C · 参考忠实度 / ROI 视角

- ✅ `dev` / `build` / `preview` / `test` / `test:e2e` / `dist:mac` / `dist:mac:dir` 与 CodePal 语义一致。
- ✅ 显式偏离已在 `reference_impl.md` / `design.md` 中给出理由（新增 `typecheck` / `format` / `format:check`；`lint` 去 `--ext`；`dist:mac:dir` 去 `CODEPAL_SKIP_RELEASE_FINISH`）。
- ✅ `typescript ^5.6` 较 CodePal 的 `^6.0.2` 保守，降低 T01 触发版本风暴的概率。
- 🟡 `@eslint/js` / `typescript-eslint` / `eslint-plugin-*` 未装。这是 T04 范围，T01 不应承担，留作后续。
- ❌ 无发现 bug / regression 级问题。

## 问题分流

| 严重度 | 条目 | 处理 |
| --- | --- | --- |
| 🔴 高（bug / regression） | 无 | — |
| 🟡 中（延后可接受） | `dist:mac:dir` CLI flag 在 T18 后简化 | 已写入 `workspace/T01/todo.md` #1 |
| 🟡 中（延后可接受） | `playwright` 冗余装在 T07 决定 | 已写入 `workspace/T01/todo.md` #2 |
| 🟡 中（延后可接受） | Electron 二进制未下载，T03 前重跑 install | 已写入 `workspace/T01/todo.md` #3（新增） |
| 🟢 低 | `README.md` 尚未加 Scripts 表格章节 | 已写入 `workspace/T01/todo.md` #5，保持 T01 不越界 |

## 最终判定

- 无高严重度问题。
- 中严重度均已分流到 `todo.md`，由后续任务消化。
- review 与 task 文档、OpenSpec change、代码状态彼此一致。
- **通过 review，可以进入最终对比与归档阶段。**
