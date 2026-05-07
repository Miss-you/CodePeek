# CodePeek

> 一个面向多 IDE 与 AI 编码代理的统一监控面板。

## 项目简介

CodePeek 旨在把分散在不同终端、IDE、浏览器标签里的 AI 编码会话聚合到一个轻量的悬浮面板：哪一个会话正在等待审批、哪一个还在跑、配额还剩多少，一眼可见，不再来回切换窗口。

## 计划支持

- Cursor
- Claude Code
- Codex
- CodeBuddy（含 GoLand / PyCharm 插件）

## 核心目标

- **会话聚合**：跨代理展示运行中 / 等待中 / 已完成 / 失败 的会话状态
- **活动时间线**：记录回复、工具调用、状态切换等事件
- **配额可视化**：监控 token 用量与限速信号
- **本地持久化**：完整保留历史活动，重启后自动恢复
- **集成自检**：一键修复本地代理的配置问题
- **双语界面**：简体中文与英文，跟随系统语言

## 项目状态

本仓库目前处于初始化阶段，尚未提交实现代码，仅含许可证文件与文档骨架。后续将基于 Electron + Vite + TypeScript 技术栈搭建桌面端。

## 开发（规划中）

```bash
git clone https://github.com/Miss-you/CodePeek.git
cd CodePeek
npm install
npm run dev        # 开发模式
npm run test       # 单元测试
npm run dist:mac   # 构建 macOS 安装包
```

> 上述命令在工程脚手架就绪后生效。

## 许可证

本项目遵循 [Apache License 2.0](./LICENSE)。

## 致谢与引用

本项目的设计思路与功能规划参考自 [shamcleren/CodePal](https://github.com/shamcleren/CodePal#)，感谢原作者的开源工作。CodePeek 是一次独立实现，并将在此基础上探索更多面向 AI 代理协作场景的能力。
