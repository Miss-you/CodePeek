# Tasks · bootstrap-package-scripts

## 1. 准备

- [ ] 1.1 确认 workspace `workspace/T01/` 已有 `current_state.md` / `reference_impl.md` / `final_impl.md` / `todo.md`
- [ ] 1.2 确认处于隔离 worktree `../CodePeek-T01` 分支 `task/T01-bootstrap-package-scripts`
- [ ] 1.3 `docs/plans/bootstrap-engineering-design-task.md` 中 T01 状态已推进到 `spec`

## 2. 落地 package.json

- [ ] 2.1 在仓库根创建 `package.json`，按 `final_impl.md` 中"最终方案"落字段（name / version / private / type / main / engines / scripts / devDependencies）
- [ ] 2.2 用 `node -e "JSON.parse(require('fs').readFileSync('package.json','utf8'))"` 校验 JSON 合法性
- [ ] 2.3 用 `node -e` 打印 `scripts` 的 keys，确认 9 条核心脚本 + 3 条辅助脚本全部存在

## 3. 生成 package-lock.json

- [ ] 3.1 在仓库根执行 `npm install`
- [ ] 3.2 确认生成 `package-lock.json`，且 `node_modules/` 被 `.gitignore` 忽略
- [ ] 3.3 若 install 失败（网络 / 平台二进制），按 `workspace/T01/todo.md` 记录降级策略，脚本契约维度仍需通过验证

## 4. 验证

- [ ] 4.1 首验：`npm run` 列出全部目标脚本（写入 `workspace/T01/verification.md`；npm 11 下 `-s` 会静默，故使用 `npm run`）
- [ ] 4.2 探针：对 `lint` / `typecheck` / `test` / `build` 各跑一次 `npm run <name>` 或 `npm run <name> -- --help`，确认不报 `missing script`（允许工具本身因缺配置而退出非 0）
- [ ] 4.3 对照 `specs/package-scripts-contract/spec.md` 每条 Scenario 的 WHEN/THEN，写入 `workspace/T01/verification.md`
- [ ] 4.4 `openspec validate bootstrap-package-scripts --strict` 通过

## 5. 收尾

- [ ] 5.1 推进 `docs/plans/bootstrap-engineering-design-task.md` 中 T01 状态 `spec` → `implementing` → `verifying` → `review` → `done`，并逐步在 Change Log 留痕
- [ ] 5.2 `openspec archive bootstrap-package-scripts` 成功（归档到 `openspec/changes/archive/`）
- [ ] 5.3 在 T01 行把 Done When 标记为已满足；补充 `workspace/T01/todo.md` 中最终遗留项
