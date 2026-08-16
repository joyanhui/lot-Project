# lot-Project

`lot-Project` 是发布与 CI 工程目录，核心入口是 `.github/workflows/lot-aio-release.yml`。该目录用于从私有源码仓库 `joyanhui/lot-manager-aio` 拉取源码，在 nix 开发环境中构建全部产物并发布压缩包资产。

## 目录结构

- `.github/workflows/lot-aio-release.yml`：手动 release workflow，通过 `dev.sh build:all` 构建并发布资产。
- `.github/workflows/ci-m1-1-app-api-rs.yml`：M1-1 控制面 CI。
- `.github/workflows/ci-m0-1-center-go.yml`：M0-1 运行态中心 CI（Go）。
- `.github/workflows/ci-m2-4-dev-reg-go.yml`：M2-4 设备注册服务 CI（Go）。
- `.github/workflows/ci-m4-1-userApp-tauri.yml`：M4-1 Tauri App CI。
- `.github/workflows/ci-m2-1-listener-rs.yml`：M2-1 接入层 CI。
- `.github/workflows/ci-m2-2-archive-query-rs.yml`：M2-2 归档查询层 CI。
- `.github/workflows/ci-m3-rs.yml`：M3 固件 CI。
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
- iOS job（macos）：构建无签名 iOS Simulator `.app`，压缩为 `.app.ipa`（zip 格式，非真机可安装 IPA）。

## 发布资产

- `linux_x86.tar.zst` + `linux_x86.tar.zst.sha256`
- `linux_arm64.tar.zst` + `linux_arm64.tar.zst.sha256`
- `m4-1-userApp-tauri-ios-simulator.app.ipa`

压缩包内布局：5 个服务二进制、`m1-9-ops-panel-ts-<tag>-single`、`dev-gui-manager-ts-<tag>-single`、`frontend_dist/`、`config.lot.v2.json5`、`m3/`（固件与分区文件）、`m4-1/`（apk/aab，仅 arm64 包）。不再发布 Docker 镜像与各模块独立压缩包。

## 与主仓库的关系

- 主仓库源码来源：`https://github.com/joyanhui/lot-manager-aio`。
- 主仓库触发链路：`.github/workflows/trigger-lot-project-release.yml` → `lot-Project/.github/workflows/lot-aio-release.yml`。
- 发布资产最终上传回 `joyanhui/lot-manager-aio` 的目标 release。

## 使用方式

手动触发 release：在仓库根目录执行 `release-tag` 命令，创建并推送 tag 即可通过 `.github/workflows/trigger-lot-project-release.yml` 自动调度 release workflow。

```bash
bash script/dev.sh release-tag:create <tag>
```

任意 `v*` 前缀 tag（含模块前缀如 `m1-1-app-api-rs-v*`）统一触发 `build:all`，发布全部资产。
