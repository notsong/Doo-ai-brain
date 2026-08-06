# Doo Label — 项目概览

> 金相图像标注工具。支持夹杂物多边形标注、晶粒度晶界标注、AI 预标注，输出 Labelme JSON 和 PASCAL VOC 格式。

## 项目简介

Doo Label 是一款专为材料科学金相显微图像设计的桌面标注工具。基于 PySide6/Qt 构建，支持两种工作模式：夹杂物多边形轮廓标注（Labelme 兼容 JSON）和晶粒度晶界折线/画笔标注（VOC 分割格式导出）。内置 ONNX AI 预标注加速标注效率。

代码约 6000 行，版本 v3.2。

## 目标

- 高效标注金相图像中的夹杂物、晶界、孪晶界
- 输出与 Labelme 兼容的 JSON 和 PASCAL VOC 分割格式
- 内置 AI 模型（ONNX）预标注，减少人工标注量
- 一键打包为独立 exe，方便非开发者使用

## 技术栈

- **UI 框架**: PySide6（Qt 6.5+），Fusion 风格 + VS Code 深色 QSS 主题
- **画布**: QGraphicsView/QGraphicsScene 体系
- **算法**: OpenCV 4.8（FloodFill, OTSU, 骨架化, 形态学）
- **AI 推理**: ONNX Runtime (CPU)
- **打包**: PyInstaller
- **测试**: pytest（57 个单元测试）

## 当前状态

🟢 v3.2 稳定运行。MVC 四层架构重构完成，晶粒度模块独立化。

## 关键文件与入口

| 文件 | 作用 |
|------|------|
| `doo_label/main.py` | 应用入口 |
| `doo_label/ui/main_window.py` | 主窗口（38KB） |
| `doo_label/ui/canvas.py` | 画布交互（43KB） |
| `doo_label/domain/` | 领域层（Shape, ClassManager, AnnotationManager） |
| `doo_label/engine/` | 纯算法层（IO, 轮廓, AI 推理, 晶界后处理, 栅格化） |
| `doo_label/application/` | 用例层（Undo, 状态机, UseCases） |
| `doo_label/modes/grain/` | 晶粒度独立模块 |
| `models/` | ONNX 模型文件 |
| `tests/` | pytest 57 个单元测试 |

## 功能清单

### 夹杂物模式
- 手动多边形描点标注（顶点 15px 吸附）
- 魔法棒提取（FloodFill + 容差）
- 框选提取（OTSU / Adaptive 阈值）
- 标签自定义（随机舒适色、动态添加）
- 快捷键：Ctrl+S 保存, Ctrl+Z 撤销, Delete 删除, Ctrl+滚轮缩放

### 晶粒度模式
- LineStrip 开放折线绘制晶界
- Brush 画笔涂抹/擦除
- AI 预标注（ONNX 推理 → 多类 mask）
- 晶界后处理管线（去噪→骨架化→剪枝→闭合→膨胀）
- PASCAL VOC 分割格式导出

## 参考资料

- 项目文档: `docs/`
- CI 脚本: `claude_ci.bat`
