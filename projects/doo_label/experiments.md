# Doo Label — 实验与踩坑记录

> 架构迭代、打包排障经验。

## 架构演进

| 版本 | 架构 | 关键变化 |
|------|------|----------|
| v2.5 | 单体 `main.py`（693 行） | 全部逻辑混在一起 |
| v3.0 | MVC 分层 | domain/engine/ui 三层拆分 |
| v3.1 | + 深色主题 + bug 修复 | Fusion 风格 + VS Code QSS |
| v3.2 | + 晶粒度独立模块 | modes/grain/ 零侵入式扩展 |

## 打包踩坑：ICU DLL 缺失

**现象**：PyInstaller 打包后 exe 启动崩溃。

**诊断过程**：
1. 手写 PE 解析器检查 DLL 导入表（`check_icu.py`）
2. 对比 conda 环境与打包产物中 DLL 的 MD5（`compare_dlls.py`）
3. 解析 Qt6Core.dll 导入依赖链（`check_imports.py`）

**结论**：
- `icuuc.dll` 哈希一致 ✓
- `icuin/icudt/icuio/icutu` 四个 DLL 在打包产物中 **MISSING** ✗
- 根因：conda 环境 DLL 与 PyInstaller 收集逻辑不一致

**当前缓解**：`runtime_hook.py` 运行时添加 DLL 目录 + PATH 兜底。**根治方案**：用干净 pip 环境替代 conda 打包。

## 已解决的 26 个 Bug（精选）

| Bug | 根因 | 解决方案 |
|-----|------|----------|
| Undo Mask 内存爆炸 | 全量 Mask 存储 | Command 模式 + MaskDiff 增量 |
| 涂抹卡死 | 逐像素循环 | NumPy 向量化 + 延迟记录 |
| 切图残留/幽灵 Mask | 引用未完全清理 | `_clear_all_refs()` 防御 + clear() 统一清空 |
| linestrip 泄漏为 polygon | `load_split()` 不区分 shape_type | 按 shape_type 分流加载 |
| 空标注覆写已有 JSON | 无保护写入 | 无新标注不写文件 |
| AI 输入尺寸不匹配 | 硬编码 input_size | ONNX 自动读取 input_size |

## 关键发现

- **Mask 增量存储是正确选择**：全量 Mask 每次 undo 存一份完整数组，50 步就是 50×数十MB = 爆炸。MaskDiff 只存变化区域，内存可控。
- **双轨栈撤销有 bug**：Segment 和 Mask 两个独立 undo 栈交替操作时顺序错乱。统一时间轴单栈彻底解决。
