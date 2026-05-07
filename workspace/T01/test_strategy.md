# T01 · 测试策略

## 本任务可测性前提

T01 只产出 `package.json` 与 `package-lock.json`，**不写任何运行时代码**。因此：

- 无 Vitest 单测可写（没有函数 / 模块可被单测，强行写会是"为测试而测试"）。
- 无 Playwright e2e 可写（没有 UI / IPC / 主进程入口存在）。
- 无 lint / typecheck / build 绿色要求（这些在 T02–T09 才开始绿）。

本任务的"测试"= **契约静态 / 动态校验**。

## 功能目标 → 验证手段映射

| 目标（来自 `specs/package-scripts-contract/spec.md`） | 证明方式 | 命令 | 能证明什么 |
| --- | --- | --- | --- |
| package.json 存在且字段合规 | 静态 | `node -e "const p=require('./package.json'); console.log(p.name, p.private, p.type, p.main, p.engines.node)"` | name/private/type/main/engines.node 5 字段与 spec 一致 |
| 9 条核心脚本 + 3 条辅助脚本全部存在 | 动态 | `npm run -s` 并对照目标清单 | 脚本名集合完整；若缺一条，立即失败 |
| 脚本命令语义与契约一致 | 静态 | `node -e "console.log(JSON.stringify(require('./package.json').scripts, null, 2))"` 并 diff 期望值 | 命令字符串一字不差 |
| devDependencies 覆盖每条脚本的二进制 | 静态 | `node -e "const d=require('./package.json').devDependencies; ['electron','electron-vite','electron-builder','eslint','prettier','typescript','vitest','@playwright/test'].forEach(k=>{if(!d[k]) process.exitCode=1})"` | 二进制包都被声明 |
| 命令可被 npm 解析（命令存在性） | 动态 | `npm run lint -- --version`；对 typecheck / test / build 同构探针 | 没有 `missing script`；工具本身是否绿不在 T01 范围 |
| package-lock.json 与 package.json 同步 | 动态 | `npm ls --depth=0 >/dev/null 2>&1; echo $?` | lockfile 自洽 |
| OpenSpec change 自身合法 | 静态 | `openspec validate bootstrap-package-scripts --strict` | change 产物结构正确，所有 Requirement 带 SHALL/MUST + Scenario |

## 显式跳过 / 降级

| 项 | 状态 | 理由 |
| --- | --- | --- |
| Vitest 单测 | 跳过 | 无可测代码 |
| Playwright e2e | 跳过 | 无 UI / 入口 |
| `npm run lint` 绿 | 跳过 | T04 范围 |
| `npm run typecheck` 绿 | 跳过 | T02 范围 |
| `npm run build` 绿 | 跳过 | T03 / T18 范围 |
| `npm run dev` 真机冒烟 | 跳过 | T03 Goal，且 T01 无 Electron 入口 |
| `npm run dist:mac` 绿 | 跳过 | T18 / T19 范围 |
| `npm install` 失败时的降级 | 允许 | 命令存在性不依赖 install；失败则在 `workspace/T01/todo.md` 记录 |

## 结果落盘位置

全部验证产出写入 `workspace/T01/verification.md`（实现阶段与验证阶段共同维护）。
