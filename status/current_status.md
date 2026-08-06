# 当前状态

> 所有活跃项目的快照。每次 `/brain-sync` 后更新。
> AI Agent：先读这个，了解我当前在做什么。

## 活跃项目

| 项目 | 状态 | 优先级 | 最后更新 | 当前重点 |
|------|------|--------|----------|----------|
| doo-ai-brain | 🟢 活跃 | 高 | 2026-08-06 | 项目文档已填充，持续积累 |
| dinov3 | 🟢 已部署 | 高 | 2025-07-29 | DINOv3 晶界分割 + 晶粒度检索，已集成检测系统 |
| grain_size (U-Net) | 🟢 多项目并行 | 高 | 2025-08-05 | SX 双相钢滑窗训练，TE-VVP 后处理协议 |
| doo_label | 🟢 v3.2 稳定 | 中 | 2025-05-11 | 架构重构完成，打包 DLL 问题待根治 |
| ai_server | 待分析 | - | - | 路径待确认 |

## 当前焦点

Doo-ai-brain 知识库初始化——已分析 3 个代码项目 + 1 个文档目录，经验已沉淀到知识库；大H（台式机 Hermes）+ 小H（笔记本 Hermes）双实例接入。

## 阻塞项

- Doo_Label 打包：conda 环境 ICU DLL 缺失，需切 pip venv
- Dino_v3-plus 文档滞后于代码（V3 架构未更新 README）

## 近期成果

- ✅ Doo-ai-brain 仓库创建、模板初始化、中文化
- ✅ `/brain-sync` skill 安装
- ✅ 3 个项目（DINOv3, U-Net, Doo_Label）深度分析并写入知识库
- ✅ 工程师画像建立
- ✅ 5 个知识文件（PyTorch, OpenCV, ONNX, CUDA, 部署）填充真实经验
- ✅ Hermes（小H）接入共享大脑：克隆仓库、建立读写协议与技能、确认角色分工

## 下一步计划

1. 分析 `ai_server` 项目（如果有独立路径）
2. 小H（Hermes）参与：技术方案设计、实验总结、知识库维护
3. 笔记本配置 Doo-ai-brain 同步
4. Dino_v3-plus 文档更新（README 与代码同步）
5. Doo_Label DLL 打包问题根治

---

*最后同步：2026-08-06*
