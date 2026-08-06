# DINOv3 — 经验教训

> 非显而易见的坑和可复用模式。

## 踩过的坑

### ViT 输入分辨率对齐
Patch=14，输入尺寸必须接近 14 倍数。选 1024≈73×14 保证整除，不 resize、不 letterbox、不 TTA，用滑窗保持训练-推理一致性。否则 ViT 内部 reshape 会越界。

### 细线目标（1-3 像素宽）分割困难
晶界仅占 1-3% 像素，CE+Dice 组合容易产生模糊断裂。加入 Boundary Head（Laplacian 边缘监督）和形态学后处理（闭运算+去碎点）后显著改善。

### NumPy 跨版本 checkpoint 不兼容
`numpy.object` → `numpy.dtypes.ObjectDType`：训练环境 NumPy 2.x，推理环境 NumPy 1.x 时 monkey-patch 解决。

### config 与 loss 实际权重不一致
需对照代码确认实际超参数，不要只看 config 注释。README 中的 0.4/0.4/0.2 与实际代码 0.25/0.4/0.15 不符。

## 可复用模式

### "大模型 + 轻量解码器"范式
冻结 ViT backbone + 只训练轻量解码器 + 后处理管线，见效快、显存可控。适用于小样本工业场景。

### 滑窗重叠平均拼接
crop × stride，计数矩阵归一化消除拼缝伪影——可复用到任何大图分割任务。

### 双引擎部署（PTH + ONNX）
PTH 用于调试和验证，ONNX 用于生产推理。额外加数值校验（diff<1e-4）确保导出正确。

### findContours > connectedComponents
去碎点用 `findContours(cv2.RETR_EXTERNAL)` 比 `connectedComponents` 快约 40 倍。

## 环境问题

- CUDA 显存 ≥12GB（ViT-B + 1024² + batch 4）
- ONNX opset 17 需要较新的 ORT 版本
- DINOv3 本地权重路径：`D:/dinov3_model`
