# T01 · 参考实现摘要（CodePal）

## 来源

[shamcleren/CodePal](https://github.com/shamcleren/CodePal) `package.json`（v1.1.4，已拉取快照）。
CodePal 是 CodePeek 源设计明确对齐的参考实现；T01 的脚本契约与其高度同构是合理的。

## 关键片段

```jsonc
{
  "name": "codepal",
  "version": "1.1.4",
  "private": true,
  "type": "module",
  "main": "out/main/main.js",
  "scripts": {
    "dev": "electron-vite dev",
    "build": "electron-vite build",
    "dist:mac": "npm run build && electron-builder --mac zip dmg --publish never",
    "dist:mac:dir": "npm run build && CODEPAL_SKIP_RELEASE_FINISH=1 electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false",
    "preview": "electron-vite preview",
    "test": "vitest run",
    "test:e2e": "npm run build && playwright test -c playwright.e2e.config.ts",
    "test:watch": "vitest",
    "lint": "eslint . --ext .ts,.tsx"
  }
}
```

## 与 CodePeek T01 的差异分析

| 主题 | CodePal | CodePeek T01 采纳方案 | 理由 |
| --- | --- | --- | --- |
| `type` | `"module"` | `"module"` | electron-vite + vite 生态默认 ESM。 |
| `main` | `"out/main/main.js"` | `"out/main/main.js"` | electron-vite 默认输出路径对齐。 |
| `dev` | `electron-vite dev` | `electron-vite dev` | 对齐。 |
| `build` | `electron-vite build` | `electron-vite build` | 对齐。 |
| `test` | `vitest run` | `vitest run` | 对齐。 |
| `test:e2e` | `npm run build && playwright test -c playwright.e2e.config.ts` | `npm run build && playwright test -c playwright.e2e.config.ts` | 对齐，强制先 build 再 e2e。 |
| `lint` | `eslint . --ext .ts,.tsx` | `eslint .` | flat config 时 `--ext` 已废弃，扩展名由 flat config 内部声明。 |
| `typecheck` | **缺失**（CodePal 靠 build 间接 typecheck） | `tsc -b` | 源设计 §2 明确把 typecheck 列为独立 gate；task 板把它写进 Done When。 |
| `format` | **缺失**（CodePal 无 prettier 脚本） | `prettier --write .` + `format:check` | 源设计 §2 要求 `prettier` + `eslint-config-prettier`，且 task 板 T01 列了 `format`。 |
| `dist:mac` | `npm run build && electron-builder --mac zip dmg --publish never` | 同 | 同。 |
| `dist:mac:dir` | `npm run build && CODEPAL_SKIP_RELEASE_FINISH=1 electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false` | `npm run build && electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false` | 去掉 CodePal 专属 `CODEPAL_SKIP_RELEASE_FINISH` 环境变量，CodePeek 暂无 release-finish 钩子。 |
| `dist:mac:unsigned:test` | 存在，调用自写脚本 | **暂不落**，源设计 §5.2 将其标为"本地 dist:mac:dir / dist:mac:unsigned:test 不签名跑一遍"但未进 T01 Goal；T19 / T18 再决定 | 不过度设计。 |
| `release:mac` | 存在 | **暂不落**，属 T21 的 CI release workflow 范围 | 边界清晰。 |
| `seed:history` | 存在 | **暂不落**，属 M3 Claude adapter 测试 fixture 范围 | 边界清晰。 |
| `preview` | 存在 | **暂不落**，task 板 T01 Goal 未列 | 不过度设计。 |
| 版本 | `1.1.4` | `0.0.0` | CodePeek 尚未首版。 |
| devDependencies 版本 | Electron 41 / vitest 3 / playwright 1.58 / typescript 6 / eslint 9 / electron-vite 5 / electron-builder 26 | 采用 CodePal 同代 major（取 npm 最新稳定版） | 与参考实现保持可比；lockfile 由 `npm install` 决定 patch。 |

## 采纳度评估

- **高忠实度**：`dev` / `build` / `test` / `test:e2e` / `dist:mac` / `dist:mac:dir` 行为语义一比一对齐。
- **显式偏离并记录**：
  - 新增 `typecheck`（源设计 P0、task 板 Done When 要求）。
  - 新增 `format` + `format:check`（源设计 §2 + T01 Goal 列名）。
  - `lint` 去 `--ext`（flat config 惯例）。
  - `dist:mac:dir` 去 CodePal 专属环境变量。
  - 不落 `preview` / `release:mac` / `seed:history` / `dist:mac:unsigned:test`（显式越界）。

结论：T01 的脚本契约 = CodePal 核心脚本同构 + 显式补 `typecheck` / `format` + 去 CodePal 项目专属分支。
