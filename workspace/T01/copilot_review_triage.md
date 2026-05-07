# T01 · Copilot Review Triage (PR #1, 2026-05-07 15:31)

Copilot review: 1 summary (`COMMENTED`) + 18 inline comments. All 18 are actionable documentation corrections — no runtime code, no `package.json` changes required.

## Finding Themes

### Theme A · `npm run -s` 在 npm 11 下静默（15 条）

Copilot correctly flagged that every doc/spec/task that prescribes `npm run -s` as the "first verification command" conflicts with `workspace/T01/verification.md`, which already records that npm 11 silences `-s` and we fell back to `npm run`.

**Fix strategy**: globally replace `npm run -s` with `npm run` across the affected files; where the doc explicitly describes the npm 11 fallback, keep the nuance.

Affected files (15 occurrences):
- `workspace/T01/current_state.md:34`
- `workspace/T01/test_strategy.md:18`
- `workspace/T01/final_compare.md:15`
- `workspace/T01/final_impl_v1.md:105`, `:114`
- `workspace/T01/todo.md:12`
- `workspace/T01/verification.md:139`
- `openspec/specs/package-scripts-contract/spec.md:44`, `:83`
- `openspec/changes/archive/2026-05-07-bootstrap-package-scripts/specs/package-scripts-contract/spec.md:41`, `:81`
- `openspec/changes/archive/2026-05-07-bootstrap-package-scripts/tasks.md:23`
- `openspec/changes/archive/2026-05-07-bootstrap-package-scripts/design.md:102`
- `openspec/changes/archive/2026-05-07-bootstrap-package-scripts/proposal.md` (via count)
- `docs/plans/bootstrap-engineering-design-task.md:42`

### Theme B · 文档内部一致性（3 条）

- **#3199664487** (`reference_impl.md:49`): 旧版写 "`preview` 暂不落"；实际 `package.json.scripts.preview = electron-vite preview`。修正为"已采纳"。
- **#3199664529** (`reference_impl.md:61`): "显式偏离并记录" 列表仍写 "不落 preview"。修正：preview 已采纳，实际显式不落的是 `release:mac` / `seed:history` / `dist:mac:unsigned:test`。
- **#3199664546** (`proposal.md:10`): "9 核心 + 4 辅助" 与实际 12 条脚本（9 + 3）不一致。修正为 3。
- **#3199664653** (`verification.md` V3): "8 核心 devDependencies" 与实际 9 条（含 `playwright`）不一致。修正计数 + 补 `playwright` 说明。

## Severity

| 等级 | 数量 | 说明 |
| --- | --- | --- |
| 🔴 高（阻塞 merge） | 0 | 无 |
| 🟡 中（文档不一致，reviewer & AI skill 复现受影响） | 18 | 本次修复 |
| 🟢 低 | 0 | 无 |

## Rejected / Deferred

**无**。所有 18 条意见都是 T01 范围内的文档偏差，均应立即修复。

## 行动

1. 批量替换 `npm run -s` → `npm run`（保留 verification.md 中"解释 npm 11 行为"的原文）。
2. 修复 `reference_impl.md` 中 `preview` 的描述。
3. 修复 `proposal.md` 辅助脚本计数。
4. 修复 `verification.md` V3 devDeps 枚举。
5. 追加 1 个 commit `address Copilot review: npm run -s → npm run + doc consistency` 推到同一分支。
6. resolve 18 条 Copilot 评论（通过推送新 commit + 在 PR 回 summary 评论）。
