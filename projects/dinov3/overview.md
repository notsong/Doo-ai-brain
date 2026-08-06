# DINOv3 晶界分割与检索 — 项目概览

> 基于 DINOv3 自监督大模型的金相晶界语义分割 + 晶粒度图谱检索系统。

## 项目简介

利用 Meta DINOv3 (ViT-B/16) 自监督预训练特征，构建轻量定制解码器，实现金相显微图像中晶界边缘的自动提取（1-3 像素宽细线目标），同时复用同一 backbone 做晶粒度标准图谱检索，辅助评级。

已集成到检测系统 `1001_025_dinov3` 部署使用。

## 目标

- 从金相图像中自动提取完整、连续的晶界网络
- 利用 DINOv3 语义特征进行晶粒度级别（GB/T 6394 图谱 0~10 级）检索
- 双引擎部署（PTH + ONNX），支持生产环境

## 技术栈

- **Backbone**: DINOv3 ViT-B/16 (patch=14, 768-dim)
- **解码器**: AttentionFusion (CBAM + 可学习加权) + Residual Decoder 四级上采样
- **辅助监督**: Boundary Head (Laplacian 边缘提取 + BCE)
- **损失**: CE + Dice + Boundary 三合一
- **部署**: ONNX Runtime (CUDA+CPU 双 Provider), opset 17
- **检索**: GeM Pooling (p=3) + L2 Norm + Cosine TopK

## 当前状态

🟢 已部署。checkpoint `best_iou0724.pth` 用于推理，ONNX 模型 `model_1024.onnx` 用于生产。

## 关键文件与入口

| 文件 | 作用 |
|------|------|
| `config.py` | 唯一配置入口（dataclass），image_size=1024 |
| `train.py` | 两阶段训练（冻结→解冻），AMP + AdamW + WarmupCosine |
| `infer_v3.py` | PTH 推理：≤1024 直推，>1024 滑窗 1024×768 |
| `pred_dino.py` | 生产部署类：ONNX+PTH 双引擎，5 种输出格式 |
| `export_onnx.py` | ONNX 导出 + 数值验证 |
| `models/` | dinov3_encoder, attention, decoder_v3, boundary_head |
| `losses/loss.py` | TotalLoss = 0.25CE + 0.4Dice + 0.15Boundary |
| `retrieval/` | 检索子系统（extractor, gallery, matcher, benchmark） |

## 数据集

- **JLD**: train 279 + val 71 张（1024×1024，晶界标注）
- **莱钢 (LaiGang)**: train 72 + val 18 张（从大图切 1024 瓦片生成）
- **检索图库**: GB/T 6394 标准图谱 5 张，预计算 GeM embedding

## 参考资料

- DINOv3: https://github.com/facebookresearch/dinov3
- 检测系统配置: `1001_025_dinov3.json`
