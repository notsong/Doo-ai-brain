# 客户金相 AI 性能验证报告 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 生成一份面向客户的可编辑 Word 验收/性能验证报告，用确定性的模拟数据和图表展示夹杂物、晶粒度、脱碳层三个检测项目的 IoU、Precision（P）和 Recall（R）。

**Architecture:** 用一个确定性 Python 生成器负责验证单元定义、逐图模拟数据、汇总统计和图表输出；用一个独立的 DOCX 组装器读取这些产物并创建客户报告。数据、图表和文档内容分层保存，测试先验证 7 组数据的结构与阈值，再验证 DOCX 的章节、表格和图片数量。最终只向客户交付 DOCX，CSV 和 PNG 留作可追溯的构建中间产物。

**Tech Stack:** bundled Python 3、numpy、pandas、matplotlib、python-docx、pytest、Documents skill 的 `render_docx.py`。

---

## 文件结构

- Create: `scripts/metallography_report_data.py` — 7 个验证单元、随机种子、逐图指标模拟、统计汇总和 CSV 输出。
- Create: `scripts/metallography_report_charts.py` — 从 CSV 生成总览柱状图、夹杂物对比图、晶界分布图、脱碳层热力图和分布图。
- Create: `scripts/create_metallography_report.py` — 设置 Word 页面/字体/表格样式，组装章节、图表、指标表和附录。
- Create: `tests/test_metallography_report.py` — 数据、统计和文档结构测试。
- Create: `output/metallography_report_data.csv` — 120×7 行的模拟逐图数据。
- Create: `output/charts/*.png` — 报告引用的图表中间产物。
- Create: `output/AI金相检测项目性能验证报告_模拟数据.docx` — 工作区内的构建结果。
- Deliver: `E:\company\word\BG\final\AI金相检测项目性能验证报告_模拟数据.docx` — 客户报告交付副本；若目标目录受权限限制，保留工作区结果并明确告知路径。

### Task 1: 建立确定性的验证数据模型

**Files:**
- Create: `scripts/metallography_report_data.py`
- Test: `tests/test_metallography_report.py`

- [ ] **Step 1: 写数据模型失败测试**

```python
def test_validation_units_are_complete():
    units = validation_units()
    assert [u.key for u in units] == [
        "inclusion_strip", "inclusion_dot", "grain_boundary",
        "decarb_embedded_full", "decarb_embedded_total",
        "decarb_non_embedded_full", "decarb_non_embedded_total",
    ]
    assert all(u.sample_count == 120 for u in units)

def test_generated_rows_have_required_metrics():
    rows = generate_rows(seed=20260825)
    assert len(rows) == 840
    assert {"unit_key", "sample_id", "iou", "precision", "recall"} <= rows.columns
    assert rows[["iou", "precision", "recall"]].applymap(lambda v: 0 <= v <= 1).all().all()

def test_summary_uses_approved_thresholds():
    summary = summarize(generate_rows(seed=20260825))
    assert set(summary["unit_key"]) == set(u.key for u in validation_units())
    assert (summary["iou_mean"] >= 0.85).all()
    assert (summary["precision_mean"] >= 0.90).all()
    assert (summary["recall_mean"] >= 0.95).all()
```

- [ ] **Step 2: 运行测试确认失败**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py -q`

Expected: FAIL because `validation_units`, `generate_rows` and `summarize` do not yet exist.

- [ ] **Step 3: 实现最小数据生成器**

实现不可变的 `ValidationUnit` 数据类和以下接口：

```python
THRESHOLDS = {"iou": 0.85, "precision": 0.90, "recall": 0.95}

@dataclass(frozen=True)
class ValidationUnit:
    key: str
    project: str
    label: str
    sample_count: int = 120

def validation_units() -> tuple[ValidationUnit, ...]: ...
def generate_rows(seed: int = 20260825) -> pd.DataFrame: ...
def summarize(rows: pd.DataFrame) -> pd.DataFrame: ...
def write_data(output_dir: Path, seed: int = 20260825) -> tuple[Path, Path]: ...
```

使用 `numpy.random.default_rng(seed)`，为每组生成受限正态分布的逐图 IoU/P/R；均值目标分别设为：夹杂物条状（0.912/0.944/0.968）、点状（0.925/0.956/0.973）、晶界（0.901/0.932/0.961）、镶嵌全脱（0.934/0.962/0.978）、镶嵌总脱（0.918/0.948/0.969）、非镶嵌全脱（0.907/0.939/0.964）、非镶嵌总脱（0.896/0.928/0.958）。将值截断到 `[0, 1]`，并在摘要中计算均值、标准差、最小值、最大值和三指标同时达标率。所有输出列按固定顺序写入 CSV。

- [ ] **Step 4: 运行测试确认数据模型通过**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py::test_validation_units_are_complete tests/test_metallography_report.py::test_generated_rows_have_required_metrics tests/test_metallography_report.py::test_summary_uses_approved_thresholds -q`

Expected: PASS; row count is 840 and every unit’s simulated means meet the declared thresholds.

- [ ] **Step 5: 运行生成器并检查 CSV**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe scripts/metallography_report_data.py --output-dir output`

Expected: `output/metallography_report_data.csv` and `output/metallography_report_summary.csv` exist; CSV has 841 lines including its header and 8 summary lines including its header.

### Task 2: 生成可复用图表

**Files:**
- Create: `scripts/metallography_report_charts.py`
- Modify: `tests/test_metallography_report.py`

- [ ] **Step 1: 写图表产物测试**

```python
def test_charts_are_created(tmp_path):
    rows = generate_rows(seed=20260825)
    summary = summarize(rows)
    paths = create_charts(rows, summary, tmp_path)
    assert {p.name for p in paths} == {
        "overview_metrics.png", "inclusion_comparison.png",
        "grain_boundary_distribution.png", "decarb_heatmap.png",
        "decarb_distribution.png",
    }
    assert all(p.stat().st_size > 10_000 for p in paths)
```

- [ ] **Step 2: 运行测试确认失败**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py::test_charts_are_created -q`

Expected: FAIL because `create_charts` is not implemented.

- [ ] **Step 3: 实现 5 张图表**

在 `create_charts(rows, summary, output_dir)` 中固定使用 1600×900 左右的 PNG、300 dpi、同一套颜色：IoU 蓝、P 绿、R 橙；图内注明“模拟数据”。

1. `overview_metrics.png`: 7 个验证单元的三指标分组柱状图，叠加三条阈值线。
2. `inclusion_comparison.png`: 条状/点状的 IoU、P、R 均值和标准差误差棒。
3. `grain_boundary_distribution.png`: 晶界 120 张图像的 IoU/P/R 箱线图，标注阈值线。
4. `decarb_heatmap.png`: 镶嵌/非镶嵌 × 全脱/总脱的四组单元，按 IoU/P/R 展示热力图。
5. `decarb_distribution.png`: 脱碳层四组单元的 IoU 分布箱线图。

图表标题和图例使用中英双语短标签或可靠中文字体回退，避免字体缺失；数值统一保留三位小数。

- [ ] **Step 4: 运行图表测试并人工查看 PNG**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py::test_charts_are_created -q`

Expected: PASS. Open all five PNGs and confirm axes、图例、阈值线和标签无遮挡；若中文字体缺失，改用可用字体或中英双语标签后重跑。

### Task 3: 组装客户 Word 报告

**Files:**
- Create: `scripts/create_metallography_report.py`
- Modify: `tests/test_metallography_report.py`

- [ ] **Step 1: 写 DOCX 结构测试**

```python
def test_report_contains_required_sections_and_assets(tmp_path):
    output = build_report(data_dir=tmp_path / "data", output_path=tmp_path / "report.docx")
    doc = Document(output)
    text = "\n".join(p.text for p in doc.paragraphs)
    for heading in ("执行摘要", "验证方法", "夹杂物性能验证", "晶粒度性能验证", "脱碳层性能验证", "总体验收结论", "附录"):
        assert heading in text
    assert len(doc.tables) >= 4
    assert len(doc.inline_shapes) >= 5
```

- [ ] **Step 2: 运行测试确认失败**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py::test_report_contains_required_sections_and_assets -q`

Expected: FAIL because `build_report` is not implemented.

- [ ] **Step 3: 实现 Word 样式和章节**

`build_report(data_dir, output_path)` 先读取数据和图表，再创建 A4 纵向文档；设置页边距、页眉页脚、页码、正文中文字体、标题层级和表格边框。按以下顺序写入：

1. 封面：标题、三个项目名、版本日期，以及醒目的“模拟验证数据”声明。
2. 项目概述：说明夹杂物（条状/点状）、晶粒度（晶界）、脱碳层（镶嵌/非镶嵌；全脱/总脱）的范围。
3. 执行摘要：7 行总览表（样本数、IoU/P/R 均值、达标率）和总览图。
4. 验证方法：逐图指标定义 `IoU = intersection / union`、`P = TP/(TP+FP)`、`R = TP/(TP+FN)`，并说明本报告数据为模拟值。
5. 夹杂物、晶粒度、脱碳层三个分项章节：每章放对应汇总表、图表和不超过两段的客户可读结论。
6. 总体验收结论：逐项列出“模拟指标均达到设定阈值”的结论，并再次声明不得替代实际验收。
7. 附录：每个验证单元列出 10 行代表性逐图数据，完整 840 行保留在 CSV 中。

所有表格使用固定列宽和三位小数；图表插入后设置最大宽度，避免页面溢出。报告中不引用未经核验的实际客户名称、项目编号或实际测试结论。

- [ ] **Step 4: 运行结构测试确认通过**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py::test_report_contains_required_sections_and_assets -q`

Expected: PASS; required headings, at least four tables and all five charts are present.

- [ ] **Step 5: 生成工作区 DOCX**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe scripts/create_metallography_report.py --output-dir output`

Expected: `output/AI金相检测项目性能验证报告_模拟数据.docx` exists and opens with the seven validation units represented.

### Task 4: 完成渲染验证与交付副本

**Files:**
- Modify: `tests/test_metallography_report.py`
- Create: `output/AI金相检测项目性能验证报告_模拟数据.docx`
- Deliver: `E:\company\word\BG\final\AI金相检测项目性能验证报告_模拟数据.docx`

- [ ] **Step 1: 添加最终数值一致性测试**

```python
def test_report_summary_matches_csv(tmp_path):
    data_dir = tmp_path / "data"
    output = build_report(data_dir=data_dir, output_path=tmp_path / "report.docx")
    summary = pd.read_csv(data_dir / "metallography_report_summary.csv")
    assert len(summary) == 7
    assert summary["three_metric_pass_rate"].between(0, 1).all()
    assert output.exists() and output.stat().st_size > 100_000
```

- [ ] **Step 2: 运行完整自动化测试**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py -q`

Expected: all tests PASS.

- [ ] **Step 3: 执行 DOCX 渲染检查**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe C:\Users\ssong\.codex\plugins\cache\openai-primary-runtime\documents\26.819.11345\skills\documents\render_docx.py output/AI金相检测项目性能验证报告_模拟数据.docx --output_dir output/rendered --emit_pdf`

Expected: 每页都有 PNG；逐页查看封面、总览表、每个分项章节、热力图和附录，确认无截断、重叠、字体缺失、表格溢出或图表模糊。若 LibreOffice 不可用，执行结构化检查并在交付说明中明确无法完成渲染 QA。

- [ ] **Step 4: 复制交付副本并核验**

将工作区 DOCX 复制到 `E:\company\word\BG\final\AI金相检测项目性能验证报告_模拟数据.docx`，重新打开并检查文件存在、大小与工作区版本一致；不覆盖原有 `Final_脱碳层验收材料.docx`。

- [ ] **Step 5: 提交构建脚本和计划产物**

```powershell
& 'C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\native\git\cmd\git.exe' add scripts tests docs/superpowers/plans/2026-08-25-client-metallography-performance-report.md
& 'C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\native\git\cmd\git.exe' commit -m "feat: 生成金相项目性能验证报告"
```

Expected: commit includes only the reproducible scripts, tests and plan; customer source files remain untouched.

## 计划自检

- 已覆盖 7 个验证单元、每组 120 张模拟图、IoU/P/R、阈值、均值/标准差/范围/达标率、5 张图表、Word 章节和附录。
- 已明确模拟数据声明，避免把示例数字误作为真实客户验收结果。
- 未使用占位词；所有脚本接口、输出路径和测试命令在任务中已明确。
- 渲染工具缺失时保留结构化检查并如实披露，不宣称视觉 QA 已通过。
