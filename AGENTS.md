# Doo-ai-brain — AI Agent 入口

这是 Doo 的共享 AI 大脑仓库（GitHub: notsong/Doo-ai-brain）。任何 AI Agent（Hermes 大H/小H、Claude Code、未来其他 Agent）在本仓库工作时，请遵守以下协议。

## 你是谁

- **大H**：台式机（Win11 主力机）上的 Hermes——长期大脑主力
- **小H**：笔记本（Win10）上的 Hermes——长期记忆、技术方案设计、实验总结、知识管理
- **Claude Code**：编码执行助手（不是大H）

## 启动必读（按顺序）

1. `HERMES_START.md` — Hermes 接入指南
2. `AI_RULES.md` — 使用规则
3. `profile/engineer_profile.md` — Doo 的背景与偏好
4. `status/current_status.md` — 当前项目状态

接手项目 → `projects/<名>/overview.md`（+ algorithm/experiments/lessons）
遇到技术问题 → `knowledge/<技术>.md`
做决策前 → `decisions/architecture_decisions.md`

## 黄金法则

> "如果 6 个月后我接手一个新项目，我需要知道什么？"

能回答就记录，不能就跳过。

**记录**：技术决策（为什么 X 不 Y）、项目经验（关键发现/坑/解法）、实验结果（测了什么/结论）、失败方案（什么不行/为什么）、工程模式、环境配置。
**不记录**：日常代码修改、临时调试过程、项目仓库已有的内容、常识、聊天日志。

## 贡献协议

1. 工作前 `git pull` 拉取最新知识
2. 更新对应 Markdown 文件（**追加**内容，不删除已有内容）
3. 更新 `sync/sync_log.md`（机器名：台式机=`desktop`，笔记本=`laptop`）
4. `git add -A && git commit -m "sync: <一句话摘要>" && git push`

## 网络

GitHub 直连不通，必须走代理 `http://127.0.0.1:7897`（Clash Verge 默认端口）。新机器克隆：
`git -c http.proxy=http://127.0.0.1:7897 -c http.version=HTTP/1.1 clone https://github.com/notsong/Doo-ai-brain.git`
