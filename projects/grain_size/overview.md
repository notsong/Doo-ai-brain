# 晶界/金相组织分割 (U-Net) — 项目概览

> 基于 U-Net 的多项目金相图像语义分割系统，覆盖晶界、双相钢、夹杂物、缺陷检测等工业场景。

## 项目简介

基于开源项目 `bubbliiiing/unet-pytorch` 深度改造，专门用于钢铁材料金相显微图像的自动分割与定量分析。一套代码支持多个子项目：晶界孪晶分割、双相钢组织分割、螺纹缺陷检测、夹杂物评级。

## 目标

- 金相图像中晶界/孪晶界、组织、缺陷的自动分割
- 支持多数据集、多分辨率、多骨干切换
- 全链路：labelme JSON → VOC 格式 → 训练 → ONNX 导出 → 推理后处理

## 技术栈

- **框架**: PyTorch
- **骨干网络**: VGG16 / ResNet50（可切换）
- **架构**: U-Net（Encoder-Decoder + Skip Connection）
- **损失函数**: CE, Focal, Dice, Tversky, Boundary, Connectivity
- **部署**: ONNX Runtime (opset 12)
- **后处理**: OpenCV（骨架化、分水岭、形态学）

## 当前状态

🟢 多项目并行维护中。最新方向：滑窗在线训练 (`train_SX_sliding.py`)、细线专用网络 (`UNet_Thin`)、TE-VVP 晶界网络后处理协议。

## 关键文件与入口

| 文件 | 作用 |
|------|------|
| `unet.py` | 主模型类（训练/推理/ONNX导出六合一） |
| `nets/unet.py` | U-Net 网络定义（VGG/ResNet 骨干） |
| `nets/unet_training.py` | 损失函数集合 |
| `train_SX_sliding.py` | 最新训练脚本（滑窗在线切块） |
| `predict.py` | 统一推理入口（6 种模式） |
| `utils/sliding_window.py` | 滑窗推理 + 重叠融合 |
| `utils/dataloader_sliding.py` | 滑窗训练数据加载 |
| `voc_annotation.py` | labelme JSON → VOC 格式转换 + 9:1 划分 |

## 数据集

| 数据集 | 数量 | 类别 | 来源 |
|--------|------|------|------|
| JLD | 222 张 | 背景 + 晶界(jj) + 孪晶界(lj) | 铂力特 BLT / 哈汽轮 HQL |
| SX | 572 张 | 背景 + 组织 (2类) | 莱钢 EX-1 双相钢 |
| HTJG | 88 张 (192² 切块) | 背景 + 缺陷(qx) | 螺纹缺陷检测 |
| JZW | 外部目录 | 多类夹杂 | 夹杂物评级 |

## 参考资料

- 原库: https://github.com/bubbliiiing/unet-pytorch
- 标注数据在 `E:\AIProject\Data\` 下各子目录
