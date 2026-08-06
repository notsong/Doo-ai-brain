# 部署知识

> 模型部署、打包和生产环境模式。

## 常用模式

<!-- TODO: 服务模式、API 设计 -->

## 工具

<!-- TODO: Docker、PyInstaller、ONNX Runtime、Triton 等 -->

## 踩过的坑

### Git 推送 GitHub 被墙（GFW）

**现象**：`git push` 报 `Recv failure: Connection was reset`，能 ping 通但连不上。

**原因**：国内网络环境，GitHub HTTPS 直连被阻断。

**解决**：配置 Git 走代理。Clash Verge 默认 HTTP 代理端口为 `7897`（不同客户端不同：Clash 通常 7890，Clash Verge 可能 7897）。

```bash
git config --global http.proxy http://127.0.0.1:7897
git config --global https.proxy http://127.0.0.1:7897
```

验证代理端口是否可用：
```bash
curl -x http://127.0.0.1:7897 https://github.com -o /dev/null -w "%{http_code}"
# 返回 200 即通
```

### gh CLI 认证所需 Scope

`gh auth login` 需要 Token 至少勾选三个 scope：
- `repo` — 读写仓库
- `read:org` — gh CLI 验证必需
- `workflow` — 可选，方便后续加 CI/CD

如果用浏览器登录失败（被墙），选 Paste an authentication token，去 https://github.com/settings/tokens 创建。

## PyInstaller 笔记

<!-- TODO: ML 模型打包的特殊问题 -->

## 参考资料

- [GitHub CLI 安装](https://github.com/cli/cli#installation)
- [GitHub Personal Access Token](https://github.com/settings/tokens)
