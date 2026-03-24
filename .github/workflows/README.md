# GitHub Actions - Docker 自动编译指南

## 已创建的工作流

### 1. `docker-build.yml` (推荐)
- **功能**: 自动编译并推送到 GitHub Container Registry (ghcr.io)
- **触发条件**:
  - Push 到 `main` 分支
  - 推送 Git tag（格式: `v*.*.*`）
  - Pull Request 到 `main` 分支（仅编译不推送）
- **支持的平台**: Linux AMD64 和 ARM64

### 2. `docker-build-advanced.yml` (可选)
- **功能**: 支持推送到 GitHub Container Registry 和 Docker Hub
- **需要配置**: Docker Hub 账号凭证

---

## 快速开始 (GitHub Container Registry)

仅使用 `docker-build.yml` 即可，无需额外配置。GitHub Actions 会自动使用 `GITHUB_TOKEN`。

### 自动构建的镜像标签:
- `ghcr.io/YOUR_USERNAME/rainyun-qiandao:main` (main 分支最新版本)
- `ghcr.io/YOUR_USERNAME/rainyun-qiandao:v1.0.0` (发布版本)
- `ghcr.io/YOUR_USERNAME/rainyun-qiandao:sha-abc123` (Git commit hash)
- `ghcr.io/YOUR_USERNAME/rainyun-qiandao:latest` (main 分支)

### 使用镜像:
```bash
docker pull ghcr.io/YOUR_USERNAME/rainyun-qiandao:latest
```

---

## 配置 Docker Hub 支持 (可选)

仅当你想同时推送到 Docker Hub 时才需要配置。

### 步骤 1: 生成 Docker Hub Token
1. 访问 [Docker Hub Settings > Security](https://hub.docker.com/settings/security)
2. 点击 "New Access Token"
3. 输入 token 名称（如 `github-actions`）
4. 点击 "Generate"
5. 复制生成的 token

### 步骤 2: 添加 GitHub Secrets
1. 进入 GitHub 仓库 → Settings → Secrets and variables → Actions
2. 点击 "New repository secret"
3. 添加以下两个 secret:
   - `DOCKER_USERNAME`: 你的 Docker Hub 用户名
   - `DOCKER_PASSWORD`: 刚生成的 Docker Hub token

### 步骤 3: 选择使用的工作流
编辑 `.github/workflows/docker-build-advanced.yml`:
- 第 4 行修改 `IMAGE_NAME` 为你的 Docker Hub 仓库名
- Rename 或删除 `docker-build.yml` 并使用 `docker-build-advanced.yml`

### 或者，也可以同时使用两个工作流:
- `docker-build.yml` - 推送到 ghcr.io（自动）
- `docker-build-advanced.yml` - 推送到 Docker Hub（需配置）

---

## 工作流详解

### 触发条件
```yaml
on:
  push:
    branches: [main]      # Push 到 main 分支时
    tags: ['v*.*.*']      # 推送 tag 时（例: v1.0.0）
  pull_request:
    branches: [main]      # PR 到 main 分支时
```

### 镜像标签 (tagging strategy)

#### 多标签输出示例:
当 push tag `v1.2.3` 时:
- `v1.2.3` (完整版本)
- `v1.2` (主次版本)
- `main` (分支名)
- `sha-abc1234` (commit hash)
- `latest` (如果是在 main 分支)

### 构建缓存
使用 GitHub Actions Cache (GHA) 加速后续构建。

---

## 测试工作流

### 在本地测试 Dockerfile
```bash
docker build -t rainyun-qiandao:test .
docker run -it rainyun-qiandao:test
```

### 测试 GitHub Actions (需要 Act 工具)
```bash
# 安装 act: https://github.com/nektos/act
act -j build push
```

---

## 常见问题

### Q: 镜像没有推送到 ghcr.io？
A: 确认:
1. 工作流是否成功运行（check Actions tab）
2. 使用正确的镜像地址：`ghcr.io/USERNAME/rainyun-qiandao`
3. 镜像是否设置为公开（Packages settings）

### Q: 如何修改镜像名称？
A: 编辑 workflow 文件中的 `images:` 部分，替换 `${{ github.repository }}` 为你的镜像名。

### Q: 支持哪些平台？
A: 目前配置支持 `linux/amd64` 和 `linux/arm64`（ARM 架构如树莓派）。

### Q: 如何禁用自动推送？
A: 删除workflow 文件或将 `push: false` 改为 `push: ${{ github.ref == 'refs/heads/main' }}`。

---

## 安全建议

✅ 推荐做法:
- 使用标签化版本发布（git tags）
- 限制 actions 权限（`permissions:` 字段已配置最小化权限）
- 定期检查镜像安全（Docker Hub/ghcr.io 扫描功能）

⚠️ 注意事项:
- 不要在代码中提交敏感信息
- Docker Hub token 仅用于 GitHub Actions
- 定期更新 Action 版本（使用 latest 或 v4）

---

## 更多资源

- [GitHub Actions 官方文档](https://docs.github.com/en/actions)
- [Docker Build and Push Action](https://github.com/docker/build-push-action)
- [Docker Metadata Action](https://github.com/docker/metadata-action)
- [GitHub Container Registry 文档](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
