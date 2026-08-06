# PyTorch 知识

> 从项目中积累的 PyTorch 模式、技巧和坑。

## 常用模式

### 冻结-解冻两阶段训练
工业小样本标配：先冻结 backbone 训 decoder（lr=1e-4, 50~60 epoch），再解冻微调（lr=1e-5, 40~50 epoch）。防过拟合效果好。

```python
# 冻结
model.freeze_backbone()
optimizer = Adam(model.parameters(), lr=1e-4)
# 解冻
model.unfreeze_backbone()
optimizer = Adam(model.parameters(), lr=1e-5)  # 重建 optimizer
```

### AMP 混合精度
全流程包括归一化常量都转 half：

```python
scaler = GradScaler()
with autocast():
    loss = model(x, y)
scaler.scale(loss).backward()
scaler.step(optimizer)
scaler.update()
```

### 学习率调度
WarmupCosine 比 Step/Exponential 更平滑，分割任务中不容易震荡。

## 性能技巧

### 在线滑窗训练 > 离线切块
- 省磁盘空间（不存切块图片）
- 切块参数可动态调整
- 过滤无意义背景块（min_fg_ratio）

### findContours > connectedComponents
去碎点场景：`cv2.findContours(RETR_EXTERNAL)` 比 `scipy.ndimage.label` 快约 40 倍。

## 踩过的坑

### 标签值必须从 0 开始连续
VOC 格式要求 `_background_=0, class1=1, ...`。网上数据集常把目标标 255（白）而非 1——不检查直接训练，模型无效。始终在标注转换脚本中校验。

### NumPy 版本跨环境不兼容
`numpy.object` (v1) → `numpy.dtypes.ObjectDType` (v2)：训练环境 NumPy 2.x，推理环境 NumPy 1.x 时 checkpoint 加载报错。Monkey-patch 或统一版本。

### checkpoint 体积
含 optimizer state 的 `.pth` 可超 1GB。部署用 `--slim` 去 optimizer 后 ~400MB。保存 best 而非每个 epoch。

### 文档与代码不一致
config 写了 `dice_weight=0.4` 但代码可能实际用了不同值。README 可能是旧版架构的描述。始终以代码为准。

## 版本记录

- PyTorch 1.x → 2.x：AMP API 变化（`torch.cuda.amp`）
- TorchVision 0.15+：预训练权重 API 改变
- NumPy 1.x → 2.x：类型系统重构，checkpoint 不兼容
