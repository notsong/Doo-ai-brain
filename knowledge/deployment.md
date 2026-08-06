# 部署知识

> 模型部署、打包和生产环境模式。

## 常用模式

### 检测系统集成
模型通过 JSON 配置文件注册到检测系统：
```json
{
  "model_name": "1001_025_dinov3.pth",
  "input_size": 1024,
  "stride": 768,
  "threshold": 0.5,
  "postprocess": { "close_kernel": 3, "min_area": 150 }
}
```
检测系统读取配置 → 加载模型 → 推理 → 后处理 → 输出 mask/JSON/灰度图。

### PySide6 桌面应用打包
```
PyInstaller + .spec + runtime_hook.py + hiddenimports
```
关键配置：
- `collect_all('PySide6')` 收集全部 Qt DLL
- `collect_all('shiboken6')` 收集绑定
- `runtime_hook.py` 运行时 `os.add_dll_directory()` 解决 DLL 搜索路径

## 工具

| 工具 | 用途 |
|------|------|
| ONNX Runtime | 模型推理（CPU/CUDA） |
| PyInstaller | Python 应用打包为独立 exe |
| Docker | 服务端部署（待引入） |

## 踩过的坑

### Git 推送被墙（GFW）
```bash
git config --global http.proxy http://127.0.0.1:7897
git config --global https.proxy http://127.0.0.1:7897
```
Clash Verge 默认端口 7897。验证：`curl -x http://127.0.0.1:7897 https://github.com`

### gh CLI 认证所需 Scope
最小 Token scope：`repo`, `read:org`, `workflow`

### PyInstaller + conda = DLL 地狱
conda 的 ICU/MSVC DLL 与 PyInstaller 的依赖收集不一致，导致打包后 DLL 缺失或版本冲突。**根治方案**：用干净 pip venv 打包，不用 conda。

### ICU DLL 缺失诊断
手写 PE 解析器检查导入表，对比 conda 与 dist 中 DLL 的 MD5。关键检查项：icuuc, icuin, icudt, icuio, icutu 五个 DLL。

## PyInstaller 笔记

### Spec 文件要点
```python
a = Analysis(
    ['doo_label/main.py'],
    pathex=[],
    binaries=[],
    datas=[],
    hiddenimports=['skimage', 'cv2', 'pyside6', 'shiboken6'],
    hookspath=[],
    runtime_hooks=['runtime_hook.py'],
)
```

### 减小打包体积
- 排除不需要的 Qt 模块（WebEngine, Multimedia 等）
- onedir 模式（非 onefile）方便排障
- 不要打包整个 conda 环境

## 参考资料

- PyInstaller 文档: https://pyinstaller.org/
- GitHub CLI: https://github.com/cli/cli
