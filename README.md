# Doo-ai-brain

**Doo 的个人 AI 工程大脑**

一个长期技术知识库，设计目标是任何 AI Agent（Claude Code、Hermes、ChatGPT 等）和人类都能直接阅读。

## 目的

这个仓库是我的外部 AI 大脑——一个持久化、版本可控的知识库。任何 AI Agent 都可以通过阅读它来理解我的项目、技术决策、实验记录和工程模式。

## 仓库结构

```
Doo-ai-brain/
├── README.md                 # 本文件
├── AI_RULES.md               # AI Agent 读取规则
├── profile/                  # 我的工程师画像和工作风格
├── status/                   # 所有活跃项目的当前状态
├── projects/                 # 按项目组织的知识和经验
├── knowledge/                # 技术领域知识
├── experiments/              # 独立实验记录
├── decisions/                # 架构决策记录
└── sync/                    # 同步日志
```

## AI Agent 使用方式

1. **启动时**：先读 `AI_RULES.md`，再读 `status/current_status.md`
2. **接手项目前**：阅读 `projects/<项目名>/overview.md` 和相关文档
3. **遇到技术问题**：搜索 `knowledge/` 中的技术笔记
4. **完成重要工作后**：更新相关文件并同步回仓库

## 我的使用方式

1. 完成一个有价值的任务
2. 在 Claude Code 中执行 `/brain-sync`
3. Agent 回顾会话，识别长期价值
4. Agent 更新相关 Markdown 文件
5. Git commit + push

## 核心原则

- **工具无关**：纯 Markdown，任何 AI 或人类都能读
- **宁缺毋滥**：记录决策、实验、失败——不记录无意义的修改
- **版本可控**：完整记录技术思考的演进
- **双向同步**：台式机和笔记本都是知识贡献者
