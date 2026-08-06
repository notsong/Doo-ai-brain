# Hermes 接入 Doo-ai-brain 指南

> 阅读时间：5 分钟。读完你就知道怎么接入和使用了。

---

## 一、这是什么

**Doo-ai-brain** 是我的个人 AI 工程大脑——一个 Git 版本化的纯 Markdown 知识库。

它不是给某个特定 AI 工具用的，而是给你（Hermes）、Claude Code、以及未来任何 AI Agent 共用的**长期外部记忆系统**。

整体架构：

```
                 Hermes (大H)  ← 台式机 (Win11 主力机)
                     |
              Doo-ai-brain Git 仓库
                     |
                 Hermes (小H)  ← 笔记本 (Win10，本实例)
                     |
              Claude Code (编码执行助手)
```

**分工**：
- **大H（台式机 Hermes）**：长期大脑主力，技术方案设计与知识管理
- **小H（笔记本 Hermes）**：长期记忆、项目经验积累、技术方案设计、实验总结、知识管理
- **Claude Code**：编码、改文件、调试、跑实验（执行助手，不是大H）

---

## 二、仓库地址与获取

```
https://github.com/notsong/Doo-ai-brain
```

接入方式：

```bash
git clone https://github.com/notsong/Doo-ai-brain.git
```

之后每次使用前：

```bash
cd Doo-ai-brain
git pull   # 拉取最新知识
```

---

## 三、仓库结构

```
Doo-ai-brain/
├── README.md                    # 仓库说明
├── AI_RULES.md                  # ⚠️ 先读这个！AI 使用规则
├── HERMES_START.md              # 本文件
│
├── profile/                     # 我的工程师画像
│   ├── engineer_profile.md      # 技术背景、技能、偏好
│   └── working_style.md         # 工作方式、跨机器模式
│
├── status/
│   └── current_status.md        # 所有项目的当前状态
│
├── projects/                    # 按项目组织的知识
│   ├── dinov3/                  # DINOv3 晶界分割 + 检索
│   ├── grain_size/              # U-Net 金相组织/缺陷分割
│   ├── doo_label/               # 金相图像标注工具
│   └── ai_server/               # 待分析
│
├── knowledge/                   # 技术领域知识
│   ├── pytorch.md
│   ├── opencv.md
│   ├── onnx.md
│   ├── cuda.md
│   └── deployment.md
│
├── experiments/                 # 独立实验记录
├── decisions/                   # 架构决策记录
└── sync/
    └── sync_log.md              # 每次同步的日志
```

---

## 四、你的使用方式

### 4.1 每次启动时（必读）

按顺序读这 3 个文件，你就知道我是谁、在做什么：

1. `AI_RULES.md` — 了解规则
2. `profile/engineer_profile.md` — 了解我的背景
3. `status/current_status.md` — 了解当前项目状态

### 4.2 接手具体项目时

读 `projects/<项目名>/overview.md` 和相关文档。

### 4.3 遇到技术问题

搜索 `knowledge/<技术>.md`。里面是我从实际项目中积累的模式、技巧、踩过的坑。

### 4.4 做出技术决策时

读 `decisions/architecture_decisions.md` 了解历史决策，避免重复讨论。

---

## 五、你如何贡献（重要）

你是知识贡献者，不只是消费者。

### 什么时候贡献

当你完成了以下事情时，应该更新知识库：

- 设计了技术方案
- 总结了实验结果
- 发现了值得记录的工程模式
- 做了架构决策

### 怎么贡献

1. 打开对应的 Markdown 文件
2. 追加你的内容（不要删除已有的）
3. 如果是新项目，在 `projects/` 下创建子目录
4. 更新 `sync/sync_log.md` 记录本次变更
5. Git commit + push

### 提交格式

```bash
git add -A
git commit -m "sync: <一句话说明改了什麼>"
git push
```

commit message 用中文或英文都行，但要清晰。

---

## 六、核心原则

这些是 `AI_RULES.md` 里定义的规则，你也遵循：

### 黄金法则
> "如果 6 个月后我接手一个新项目，我需要知道什么？"

能回答这个的就记录。不能就跳过。

### 应该记录
- 技术决策（为什么选 X 不选 Y）
- 项目经验（关键发现、坑、解决方案）
- 实验结果（测试了什么、结论）
- 失败方案（什么不行、为什么——这个很重要！）
- 工程模式（可复用的做法）
- 环境配置（花了时间才搞定的）

### 不要记录
- 日常代码修改
- 临时调试过程
- 项目自己仓库已经写了的东西
- 常识
- 聊天日志

---

## 七、关于我（Doo）

- 钢铁材料金相检测 × 计算机视觉 复合型工程师
- 主要用 Python / PyTorch / OpenCV / ONNX
- 当前在做：晶界分割（DINOv3 + U-Net）、标注工具（Doo_Label）、晶粒度评级等
- 更多细节读 `profile/engineer_profile.md`

---

## 八、下一步

1. `git clone https://github.com/notsong/Doo-ai-brain.git`
2. 读 `AI_RULES.md`
3. 读 `profile/engineer_profile.md`
4. 读 `status/current_status.md`
5. 告诉我你读完了，然后我们讨论你第一个要分析的项目

---

*这就是全部。接上吧 Hermes。*
