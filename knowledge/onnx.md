# ONNX 知识

> ONNX 模型导出、优化和部署模式。

## 常用模式

### 双引擎部署（PTH + ONNX）
PTH 调试验证 → ONNX 导出 → 生产 ORT 推理：

```python
class Model:
    def __init__(self, pth_path, onnx_path):
        self.pth = load_pth(pth_path)    # 调试用
        self.ort = ort.InferenceSession(onnx_path)  # 生产用
    
    def predict(self, x, engine='onnx'):
        if engine == 'pth':
            return self._predict_pth(x)
        return self._predict_onnx(x)
```

### ONNX 导出后数值验证
导出后喂同一输入对比 PTH 和 ONNX 输出，`diff < 1e-4` 才放行：

```python
pth_out = model(input)
ort_out = session.run(None, {'input': input.numpy()})
assert np.abs(pth_out - ort_out).max() < 1e-4
```

### Dynamic Batch
固定 spatial dims，batch 维度设 dynamic：
```python
dynamic_axes = {'input': {0: 'batch'}, 'output': {0: 'batch'}}
```

## 性能技巧

### Provider 选择
- **开发/调试**：CPUExecutionProvider（省显存、方便断点）
- **生产推理**：CUDAExecutionProvider + CPUExecutionProvider 双 Provider 兜底

### Opset 选择
- **opset 12**：兼容性好，大部分 ORT 版本支持
- **opset 17**：需要较新 ORT（≥1.14），支持更多算子优化

## 踩过的坑

### ONNX 输出顺序
多输出模型（如 seg + boundary），ORT 按字母序排列输出名。代码要按名字取而非按位置：
```python
out_seg = session.run(['seg'], ...)  # 按名取，不要 out[0]
```

### ONNX 输入尺寸
推理时 ONNX 自动报告的 `input_size` 可能与训练 config 不同。始终从 session 动态读取：
```python
input_shape = session.get_inputs()[0].shape  # [B, 3, H, W]
```

### CUDA Provider 加载失败
无 GPU 或 CUDA 版本不匹配时，ORT 静默回退到 CPU。显式检查 provider 列表。

## 部署笔记

- 模型文件 `.onnx` 约 95-99MB（U-Net 1024 输入），需随 exe 打包或放固定路径
- ORT 依赖 `onnxruntime` 或 `onnxruntime-gpu`，PyInstaller 打包注意 hiddenimport

## 参考资料

- ONNX opset 文档: https://github.com/onnx/onnx/blob/main/docs/Operators.md
- ORT 文档: https://onnxruntime.ai/docs/
