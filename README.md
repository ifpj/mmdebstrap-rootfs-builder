# mmdebstrap-rootfs-builder

纯 GitHub Actions 方案：使用原生 ARM64 runner（`ubuntu-24.04-arm`）通过 [mmdebstrap](https://gitlab.mister-muffin.de/josch/mmdebstrap) 并行构建 **Debian trixie** 和 **Ubuntu resolute (26.04 LTS)** 的 `minbase` rootfs，并自动上传到 GitHub Release。

> **注意：** mmdebstrap 的 variant 拼写是 `minbase`（不是 `minibase`）。

## 特性

- **原生 ARM64 构建**：使用 GitHub 官方 ARM64 runner，无需 QEMU 转译。
- **mmdebstrap**：比传统 `debootstrap` 快约一倍，支持多 mirror 和现代依赖解析。
- **并行构建**：Debian + Ubuntu 同时跑，总耗时 ≈ 1 分钟。
- **多格式压缩**：同时提供 `tar.gz` 和 `tar.xz` 两种压缩格式。
- **最小化 rootfs**：默认仅预装 `apt`、`ca-certificates`，可通过 `workflow_dispatch` 自定义。
- **纯 Actions**：所有逻辑均在 GitHub Actions 内完成，无需本地脚本。

## Release 产物

推送 `v*` 开头的 tag 后，Actions 会自动构建并上传以下文件：

| 文件 | 说明 |
|------|------|
| `debian-trixie-minbase-arm64.tar.gz` | Debian trixie rootfs (gzip) |
| `debian-trixie-minbase-arm64.tar.xz` | Debian trixie rootfs (xz) |
| `debian-trixie-minbase-arm64.sha256` | SHA256 校验 |
| `debian-trixie-minbase-arm64-manifest.txt` | 完整包清单 |
| `ubuntu-resolute-minbase-arm64.tar.gz` | Ubuntu resolute rootfs (gzip) |
| `ubuntu-resolute-minbase-arm64.tar.xz` | Ubuntu resolute rootfs (xz) |
| `ubuntu-resolute-minbase-arm64.sha256` | SHA256 校验 |
| `ubuntu-resolute-minbase-arm64-manifest.txt` | 完整包清单 |

## 快速开始

### 方式一：Push Tag 自动发布（推荐）

```bash
git tag v1.0.0
git push origin v1.0.0
```

等待 Actions 完成后，在仓库 **Releases** 页面下载 rootfs。

### 方式二：手动触发（workflow_dispatch）

进入 **Actions** → **Build and Release rootfs** → **Run workflow**，可自定义以下参数：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `variant` | mmdebstrap variant；设为 `default` 则省略 `--variant` | `minbase` |
| `arch` | 目标架构 | `arm64` |
| `include` | 额外预装的包（逗号分隔） | `apt,ca-certificates` |
| `debian_suite` | Debian 版本代号 | `trixie` |
| `debian_mirror` | Debian 镜像源 | `http://deb.debian.org/debian` |
| `debian_components` | Debian 组件（逗号分隔） | `main,contrib,non-free,non-free-firmware` |
| `ubuntu_suite` | Ubuntu 版本代号 | `resolute` |
| `ubuntu_mirror` | Ubuntu 镜像源 | `http://ports.ubuntu.com/ubuntu-ports` |
| `ubuntu_components` | Ubuntu 组件（逗号分隔） | `main,restricted,universe,multiverse` |

## 本地使用 rootfs

```bash
# 安装工具
sudo apt install xz-utils

# 准备目录
mkdir -p rootfs

# 解压（gzip 格式）
tar -xzf debian-trixie-minbase-arm64.tar.gz -C rootfs

# 或解压（xz 格式）
tar -xJf debian-trixie-minbase-arm64.tar.xz -C rootfs

sudo chroot rootfs /bin/bash
```

## 构建参数

| 参数 | 说明 |
|------|------|
| `--variant=minbase` | 最小基础系统（含 apt/dpkg 等必要工具） |
| `--architecture=arm64` | 目标架构 |
| `--components=main,contrib,non-free,non-free-firmware` | 包含的仓库组件（Debian） |
| `--include=apt,ca-certificates` | 额外预装的包 |
| `gzip` / `xz -T0` | 压缩格式（同时生成两种） |

## 许可证

MIT
