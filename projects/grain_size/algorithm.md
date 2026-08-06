# 晶界/金相组织分割 (U-Net) — 算法设计

> 算法设计、模型架构和技术方案。

## 整体方案

标准 U-Net Encoder-Decoder + Skip Connection 架构，支持 VGG16 和 ResNet50 两种骨干。针对金相图像特点做了多项定制：细线专用网络、滑窗训练、TE-VVP 晶界后处理协议。

## 模型架构

```
输入 (batch, 3, H, W)
  ↓
Encoder: VGG16 / ResNet50
  │ 返回 5 层特征图
  │ VGG:  [192, 384, 768, 1024]
  │ ResNet: [192, 512, 1024, 3072]
  ↓
Decoder: 4× unetUp 块
  │ 双线性上采样 + Skip Concat
  │ 两层 3×3 Conv + ReLU
  ↓
1×1 Conv → num_classes
```

### 自定义变体

| 网络 | 用途 | 特点 |
|------|------|------|
| `UNet_Thin` | 细线目标（晶界） | 非对称卷积(11×1+1×11)、ASPP、转置卷积 |
| `UNet_FPN` | 多尺度目标 | ASPP + FPN 解码器 + 深度监督 |
| `UNet_Shallow` | 轻量推理 | 4 层浅编码器 + DSN |
| `UNet_Attention` | 注意力增强 | Attention Gate 跳跃连接 |

## 训练策略

### 标准流程
1. **冻结阶段**（epoch 0-49）：冻结 backbone，只训 decoder，lr=1e-4
2. **解冻阶段**（epoch 50-99）：解冻全部，lr=1e-5 低学习率微调
3. **优化器**：Adam + Cosine 学习率衰减

### 滑窗训练（最新）
- `train_SX_sliding.py`：大图在线切块，crop=512², stride=400
- `min_fg_ratio=0.001`：过滤前景占比过低的无意义背景块
- 优势：不需预先切块、不占用额外磁盘空间、训练时可动态调整

### 损失函数
- **标准**：Focal Loss (α=0.5, γ=2) + Dice Loss
- **类别不平衡**：`cls_weights`（逆频率/中位数频率/对数平滑三种计算方式）
- **细线专用**：BoundaryLoss（拉普拉斯边界）、ConnectivityLoss（连通性）、WeightedTverskyLoss

## 推理与后处理

### 滑窗推理
- `utils/sliding_window.py`：滑窗 + 重叠区域平均融合
- 192² 分块推理后拼接还原全图（SX）

### TE-VVP 晶界后处理协议
```
jj mask → 骨架化 → 剪枝 → 分水岭补全断裂 → 端点验证 → 晶界网络合成
```
专门修复细线分割的拓扑断裂问题。

## 已知局限

1. 多套训练脚本代码重复度高，维护成本大
2. 细线目标分割仍不够完美，依赖后处理管线补全
3. 各子项目 log 和模型分散在不同目录
