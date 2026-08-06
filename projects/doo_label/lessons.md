# Doo Label — 经验教训

> PySide6 桌面应用开发和 PyInstaller 打包的核心经验。

## 踩过的坑

### Qt Widget 所有权是单一的
一个 widget 只能有一个 parent。放到 QStackedWidget 又放到 QTabWidget 会造成 reparent（被第二个抢走），第一个容器变成空槽。画 widget 树图比凭记忆写布局重要。

### Qt 信号在构造期间就触发
`QTabWidget.addTab()` 等操作在 `__init__` 中就会触发 `currentChanged` 信号，此时依赖对象可能还没创建。始终用 `hasattr` 守卫或调整 `__init__` 顺序。

### PyInstaller + conda = DLL 地狱
conda 的 ICU/MSVC DLL 与 PyInstaller 的依赖收集逻辑不一致。根治方案是用干净 pip venv 打包，而非 conda。

## 可复用模式

### MVC 四层 + 独立模块扩展
- domain（纯数据，零依赖）→ engine（纯算法）→ application（用例）→ ui
- 新功能作为独立 modes/ 模块，不侵入现有代码
- 增量信号（非全量刷新）驱动 UI 更新

### 状态机：State × Tool 二维正交
5 个 State 永不新增，新 Tool 只需加到 Tool 维度。二维表确保交互行为可预测。

### Command 模式 + 统一时间轴撤销
- 所有可撤销操作实现 Command 接口（execute/undo）
- 单栈统一管理，避免多栈交替操作顺序错乱
- Mask 用 Diff 增量存储控制内存

### 增量验证（骨架→内容→信号）
>3 文件或 >100 行的重构不要一次写完。骨架阶段先跑通，内容阶段填充，信号阶段接线。两个 trivial bug（widget reparenting, signal ordering）在骨架阶段 30 秒就能发现，但 300 行全写完再跑就花了数小时排查。

## 环境问题

- PySide6 ≥ 6.5 需要 Qt 6.5+ DLL
- 打包产物需附带 `runtime_hook.py` 处理 DLL 搜索路径
- 注意 `shiboken6` 的 `collect_all` 在 PyInstaller 中的配置
