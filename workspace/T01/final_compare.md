# T01 · Final Compare

最后一次回到原始目标与参考实现，确认无未记录的高严重度偏差。

## A · 与 task 板 T01 行的对照

| 维度 | task 板要求 | 实际产出 | 判定 |
| --- | --- | --- | --- |
| Goal | 落地 `package.json`，定义 9 条核心脚本 | `package.json` 含 `dev` / `build` / `test` / `test:e2e` / `lint` / `typecheck` / `format` / `dist:mac` / `dist:mac:dir` | ✅ |
| Depends On | — | 无前置依赖 | ✅ |
| Workspace | `workspace/T01/` | `workspace/T01/{current_state,reference_impl,final_impl_v1,final_impl,test_strategy,verification,review,todo}.md` | ✅ |
| Change | `bootstrap-package-scripts` | `openspec/changes/bootstrap-package-scripts/{proposal,design,tasks}.md` + `specs/package-scripts-contract/spec.md`，`openspec validate --strict` 通过 | ✅ |
| Done When 1 | 仓库根存在 `package.json` | 存在；JSON 合法 | ✅ |
| Done When 2 | `npm run lint/typecheck/test/build` 在 M1 完成后能被无错调用 | 已逐项 dry run；无 `missing script`；工具级失败由 T02–T06 范围解释 | ✅ |
| 首验 | `npm run` 列出全部目标脚本（npm 11 下 `-s` 静默，故 Scenario 统一 `npm run`） | 12 条脚本（9 核心 + 3 辅助）全部列出 | ✅ |

**对原始目标无任何未记录偏差。**

## B · 与源设计 §1 / 第二部分 P0 的对照

| 源设计要求 | 实际 | 判定 |
| --- | --- | --- |
| 9 条核心脚本契约 | 全部存在 | ✅ |
| `engines.node>=22` | `>=22` | ✅ |
| `electron-vite` 三进程入口 | T03 范围；T01 仅装 `electron-vite` 占位 | ✅（不越界） |
| 三套 tsconfig | T02 范围；T01 仅装 `typescript` 占位，`typecheck` 用 `tsc -b` | ✅（前置兼容） |
| ESLint flat + Prettier | T04 范围；T01 仅装 `eslint` / `prettier` 占位 | ✅（不越界） |
| Vitest / Playwright | T06 / T07 范围；T01 仅装占位 | ✅（不越界） |
| `simple-git-hooks` + `lint-staged` | T08 范围 | ✅（不越界） |
| `.editorconfig` / `.gitattributes` / `.nvmrc` | T05 范围 | ✅（不越界） |

**P0 工程化清单中属于 T01 的项全部覆盖；属于他任务的项一概不触达。**

## C · 与参考实现 CodePal 的对照

| 维度 | CodePal | CodePeek T01 | 偏差是否记录 |
| --- | --- | --- | --- |
| 脚本核心语义 | 见 `reference_impl.md` 表 | 一致 | ✅ |
| `lint` 写法 | `eslint . --ext .ts,.tsx` | `eslint .` | 已在 `design.md` D4 / `reference_impl.md` 解释 |
| `typecheck` | 缺失 | `tsc -b` | 已在 `design.md` D3 解释（任务板硬要求） |
| `format` / `format:check` | 缺失 | 存在 | 已在 `design.md` D5 解释 |
| `dist:mac:dir` 环境变量 | `CODEPAL_SKIP_RELEASE_FINISH=1` | 不带 | 已在 `reference_impl.md` 解释（CodePal 专属） |
| `release:mac` / `seed:history` / `dist:mac:unsigned:test` / `preview` | 全有 | 仅 `preview` | 已在 `final_impl.md` / `design.md` 解释（不越界） |
| TypeScript 主版本 | 6.0.2 | 5.6.x | 已在 `reference_impl.md` / `final_impl_v1.md` 解释（保守） |

**所有偏离都已就地记录原因；无静默偏差。**

## D · 风险与遗留收口

`workspace/T01/todo.md` 已记录 6 项后续清理点（dist:mac:dir 简化、playwright 冗余、Electron 二进制重装、electron-updater 后补、README scripts 段、`.gitignore` 扩充）。

**无未记录的高严重度风险。**

## E · 状态一致性快照

| 证据维度 | 状态 |
| --- | --- |
| task 板（即将翻 `done`） | T01 = `review`，本步骤翻 `done` |
| OpenSpec change | `bootstrap-package-scripts` validate 通过；本步骤 archive |
| 仓库代码 | `package.json` + `package-lock.json` 已落 |
| workspace 产物 | 8 份齐全 |
| Change Log | 已留痕 claim/spec/implementing/verifying/review；本步骤补 done |

**一致性达成，可关单。**
