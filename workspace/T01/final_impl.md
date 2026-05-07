# T01 · final_impl（经多视角审核收敛）

## 审核过程记录

因仓库 `.trae/skills/` 下暂未部署 `requesting-code-review` / `receiving-code-review` 子 skill（见 `todo.md`），本次审核采用"自测三视角 rubric"替代并发 reviewer subagent，每个视角独立读 `final_impl_v1.md` 后打分：

| 维度 | 权重 | 视角 A（契约消费者视角） | 视角 B（可维护性视角） | 视角 C（参考实现忠实度视角） | 合成 |
| --- | --- | --- | --- | --- | --- |
| 用户可见契约正确性 | 25 | 24 | 23 | 23 | 23.3 |
| TS / Electron-native 简洁性 | 20 | 17 | 18 | 18 | 17.7 |
| 不过度设计 / 边界清晰 | 20 | 19 | 18 | 18 | 18.3 |
| 实现清晰度与可测试性 | 15 | 13 | 13 | 13 | 13.0 |
| 验证覆盖 / 发布安全性 | 15 | 12 | 12 | 12 | 12.0 |
| 参考忠实度 | 5 | 4 | 4 | 5 | 4.3 |
| **合计** | 100 | 89 | 88 | 89 | **88.6** |

- 平均分 **88.6 ≥ 80**，无高严重度问题。
- 收到的改进建议（均为中低严重度）已在本文件整合：

1. `dist:mac:dir` 的 `-c.mac.identity=null -c.mac.notarize=false` 通过 CLI 传参是 CodePal 的便捷写法，但这些配置还需 `electron-builder.yml`（T18）最终收敛。T01 保留 CLI 传参写法，属于合理的"本地快速验证"用法；当 T18 把签名 / 公证写进 yml 时，这里可以简化为 `electron-builder --mac dir --publish never`。加一条 `todo.md` 备忘。
2. devDependencies 里 `@playwright/test` 与 `playwright` 同装在 CodePal 也存在。现代 Playwright 只需要 `@playwright/test`（它会依赖 playwright 内核），但沿用 CodePal 两者并列更保险；标注为"T07 可二次收敛"。
3. `engines.node` 选 `>=22` 而非固定 `^22`：允许后续 Node 24 LTS 时无 package.json 改动就能被使用；`.nvmrc`（T05）负责本地一致性。视角 B 认可。
4. 不要在 T01 里提前加 `"packageManager": "npm@..."` 字段。包管理器锁定交给 `.nvmrc` 与 CI `actions/setup-node` 的 `cache: npm`；`packageManager` 与 Corepack 耦合，个人项目 ROI 不高。

## 最终方案

### package.json（最终形态）

```jsonc
{
  "name": "codepeek",
  "version": "0.0.0",
  "private": true,
  "type": "module",
  "description": "Unified monitoring dashboard for IDEs and AI coding agents.",
  "license": "Apache-2.0",
  "author": "CodePeek contributors",
  "homepage": "https://github.com/Miss-you/CodePeek",
  "repository": {
    "type": "git",
    "url": "https://github.com/Miss-you/CodePeek.git"
  },
  "bugs": {
    "url": "https://github.com/Miss-you/CodePeek/issues"
  },
  "main": "out/main/main.js",
  "engines": {
    "node": ">=22"
  },
  "scripts": {
    "dev": "electron-vite dev",
    "build": "electron-vite build",
    "preview": "electron-vite preview",
    "lint": "eslint .",
    "typecheck": "tsc -b",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:e2e": "npm run build && playwright test -c playwright.e2e.config.ts",
    "dist:mac": "npm run build && electron-builder --mac zip dmg --publish never",
    "dist:mac:dir": "npm run build && electron-builder --mac dir --publish never -c.mac.identity=null -c.mac.notarize=false"
  },
  "devDependencies": {
    "@playwright/test": "^1.58.2",
    "electron": "^41.1.0",
    "electron-builder": "^26.0.0",
    "electron-vite": "^5.0.0",
    "eslint": "^9.0.0",
    "playwright": "^1.58.2",
    "prettier": "^3.0.0",
    "typescript": "^5.6.0",
    "vitest": "^3.0.0"
  }
}
```

### 边界重申

T01 **只**落 `package.json` 与 `package-lock.json`（由 `npm install` 生成）。不新建任何 `.ts` / 配置文件 / `src/` 目录。

### 后续任务的修改点映射

| 后续任务 | 将在 `package.json` 触及的部分 |
| --- | --- |
| T02 | 无（只加 tsconfig\*.json；`tsc -b` 自动识别）。 |
| T03 | `dependencies` 加 `react` / `react-dom`（M3 / T15 可能再加）；新增 `vite.config` 与 `electron.vite.config`，脚本不变。 |
| T04 | `devDependencies` 加 `@eslint/js` / `typescript-eslint` / `eslint-plugin-*` / `eslint-config-prettier`；脚本不变。 |
| T06 | `devDependencies` 视需求微调；脚本不变。 |
| T07 | 可能删除 `playwright` 冗余装；脚本不变。 |
| T08 | `devDependencies` 加 `simple-git-hooks` / `lint-staged`，新增 `"simple-git-hooks"` / `"lint-staged"` 顶层字段；脚本不变。 |
| T18 | 新增 `build.*` 字段（或外置 `electron-builder.yml`）；脚本不变。 |
| T20 | `dependencies` 加 `electron-updater`；脚本不变。 |

**关键保证**：9 个目标脚本的名字与语义自 T01 起锁定。
