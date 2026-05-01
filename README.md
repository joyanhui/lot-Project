# lot-Project

公开发布仓库。

## M1-3 App Release Workflow

- Workflow 位置：`.github/workflows/release-m1-3-app.yml`
- 源码来源：`https://github.com/joyanhui/lot-manager-aio`
- 私有仓库拉取凭据：GitHub Actions secret `TOKEN_GH`

## iOS 产物说明

- 当前 workflow 在 macOS runner 上构建无签名 iOS Simulator `.app`
- 然后将 `.app` 压缩为 `.app.ipa`（zip 格式）
- Release 资产上传 `.app.ipa`
- 原始 `.app` 作为 Actions artifact 保留

这不是可直接安装到真机的已签名 IPA，而是按当前需求导出的无签名包。
