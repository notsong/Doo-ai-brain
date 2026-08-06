# OpenCV 知识

> 从项目中积累的 OpenCV 模式、技巧和坑。

## 常用模式

### 细线后处理管线（金相晶界专用）
```
骨架化 → 剪枝（去毛刺） → 形态学闭合（桥接 <10px 断裂） → 膨胀（恢复线宽）
```
代码见 `doo_label/engine/grain_postprocess.py`。

### TE-VVP 晶界网络协议
```
分割 mask → 骨架化 → 分水岭补全 → 端点验证 → 网络合成
```
用于修复分割模型输出的拓扑断裂。

## 性能技巧

### findContours + RETR_EXTERNAL
去碎点/计数场景比 `connectedComponents` 快约 40 倍：
```python
contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
valid = [c for c in contours if cv2.contourArea(c) > min_area]
```

### 双阈值"带通滤波"二值化
金相反光/偏光场景：低阈值过滤暗噪声 + 高阈值过滤基体反光，取中间范围。
见 `Blog/word/清洁度颗粒测量-传统CV知识点总结.md`。

## 踩过的坑

### 色彩空间
`cv2.imread` 默认 BGR，`QImage` 期望 RGB。转换遗漏导致颜色错误或推理结果异常。

### RETR_TREE vs RETR_EXTERNAL
- `RETR_EXTERNAL`：只外层轮廓（孔被忽略）
- `RETR_TREE`：全层级（孔洞单独出轮廓）
清洁度测量用 `RETR_EXTERNAL` 避免孔洞干扰夹杂物计数。

### Feret 直径需要凸包
直接对凹多边形算 Feret（卡规直径）结果偏小。先 `cv2.convexHull()` 再 `cv2.minAreaRect()`。

### mask 坐标保持全局
涉及 ROI 操作时，mask 坐标很容易"漂"到 ROI 局部坐标系。始终在全局坐标系下操作。

## 平台差异

- Windows：OpenCV 4.8+ 对中文路径支持改善但仍建议避免
- Linux：`cv2.imshow` 需要 GUI 后端（`apt install libgtk2.0-dev`）

## 参考资料

- OpenCV 官方文档: https://docs.opencv.org/4.x/
- scikit-image 骨架化: `skimage.morphology.skeletonize`
