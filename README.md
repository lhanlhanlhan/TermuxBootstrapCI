# Termux Bootstrap CI

在 GitHub Actions 云端自动编译自定义包名的 `bootstrap-{arch}.zip`。

## 为什么需要这个

Termux 软件包在编译时会**硬编码路径名**（如 `/data/data/com.termux/files/usr/`）。使用自定义包名时必须从源码重新编译整个 bootstrap。本项目用 GitHub Actions 在云端完成这一耗时任务。

## 快速开始

1. Fork 或创建新仓库，将 `.github/workflows/build-bootstrap.yml` 放入
2. 进入 **Actions → Build Termux Bootstrap → Run workflow**
3. 填写参数后运行
4. 构建完成后从 **Artifacts** 下载 `bootstrap-{arch}.zip`

## 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `package_name` | `com.mytermux` | 自定义包名，替换 `com.termux` |
| `packages` | _(空)_ | 额外包含的软件包，逗号分隔。**留空 = 仅编译 bootstrap 基础包集**（bash, apt, coreutils 等最小启动集），不是所有包 |
| `termux_packages_ref` | `master` | termux-packages 的分支/tag |

> 当前架构硬编码为 `aarch64`（调试阶段），后续可通过 input 参数化。

## 拿到 zip 后怎么用

1. 解压 `bootstrap-aarch64.zip` 放入 termux-app 的 `/app/src/main/cpp/` 目录
2. 在 `app/build.gradle` 的 `task downloadBootstraps()` 开头加 `return;`
3. Clean Project → Build

## 关于 `packages` 参数

- **留空**：仅编译 bootstrap 基础包集（`generate-bootstraps.sh` 中预定义的最小集合，通常包含 apt、bash、busybox、coreutils、dash、sed、tar、termux-tools 等）
- **填写如 `vim,curl,openssh`**：在基础集之上额外编译并打入这些包，安装后即可直接使用
- 无论哪种方式，由于包名/路径已硬编码为自定义值，**构建后无法通过 `apt install` 从官方仓库安装新包**

## License

与 Termux 一致，遵循 GPLv3。
