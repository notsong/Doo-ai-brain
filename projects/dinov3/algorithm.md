# DINOv3 — 算法设计

> 算法设计、模型架构和技术方案。

## 整体方案

采用"大模型自监督特征 + 轻量定制解码器"路线。冻结 DINOv3 ViT-B/16 提取多尺度特征，不修改 backbone 权重（第一阶段），专注训练解码器适配晶界分割任务。

## 模型架构

```
输入 1024×1024×3
  ↓
DINOv3 ViT-B/16 Encoder (冻结)
  │ 取最后 3 层 hidden_states
  │ 去 CLS(1) + register tokens(4)
  │ 全部 reshape 为 [B, 768, 73×73]
  ├─ f4 (low)  ─┐
  ├─ f8 (mid)  ─┼─▶ AttentionFusion
  └─ f16(high) ─┘    (逐尺度 CBAM + 可学习 softmax 加权 + 融合卷积)
  ↓
1×1 Projection (768→512, GroupNorm+GELU)
  ↓
Residual Decoder ×4 (512→256→128→64→32)
  ↓
├─ SegHead (1×1 conv → 2ch)     → seg logits
└─ BoundaryHead (3层 conv → 1ch) → boundary logits (辅助监督)
```

### 关键设计

- **三尺度同分辨率**：ViT patch=14, 1024≈73×14，三尺度均 reshape 到相同分辨率
- **AttentionFusion**：各尺度独立 CBAM 后，可学习 softmax 权重加权相加 + 双层 3×3 融合卷积
- **Boundary Head**：Laplacian 核 `[[-1,-1,-1],[-1,8,-1],[-1,-1,-1]]` 从 GT 提取边缘做 BCE，强化细线连续性

## 训练策略

1. **两阶段训练**：epoch 0-59 冻结 backbone（lr=1e-4），epoch 60 起解冻（lr=1e-5）重建 optimizer+scheduler
2. **AMP 混合精度**：GradScaler, grad_clip=1.0
3. **优化器**：AdamW + WarmupCosine 学习率衰减
4. **损失**：TotalLoss = 0.25×CE + 0.4×Dice + 0.15×BoundaryBCE
5. **数据增强**：翻转/旋转90/Affine/Elastic/亮度对比度/高斯噪声
6. **保存策略**：best_iou, best_dice, last

## 推理与后处理

- **小图**（≤1024）：reflect pad 直接推理
- **大图**（>1024）：滑窗 1024×768, stride=768, 计数矩阵归一化消除拼缝
- **后处理**：形态学闭运算（核 3）+ findContours 去碎点（<150px，比 connectedComponents 快 40 倍）
- **FP16 全流程**：含归一化常量 half

## 检索子系统

- **Pooling**：GeM (p=3) + L2 归一化 → 768-dim embedding
- **匹配**：Cosine 相似度 TopK
- **增强**：五分类裁剪提鲁棒性
- **评估**：benchmark.py → Top1/3/5 + 时延 CSV
- **可视化**：UMAP 降维

## 已知局限

1. ViT-B + 1024×1024 + batch 4 显存需求 ≥12GB
2. 文档落后于代码：README 描述旧版 768 FPN-UNet，实际已升级为 V3 AttentionFusion + Residual Decoder
3. config 中 `bce_pos_weight`、`connectivity_weight` 定义后未使用
