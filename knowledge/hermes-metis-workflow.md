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


## Codex CLI 独立代码审查（2026-08-23 实践）

给改动做"第二双眼睛"——尤其无 git 项目时，用临时仓库呈现精确 diff 后让 Codex 独立审查。

### 无 git 项目 → 临时 git 仓库呈现 diff

```bash
REV=/d/temp/codex_review_xxx && rm -rf $REV && mkdir -p $REV && cd $REV
git init && git config core.autocrlf false          # ⚠️ 必须先关，见坑
cp 备份的旧版文件 . && git add -A
git config user.email/user.name                      # 临时仓库局部身份
git commit -m base
cp 新版文件 .                                        # 工作区 = 改动
git diff --stat                                      # 应只显示真实改动行数
```

**autocrlf 坑**：全局 autocrlf=true 会把 CRLF 文件归一化成 LF 入库 → `git diff` 显示全文件重写（如 407+/292- 而实际只改 115 行）。修复：仓库局部 `core.autocrlf false` + 恢复旧版重新 `git commit --amend`。

### Codex 调用

```bash
export HTTPS_PROXY=http://127.0.0.1:7897 HTTP_PROXY=http://127.0.0.1:7897  # 必须，直连被墙
codex exec --sandbox danger-full-access "任务描述"    # 后台跑 + pty=true
```

- 认证 = ChatGPT 订阅 OAuth（`~/.codex/auth.json`，模型 gpt-5.6-terra），烧订阅额度（实测 68 秒 ~23.4k tokens）
- **只读审查靠提示词约束**（"只审查，禁止修改任何文件"），不依赖沙箱；`codex review` 子命令是 PR 场景，本地审查用 exec + git diff
- 提示词要素：背景（改动目的）+ 检查要点（编号列表，给公式和对照文件）+ 输出格式（严重级别+行号+验证过的检查点清单）
- 实测价值：独立验证 7 项检查点全过；顺带发现既有隐患（`_resultList.Add` 重复键会抛 ArgumentException——非本次改动引入，仅记录）
