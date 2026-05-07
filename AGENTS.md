# AGENTS.md

面向在本仓库中协作的 AI 编码代理（Claude Code、Codex、Cursor 等）的指引。

## 项目概况

CodePeek 是一个跨 IDE / AI 代理的统一监控面板，目标技术栈为 Electron + Vite + TypeScript + Tailwind CSS。当前仓库处于初始化阶段，尚无实现代码。

设计思路参考自 [shamcleren/CodePal](https://github.com/shamcleren/CodePal#)，但 CodePeek 是独立实现，不直接复用其代码。

## 协作约定

### 写代码

- 优先编辑现有文件，不要无端新建模块或抽象。
- 不要为不存在的边界条件添加防御性逻辑；只在系统边界（用户输入、外部 API）做校验。
- 默认不写注释；只在「为什么」非显而易见时才补一行。
- 不要在代码里留「为 X 任务添加」「修复 issue #123」之类的临时性注释，这类信息属于 PR 描述。
- 不引入未被任务要求的重构、清理或预留接口。

### 测试与验证

- UI / 前端改动必须在浏览器或 Electron 实例中亲自走一遍主流程，再声明完成。
- 类型检查与测试通过 ≠ 功能正确，需如实区分。
- 提交前确认 lint、type-check、相关测试均已运行。

### 提交规范

- 用祈使句、英文小写动词开头，一行总结主旨：`add session timeline view`、`fix quota refresh race`。
- 重要决策放正文，解释「为什么」而非「做了什么」。
- 不要在未授权时执行 `git push --force`、`git reset --hard`、`--no-verify` 等破坏性操作。

### 提问与澄清

需求模糊或与现有实现冲突时，先停下来确认，不要凭直觉补全。设计取舍优先在 PR / Issue 中讨论，再落到代码。

## 仓库结构（规划）

```
CodePeek/
├── src/             # Electron 主进程与渲染进程源码（待建立）
├── tests/           # 单元 / e2e 测试（待建立）
├── README.md
├── AGENTS.md        # 通用代理指引（本文件）
├── CLAUDE.md        # Claude Code 专属补充
└── LICENSE
```

## 引用

- 灵感来源：[shamcleren/CodePal](https://github.com/shamcleren/CodePal#)
