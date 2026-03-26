# Termux Bootstrap CI

在 GitHub Actions 云端自动编译自定义包名的 `bootstrap-{arch}.zip`。

## 为什么需要这个

Termux 软件包在编译时会**硬编码路径名**（如 `/data/data/com.termux/files/usr/`）。使用自定义包名时必须从源码重新编译整个 bootstrap。本项目用 GitHub Actions 在云端完成这一耗时任务。

## 工作原理

与官方 `generate-bootstraps.sh`（直接从 apt 仓库下载预编译包）不同，本 CI 采用 **全量源码编译** 策略：

1. **修改 `properties.sh`** — 将 `com.termux` 替换为自定义包名
2. **补丁 `generate-bootstraps.sh`** — 修复自定义路径下 `mkdir -p` 缺失的问题
3. **源码编译全部 ~30 个 bootstrap 包** — `build-package.sh` 读取修改后的 properties，编译出路径正确的 `.deb`
4. **搭建本地 apt 仓库** — 用编译产物生成 `Packages` 索引
5. **组装 bootstrap zip** — `generate-bootstraps.sh --repository file:///local-repo` 从本地仓库拉包，路径全部一致

这确保了 bootstrap 中每一个二进制文件和元数据的路径都使用自定义包名。

## 快速开始

1. Fork 或创建新仓库，将 `.github/workflows/build-bootstrap.yml` 放入
2. 进入 **Actions → Build Termux Bootstrap → Run workflow**
3. 填写参数后运行（首次构建预计 2-4 小时）
4. 构建完成后从 **Artifacts** 下载 `bootstrap-{arch}.zip`

## 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `package_name` | `com.mytermux` | 自定义包名，替换 `com.termux` |
| `packages` | _(空)_ | 额外包含的软件包，逗号分隔。**留空 = 仅编译 bootstrap 基础包集**（约 30 个最小启动包） |
| `termux_packages_ref` | `master` | termux-packages 的分支/tag |

> **架构**: 当前硬编码为 `aarch64`（调试阶段），后续可通过 input 参数化。
>
> **超时**: 360 分钟（6 小时），源码编译 ~30 个包需要较长时间。

## Bootstrap 基础包集

以下包会从源码编译并打入 bootstrap：

```
apt, bash, bzip2, command-not-found, coreutils, curl, dash, debianutils,
diffutils, dos2unix, ed, findutils, gawk, grep, gzip, inetutils, less,
lsof, nano, net-tools, patch, procps, psmisc, sed, tar, termux-core,
termux-exec, termux-keyring, termux-tools, unzip, util-linux, xz-utils
```

这些包的依赖项也会被递归编译。

## 拿到 zip 后怎么用

1. 解压 `bootstrap-aarch64.zip` 放入 termux-app 的 `/app/src/main/cpp/` 目录
2. 在 `app/build.gradle` 的 `task downloadBootstraps()` 开头加 `return;`
3. Clean Project → Build

## 关于 `packages` 参数

- **留空**: 仅编译 bootstrap 基础包集
- **填写如 `vim,curl,openssh`**: 在基础集之上额外编译并打入这些包
- 由于所有路径已硬编码为自定义包名，**构建后无法通过 `apt install` 从官方仓库安装新包**（需要同样重编译的仓库）

## 为什么不能用预编译包？

官方 apt 仓库（`packages-cf.termux.dev`）中的 `.deb` 包内部路径是 `/data/data/com.termux/files/usr/...`。如果下载这些包组装 bootstrap，但运行时 app 使用 `/data/data/com.your.pkg/files/usr/...`，所有硬编码路径都会错配，导致 Termux 完全无法工作。

## License

与 Termux 一致，遵循 GPLv3。
