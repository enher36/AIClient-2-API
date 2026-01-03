# GitHub Actions Docker 自动构建配置说明

## 📋 前提条件

在使用 GitHub Actions 自动构建 Docker 镜像之前，需要在 GitHub 仓库中配置以下 Secrets：

### 1. 配置 Docker Hub Secrets

进入你的 GitHub 仓库，按照以下步骤配置：

1. 点击仓库的 **Settings** (设置)
2. 在左侧菜单中找到 **Secrets and variables** → **Actions**
3. 点击 **New repository secret** 添加以下两个密钥：

#### DOCKERHUB_USERNAME
- **名称**: `DOCKERHUB_USERNAME`
- **值**: 你的 Docker Hub 用户名
- **示例**: `enher36`

#### DOCKERHUB_TOKEN
- **名称**: `DOCKERHUB_TOKEN`
- **值**: Docker Hub 访问令牌 (Access Token)
- **获取方式**:
  1. 登录 [Docker Hub](https://hub.docker.com/)
  2. 点击右上角头像 → **Account Settings**
  3. 选择 **Security** → **New Access Token**
  4. 输入描述信息（如 "GitHub Actions"）
  5. 设置权限为 **Read, Write, Delete**
  6. 点击 **Generate** 生成令牌
  7. **复制令牌并保存**（令牌只显示一次）

## 🚀 工作流程说明

### 触发条件
- **自动触发**: 每次推送代码到 `main` 分支时自动执行
- **手动触发**: 可在 GitHub Actions 页面手动运行

### 执行步骤

1. **读取版本号**: 从 `VERSION` 文件读取当前版本
2. **构建镜像**: 使用 Docker Buildx 构建多平台镜像 (amd64/arm64)
3. **推送镜像**: 推送到 Docker Hub，标签包括:
   - `latest` - 最新版本
   - `<version>` - 版本号（如 `2.3.4`）
   - `main-<sha>` - 带提交 SHA 的标签
4. **更新 README**: 自动将所有 README 文件中的镜像名更新为你的 Docker Hub 用户名
5. **提交更改**: 如果 README 有更新，自动提交并推送

### 镜像标签示例

假设你的 Docker Hub 用户名是 `enher36`，VERSION 文件内容是 `2.3.4`，则会生成以下标签：

```
enher36/aiclient-2-api:latest
enher36/aiclient-2-api:2.3.4
enher36/aiclient-2-api:main-abc1234
```

## 📦 使用构建的镜像

构建完成后，可以使用以下命令拉取和运行：

```bash
# 拉取最新版本
docker pull <your-username>/aiclient-2-api:latest

# 运行容器
docker run -d -p 3000:3000 -p 8085:8085 -p 8086:8086 -p 19876-19880:19876-19880 \
  --restart=always \
  -v "your_path:/app/configs" \
  --name aiclient2api \
  <your-username>/aiclient-2-api:latest
```

## 🔍 查看构建状态

1. 进入仓库的 **Actions** 标签页
2. 查看 "Build Docker Image and Update README" 工作流
3. 点击具体的运行记录查看详细日志

## ❓ 常见问题

### Q1: 构建失败，提示 "denied: requested access to the resource is denied"
**A**: 检查 `DOCKERHUB_TOKEN` 是否正确配置，确保令牌有 Write 权限

### Q2: README 没有自动更新
**A**: 检查镜像名称是否与 README 中的不同，只有检测到差异时才会更新

### Q3: 如何禁用自动构建？
**A**:
- 方法1: 删除 `.github/workflows/docker-build-and-update.yml` 文件
- 方法2: 在 workflow 文件中注释掉触发条件

### Q4: 想要修改镜像名称
**A**: 镜像名称自动从 `DOCKERHUB_USERNAME` secret 获取，修改 secret 即可

## 📝 注意事项

1. **首次推送**: 配置好 Secrets 后，首次推送代码会触发构建
2. **构建时间**: 多平台构建大约需要 5-10 分钟
3. **Docker Hub 限制**: 免费账户有拉取次数限制（200次/6小时）
4. **README 更新**: 自动更新会创建新的提交，可能触发再次构建（但第二次不会更新 README）
5. **版本管理**: 修改 `VERSION` 文件可以创建新的版本标签

## 🔗 相关链接

- [Docker Hub](https://hub.docker.com/)
- [GitHub Actions 文档](https://docs.github.com/actions)
- [Docker Buildx 文档](https://docs.docker.com/buildx/working-with-buildx/)
