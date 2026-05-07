# CLAUDE.md

本文件是 [Claude Code](https://claude.com/claude-code) 在本仓库中工作的入口指引。通用约定见 [AGENTS.md](./AGENTS.md)，本文件仅补充 Claude Code 专属内容。

## 优先阅读

- [README.md](./README.md)：项目愿景与计划功能
- [AGENTS.md](./AGENTS.md)：所有 AI 代理共用的协作约定

## Claude Code 特定建议

- 仓库目前为空骨架，进行任何编码任务前先确认目标目录结构与脚手架已经初始化；如未就绪，先与用户澄清而不是自行铺开框架。
- 涉及 Electron 主进程 / 渲染进程的改动，桌面 UI 不能仅靠类型检查通过就声明完成，需要真实运行。
- 引用其他代理或 IDE 的协议时，以官方文档为准，不要根据记忆杜撰接口。

## 引用

本项目设计思路参考自 [shamcleren/CodePal](https://github.com/shamcleren/CodePal#)，CodePeek 为独立实现。
