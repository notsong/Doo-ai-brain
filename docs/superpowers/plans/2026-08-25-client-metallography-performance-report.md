# 客户金相 AI 性能验证报告 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 生成一份简洁的客户 Word 验收/性能验证报告，只展示 7 个验证单元的 IoU、Precision（P）和 Recall（R）三项指标、模拟数据和简单图表。

**Architecture:** 一个 Python 脚本生成固定的 7 组模拟汇总数据和 4 张简单柱状图，另一个脚本将数据、图表和客户化文字组装为 Word。报告不包含逐图数据、标准差、分位数、箱线图、热力图或复杂附录。

**Tech Stack:** bundled Python 3、pandas、matplotlib、python-docx、pytest、Documents skill 的 `render_docx.py`。

---

## 文件结构

- Create: `scripts/create_metallography_report.py` — 生成模拟数据、4 张图表和 Word 报告。
- Create: `tests/test_metallography_report.py` — 验证 7 组数据、指标阈值、图表和 DOCX 结构。
- Create: `output/AI金相检测项目性能验证报告_模拟数据.docx` — 工作区构建结果。
- Deliver: `E:\company\word\BG\final\AI金相检测项目性能验证报告_模拟数据.docx` — 客户报告交付副本。

## 固定模拟数据

| 项目 | 验证单元 | IoU | P | R |
|---|---|---:|---:|---:|
| 夹杂物 | 条状 | 0.912 | 0.944 | 0.968 |
| 夹杂物 | 点状 | 0.925 | 0.956 | 0.973 |
| 晶粒度 | 晶界 | 0.901 | 0.932 | 0.961 |
| 脱碳层 | 镶嵌-全脱 | 0.934 | 0.962 | 0.978 |
| 脱碳层 | 镶嵌-总脱 | 0.918 | 0.948 | 0.969 |
| 脱碳层 | 非镶嵌-全脱 | 0.907 | 0.939 | 0.964 |
| 脱碳层 | 非镶嵌-总脱 | 0.896 | 0.928 | 0.958 |

阈值固定为 IoU ≥ 0.85、P ≥ 0.90、R ≥ 0.95。全部数字必须标注为“模拟验证数据”，不得写成真实项目测试结论。

### Task 1: 实现数据、图表和基础测试

**Files:**
- Create: `scripts/create_metallography_report.py`
- Test: `tests/test_metallography_report.py`

- [ ] **Step 1: 写失败测试**

```python
def test_summary_has_seven_units_and_three_metrics():
    summary = build_summary()
    assert len(summary) == 7
    assert {"project", "label", "iou", "precision", "recall"} <= set(summary.columns)
    assert (summary["iou"] >= 0.85).all()
    assert (summary["precision"] >= 0.90).all()
    assert (summary["recall"] >= 0.95).all()

def test_charts_are_created(tmp_path):
    summary = build_summary()
    paths = create_charts(summary, tmp_path)
    assert {p.name for p in paths} == {
        "overview_metrics.png", "inclusion_metrics.png",
        "grain_boundary_metrics.png", "decarb_metrics.png",
    }
```

- [ ] **Step 2: 运行测试确认失败**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py -q`

Expected: FAIL because `build_summary` and `create_charts` do not yet exist.

- [ ] **Step 3: 实现最小数据和图表**

实现以下接口，并让 `build_summary()` 返回固定的 7 行数据：

```python
THRESHOLDS = {"iou": 0.85, "precision": 0.90, "recall": 0.95}

def build_summary() -> pd.DataFrame: ...
def create_charts(summary: pd.DataFrame, chart_dir: Path) -> list[Path]: ...
def build_report(output_dir: Path) -> Path: ...
```

`create_charts()` 生成 4 张简单分组柱状图：总览 1 张、夹杂物 1 张、晶粒度 1 张、脱碳层 1 张。统一三位小数、同一套 IoU/P/R 配色，并标注“模拟验证数据”。

- [ ] **Step 4: 运行数据和图表测试**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py::test_summary_has_seven_units_and_three_metrics tests/test_metallography_report.py::test_charts_are_created -q`

Expected: PASS，并生成 4 张 PNG。

### Task 2: 组装简洁客户 Word 报告

**Files:**
- Modify: `scripts/create_metallography_report.py`
- Modify: `tests/test_metallography_report.py`

- [ ] **Step 1: 添加 DOCX 结构测试**

```python
def test_docx_contains_required_sections(tmp_path):
    output = build_report(tmp_path)
    doc = Document(output)
    text = "\n".join(p.text for p in doc.paragraphs)
    for heading in ("执行摘要", "验证方法", "夹杂物性能验证", "晶粒度性能验证", "脱碳层性能验证", "总体验收结论"):
        assert heading in text
    assert len(doc.tables) >= 4
    assert len(doc.inline_shapes) == 4
```

- [ ] **Step 2: 实现 Word 内容**

创建 A4 Word，章节仅包括：封面与模拟数据声明、项目范围、执行摘要、验证方法、夹杂物性能验证、晶粒度性能验证、脱碳层性能验证、总体验收结论。每个项目放一张指标表和一张对应柱状图；总览放一张汇总表和总览图。指标定义只保留 `IoU`、`P = TP/(TP+FP)`、`R = TP/(TP+FN)`。

表格列固定为“验证单元、IoU、P、R、是否达到阈值”，数值保留三位小数。结论统一写明“模拟数据结果显示”，不加入逐图数据、标准差、热力图、箱线图或算法研发细节。

- [ ] **Step 3: 运行 DOCX 结构测试并生成报告**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe -m pytest tests/test_metallography_report.py -q`

Expected: all tests PASS and `output/AI金相检测项目性能验证报告_模拟数据.docx` exists.

### Task 3: 渲染验证与交付

**Files:**
- Modify: `tests/test_metallography_report.py`
- Deliver: `E:\company\word\BG\final\AI金相检测项目性能验证报告_模拟数据.docx`

- [ ] **Step 1: 渲染 DOCX 并逐页检查**

Run: `C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe C:\Users\ssong\.codex\plugins\cache\openai-primary-runtime\documents\26.819.11345\skills\documents\render_docx.py output/AI金相检测项目性能验证报告_模拟数据.docx --output_dir output/rendered --emit_pdf`

检查封面、表格、四张图表和结论页，确认无文字截断、图表溢出、表格重叠或字体缺失。若环境缺 LibreOffice，做 DOCX 结构检查并明确说明未完成渲染 QA。

- [ ] **Step 2: 复制交付副本**

将工作区 DOCX 复制到 `E:\company\word\BG\final\AI金相检测项目性能验证报告_模拟数据.docx`，不覆盖原有脱碳层报告。

- [ ] **Step 3: 提交脚本和计划**

```powershell
& 'C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\native\git\cmd\git.exe' add scripts tests docs/superpowers/plans/2026-08-25-client-metallography-performance-report.md
& 'C:\Users\ssong\.cache\codex-runtimes\codex-primary-runtime\dependencies\native\git\cmd\git.exe' commit -m "feat: 生成简洁金相指标验证报告"
```

## 自检

- 只保留 IoU、P、R 三个指标。
- 覆盖夹杂物 2 组、晶粒度 1 组、脱碳层 4 组，共 7 组。
- 只生成 4 张简单柱状图和必要表格。
- 明确所有数据为模拟数据，不虚构真实验收结果。
