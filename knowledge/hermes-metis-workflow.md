# Hermes + Claude Code — Metis C# 协作模式

> 从 Metis 晶粒度项目实战中沉淀的 AI 协作工作流。可复用到任何「Hermes 诊断 + CC 编码」场景。

## 角色分工

| 角色 | 干什么 | 不干什么 |
|------|--------|---------|
| **Hermes（指挥官）** | 读代码、诊断根因、制定方案、写精确指令、验证结果 | 不改代码 |
| **Claude Code（执行者）** | 按指令改代码、跑构建、报结果 | 不做架构决策 |

## 标准流程

```
1. 备份 → 2. 诊断 → 3. 写指令 → 4. 委托 CC → 5. 验证 → 6. 汇报
```

### 1. 备份（无 git 项目必须）

Metis 没有 git 仓库，改动前必须备份：

```bash
mkdir -p D:\work\Metis\_fix_backup_YYYYMMDD
cp file1.cs file2.cs D:\work\Metis\_fix_backup_YYYYMMDD/
```

### 2. 诊断

Hermes 读代码、定位问题、区分「安全修复」和「会改变结果的修复」——后者需要实图验证。

### 3. 写指令（CC 用）

指令要点：
- 精确到文件路径 + 方法名 + 行号范围
- 说明「改什么」和「为什么」
- 标注向后兼容要求（共享模块的参数用可空+默认值）
- 用 `_fix_task_YYYYMMDD.md` 存放，CC 读取后执行

### 4. 委托 CC 执行

```bash
claude -p "$(cat _fix_task_20260805.md)" \
  --allowedTools "Read,Edit,Write" \
  --max-turns 15 \
  --workdir D:\work\Metis
```

CC 的 `ANTHROPIC_AUTH_TOKEN` 走 DeepSeek Anthropic 兼容端点。

### 5. 验证（三步）

| 步骤 | 方法 |
|------|------|
| 读回 | 逐段读改过的代码，确认改动落在预期位置 |
| 构建 | `dotnet build` 能编的模块（共享模块可编，HQL 需 VS） |
| 静态 | 括号配平、lambda 不捕获 ref 参数、关键字符串存在 |

**已知限制**：dotnet CLI 无法编整解决方案——
- `Metis.Adjust` 的 post-build 脚本有 `*Undefined*` 路径 bug
- 旧式 .NET Framework 项目报 AL 任务 MSB4063
- HQL 模块缺依赖 DLL

→ 共享模块用 CLI 验证，HQL/整方案靠 VS。

### 6. 汇报

格式：完成清单表（改动点 + 验证状态）+ 需要用户确认的点 + 备份位置。

## 踩过的坑

### CS1628：ref 参数不能进 lambda

C# 不允许 lambda 捕获 `ref`/`out`/`in` 参数。如果方法签名是 `void Foo(ref Mat disp)`，lambda 里写 `disp.Rows` 编译报错。

**修复**：先把需要的值存到局部变量：
```csharp
int rows = disp.Rows;
int cols = disp.Cols;
var result = list.Where(x => check(x, rows * cols));
```

### 共享模块改动必须向后兼容

`GrainSizeAreaMethod.cs` 被多个模块引用（BLT/DHTG/IMC/HQL）。新增参数用可空类型 + 默认值：

```csharp
// ✅ 向后兼容：不传参数的旧调用方不受影响
public double? FovCenterX = null;
public double MaxAreaRatio = 0.25;
```

### CC 执行后必须读回验证

CC 可能在正确位置改了代码，也可能在错误位置。不能只看它的汇报摘要，必须自己读回代码确认。

## 适用场景

任何「分析需要人的判断、执行需要批量改代码」的任务都可用此模式。不限于 C#——Python、JS 等同理。
