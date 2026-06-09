# lot-Project

`lot-Project` 是发布与 CI 工程目录，核心入口是 `.github/workflows/release.yml`。该目录用于从私有源码仓库 `joyanhui/lot-manager-aio` 拉取源码，构建服务端二进制、M1-3 客户端、Docker 镜像、M3 固件和校验资产，并把发布产物上传回目标 release。

## 目录结构

- `.github/workflows/release.yml`：手动 release workflow，按 `build_target` 构建并发布资产。
- `.github/workflows/ci-m1-1-platform-rs.yml`：M1-1 控制面 CI。
- `.github/workflows/ci-m1-3-app.yml`：M1-3 Tauri App CI。
- `.github/workflows/ci-m1-4-wechat-app.yml`：微信小程序 CI。
- `.github/workflows/ci-m2-1-listener.yml`：M2-1 接入层 CI。
- `.github/workflows/ci-m2-2-telemetry-tracking-and-archive.yml`：M2-2 归档查询层 CI。
- `.github/workflows/ci-m3-v1.yml`：M3 固件 CI。
- `.gitignore`：发布工程忽略规则。

## release workflow

入口：`.github/workflows/release.yml`

输入参数：

- `build_target`：构建目标，默认 `all`。
- `publish_release`：是否发布到 release。
- `release_tag`：发布 tag。
- `source_ref`：源码 ref，通常是分支、tag 或 commit。

私有仓库拉取凭据：GitHub Actions secret `TOKEN_GH`。

## 构建目标

- `all`：构建全部 release 资产。
- `lot-manager-aio-docker`：构建整包 Docker 目标。
- `lot-manager-aio-x86_64-gnu-docker`：构建 x86_64 GNU Docker 目标。
- `lot-manager-aio-aarch64-musl-docker`：构建 aarch64 musl Docker 目标。
- `m1-1-platform-rs`：构建 M1-1 控制面服务。
- `m2-1-listener`：构建 M2-1 接入层服务。
- `m2-2-telemetry-tracking-and-archive`：构建 M2-2 归档查询服务。
- `m4-1-inlet-manager`：构建 M4 inlet 分发服务。
- `m3-v1-esp32c3-classicsimple-firmware`：构建 ESP32-C3 classicsimple 固件。

## 主要产物

- Rust 服务二进制：M1-1、M2-1、M2-2、M4。
- M1-3 桌面/移动端资产：Linux、Windows、macOS、Android、iOS simulator 相关包。
- Docker 镜像或镜像构建资产。
- M3 ESP32-C3 固件资产。
- `SHA256SUMS.txt`：发布资产 sha256 汇总。

## iOS 产物说明

- workflow 在 macOS runner 上构建无签名 iOS Simulator `.app`。
- `.app` 会压缩为 `.app.ipa` zip 格式上传 release。
- 原始 `.app` 作为 Actions artifact 保留。
- 该产物不是可直接安装到真机的已签名 IPA。

## 与主仓库的关系

- 主仓库源码来源：`https://github.com/joyanhui/lot-manager-aio`。
- 主仓库触发链路：`.github/workflows/trigger-lot-project-release.yml` → `lot-Project/.github/workflows/release.yml`。
- 发布资产最终上传回 `joyanhui/lot-manager-aio` 的目标 release。

## 使用方式

手动触发 release：在仓库根目录执行 `release-tag` 命令，创建并推送 tag 即可通过 `.github/workflows/trigger-lot-project-release.yml` 自动调度 release workflow。

```bash
bash script/dev.sh release-tag <tag>
```

tag 前缀决定 `build_target`：

| tag 示例 | 构建目标 |
|---|---|
| `v0.1.0` | `all`（全部资产） |
| `m1-1-platform-rs-v0.1.0` | `m1-1-platform-rs` |
| `m1-3-app-v0.1.0` | M1-3 桌面/移动端 |
| `m2-1-listener-v0.1.0` | `m2-1-listener` |
| `m2-2-telemetry-tracking-and-archive-v0.1.0` | M2-2 归档查询 |
| `m4-1-inlet-manager-v0.1.0` | `m4-1-inlet-manager` |
| `m3-v1-firmware-v0.1.0` | ESP32-C3 固件 |
