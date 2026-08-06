# Doo Label — 架构设计

> 领域驱动 MVC 四层架构 + 晶粒度独立模块。

## 整体架构

```
doo_label/
├── config.py              # 全局配置（配色、默认标签、快捷键、常量）
├── main.py                # 入口
├── domain/                # 领域层（夹杂物，纯数据，零 Qt 依赖）
│   ├── shape.py           # AnnotationShape（UUID 标识，Shoelace 面积）
│   ├── class_manager.py   # ClassManager（标签唯一数据源，15 色调色板）
│   └── annotation_manager.py  # 标注 CRUD + 增量信号
├── application/           # 用例层
│   ├── undo_service.py    # 统一时间轴撤销栈 List[(type, payload)]
│   ├── interaction_state.py  # 状态机 State×Tool 二维正交
│   └── use_cases.py       # Create/Delete/UpdatePolygon, ChangeLabel
├── engine/                # 纯算法层（零 Qt 依赖）
│   ├── io_manager.py      # JSON 读写 + VOC 导出 + mask 读写
│   ├── contour_utils.py   # 轮廓工具 + 法线梯度搜索边缘精炼
│   ├── box_extract.py     # OTSU / Adaptive 阈值提取
│   ├── magic_wand.py      # FloodFill 提取
│   ├── ai_inference.py    # ONNX/PyTorch 推理引擎
│   ├── grain_postprocess.py  # 晶界后处理管线
│   └── rasterize.py       # LineStrip→Mask 栅格化
├── ui/                    # 界面层
│   ├── canvas.py (43KB)   # 画布交互
│   ├── main_window.py (38KB)  # 主窗口
│   ├── right_panel.py (25KB)  # 右侧面板
│   ├── file_list.py       # 文件列表导航
│   └── styles.py          # VS Code 深色 QSS 主题
└── modes/grain/           # 晶粒度独立模块（零侵入）
    ├── domain.py          # GrainLine / MaskDiff / GrainMask
    ├── manager.py         # 线段+Mask 双数据 + 独立信号
    ├── use_cases.py       # Command 模式（Snapshot/MaskDiff/FullMask）
    ├── io_manager.py      # Grain IO
    └── tools/             # LineStrip/Brush 工具
```

## 核心设计原则

### 状态机：State × Tool 二维正交
- **5 个 State**：IDLE → DRAWING → EDITING → SELECTING → PANNING
- 永不新增临时状态，保证行为可预测

### 单一数据源 + 单向数据流
```
UI → UseCase → Domain → Signal → UI（更新）
```
- AnnotationManager 是标签数据的唯一权威源
- 增量信号（shape_added/removed/updated）驱动 UI 更新

### Command 模式撤销
- 统一时间轴单栈（非双轨栈，避免交替操作顺序错乱）
- MaskDiff 增量存储（只记录变化像素），解决全量 Mask 内存爆炸

## 晶界后处理管线

```
去噪 + 骨架化 → 毛刺剪枝 → 形态学闭合（桥接 <10px 断裂） → 线宽膨胀
```

## 已知局限

1. UI 层无自动化测试（全部 57 个测试在领域和引擎层）
2. conda 环境 PyInstaller 打包仍有 ICU DLL 缺失问题
3. 根目录遗留旧版 `main.py`（v2.5 单体版）和 `main_v2_backup.py`
