# docker-images

Monorepo for managing Docker images, built and pushed to Docker Hub via GitHub Actions.

## Structure

Each directory contains a `Dockerfile` for one image:

```
crosscell-benchmark/Dockerfile  →  biodancer/crosscell-benchmark
```

## Usage

### Add a new image

1. Create a new directory with a `Dockerfile`:
   ```
   mkdir my-new-image
   # add Dockerfile inside
   ```

### Build and push

Tag with `<image-name>/v<version>` to trigger a build:

```bash
git tag crosscell-benchmark/v1.0.0
git push origin crosscell-benchmark/v1.0.0
```

This builds and pushes:
- `biodancer/crosscell-benchmark:v1.0.0`
- `biodancer/crosscell-benchmark:latest`

## Git 操作

### 初始化仓库（首次）

```bash
git init
git remote add origin https://github.com/biodancerwangzhi/docker-images.git
git add .
git commit -m "init: project setup"
git branch -M main
git push -u origin main
```

### 日常提交

```bash
# 修改 Dockerfile 后提交
git add .
git commit -m "update: crosscell-benchmark dockerfile"
git push
```

### 触发镜像构建

```bash
# 打 tag 触发对应镜像的构建和推送
git tag crosscell-benchmark/v0.1.1
git push origin crosscell-benchmark/v0.1.1
```

### 新增镜像

```bash
mkdir my-new-image
# 在目录内编写 Dockerfile
git add .
git commit -m "feat: add my-new-image"
git push

# 准备好后打 tag 触发构建
git tag my-new-image/v1.0.0
git push origin my-new-image/v1.0.0
```

## Docker hub Setup

GitHub Actions 需要你的 Docker Hub 凭据才能推送镜像。这里用的是 Docker Hub Access Token（不是密码），更安全。

### 1. 创建 Docker Hub Access Token

1. 登录 [Docker Hub](https://hub.docker.com/)
2. 点击右上角头像 → **Account Settings**
3. 左侧菜单选择 **Security**
4. 点击 **New Access Token**
5. Description 填写 `github-actions`，权限选择 **Read & Write**
6. 点击 **Generate**，复制生成的 token（只显示一次，务必保存好）

### 2. 在 GitHub 仓库添加 Secrets

1. 打开你的 GitHub 仓库 `docker-images`
2. 进入 **Settings** → 左侧 **Secrets and variables** → **Actions**
3. 点击 **New repository secret**，添加以下两个：

| Name | Value |
|------|-------|
| `DOCKERHUB_USERNAME` | `biodancer` |
| `DOCKERHUB_TOKEN` | 上一步复制的 Access Token |

添加完成后，GitHub Actions 就能自动登录 Docker Hub 并推送镜像了。
