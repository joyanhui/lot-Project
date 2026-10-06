# lot-Project

`lot-Project` 是发布与 CI 工程目录，核心入口是 `.github/workflows/lot-aio-release.yml`。该目录用于从私有源码仓库 `joyanhui/lot-manager-aio` 拉取源码，在 nix 开发环境中构建全部产物并发布压缩包资产与私有 ghcr 镜像。

## 目录结构

- `.github/workflows/lot-aio-release.yml`：手动 release workflow，通过 `dev.sh build:all` 构建并发布资产。
- `.github/workflows/ci-m1-1-app-api-rs.yml`：M1-1 控制面 CI。
- `.github/workflows/ci-m1-2-device-api-rs.yml`：M1-2 设备面 CI。
- `.github/workflows/ci-m0-1-center-rs.yml`：M0-1 运行态中心 CI（Rust）。
- `.github/workflows/ci-m4-1-userApp-tauri.yml`：M4-1 Tauri App CI。
- `.github/workflows/ci-m2-1-listener-rs.yml`：M2-1 接入层 CI。
- `.github/workflows/ci-m2-2-archive-query-rs.yml`：M2-2 归档查询层 CI。
- `.gitignore`：发布工程忽略规则。

## release workflow

入口：`.github/workflows/lot-aio-release.yml`

输入参数：

- `publish_release`：是否发布到 release。
- `release_tag`：发布 tag。
- `source_ref`：源码 ref，通常是分支、tag 或 commit。

私有仓库拉取凭据：GitHub Actions secret `TOKEN_GH`。

构建环境统一复用私有仓库的 `flake.nix` / `flake_pkgs_let.nix` / `flake.lock`（`cachix/install-nix-action` + `nix develop`），不在 runner 上手写依赖；仅 iOS job 因 macOS 无法使用 linux flake 而保留原生步骤。

## 构建方式

- x86_64 job：`bash script/dev.sh build:all`，Rust 服务端 native glibc 构建（CI 用 runner 系统 gcc，兼容 Debian），产出 `dist/linux_x86.tar.zst`。
- arm64 job：`bash script/dev.sh build:all arm64`，Rust 服务端经 `cargo-zigbuild`（zig）交叉编译为静态 musl 二进制，产出 `dist/linux_arm64.tar.zst`。
- 两个 linux job 在 `build:all` 之后继续构建并推送 Docker 镜像到私有 ghcr `ghcr.io/joyanhui/lot-manager-aio`：arm64 先 `docker/setup-qemu-action`，统一 `docker/setup-buildx-action`，`docker login ghcr.io`（`TOKEN_GH`）后 `nix develop` 调用 `bash script/dev.sh build:dockerimage <arch>`（`LOT_IMAGE_PUSH=1`，镜像内二进制为 musl 静态，基础镜像 `debian:trixie-slim`）；`merge-image` job 把两个平台 tag 合并为 `:<release_tag>` 多平台清单，仅正式 tag 追加 `latest`，并强制包可见性为 private。
- iOS job（macos）：构建无签名 iOS Simulator `.app`，压缩为 `.app.ipa`（zip 格式，非真机可安装 IPA）。

## 发布资产

- `linux_x86.tar.zst` + `linux_x86.tar.zst.sha256`
- `linux_arm64.tar.zst` + `linux_arm64.tar.zst.sha256`
- `m4-1-userApp-tauri-ios-simulator.app.ipa`
- 私有 Docker 镜像：`ghcr.io/joyanhui/lot-manager-aio`（`<release_tag>-amd64`/`-arm64` 单平台 tag，`<release_tag>` 多平台清单；仅正式版本 `^v[0-9]+\.[0-9]+\.[0-9]+$` 附加 `latest`；可见性 private）

压缩包内布局：5 个服务二进制、`m1-9-ops-panel-rs-single` 与 `m1-9-ops-panel-rs/frontend-ops/dist`、`frontend_dist/`、`config.lot.v2.json5`、`m3/`（固件与分区文件）、`m4-1/`（apk/aab，仅 arm64 包）。`m-simulators` 模拟器（`m-sim`）为开发工具，不随部署包发布，本地构建与使用见根 README 与 `m-simulators/README.md`。Docker 镜像为独立资产（一个镜像含全部服务，用 `SERVICE_ROLE` 选择服务，详见 `script/README.md`），不再发布各模块独立压缩包，也不再把镜像包作为 release 资产上传。

## 与主仓库的关系

- 主仓库源码来源：`https://github.com/joyanhui/lot-manager-aio`。
- 主仓库触发链路：`.github/workflows/trigger-lot-project-release.yml` → `lot-Project/.github/workflows/lot-aio-release.yml`。
- 发布资产最终上传回 `joyanhui/lot-manager-aio` 的目标 release；Docker 镜像推送到私有 ghcr，供远端 docker-compose 部署拉取。

## 使用方式

手动触发 release：在仓库根目录执行 `release-tag` 命令，创建并推送 tag 即可通过 `.github/workflows/trigger-lot-project-release.yml` 自动调度 release workflow。

```bash
bash script/dev.sh release:tag <tag>
```

任意 `v*` 前缀 tag（含模块前缀如 `m1-1-app-api-rs-v*`）统一触发 `build:all`，发布全部资产并推送 ghcr 镜像；只有正式版本 tag（`vX.Y.Z`，无 alpha/rc 等后缀）才会同时打 `latest`。
