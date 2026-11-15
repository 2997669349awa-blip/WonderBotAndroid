# 构建 WonderBot for Android APK

本工程基于上游 `JunWan666/astrbot-for-android`（MIT）二次开发。构建分两档，需要先理解区别。

## 两种 APK

| 变体 | 产物 | 是否含运行环境 | 装到手机后 |
| --- | --- | --- | --- |
| `bootstrap` | 约数 MB | **不含** PRoot Ubuntu 镜像 | 可安装、可打开原生管理界面；需另外导入运行环境才能启动机器人 |
| `bundled` | 约 1 GiB | **含** ARM64 Ubuntu + Python + NapCat + Linux QQ | 装完初始化即可在手机本地跑 QQ 机器人 |

第三方正式发布的「一个 APK 装上就能用」是 `bundled` 变体。本仓库源码**不含**那 1 GiB 的预置镜像（上游也不放进 Git），需要按下面步骤自行生成。

## 环境

- JDK 17（本端已装于 `/opt/jdk17`）
- Android SDK：platform android-35、build-tools 35.0.0（已装）
- Gradle 8.11.1（wrapper 已配置）
- Python 3.11+，Android NDK 27.0.12077973（构建 `bundled` 时的原生运行器需要）
- 生成完整镜像另需：Linux root、`dpkg-deb`、`qemu-user-static` + binfmt（x86 主机跑 ARM64 包）

国内网络已在以下位置配好镜像，无需改动：

- `settings.gradle.kts`：阿里云 / 腾讯云 Maven 优先
- `gradle/wrapper/gradle-wrapper.properties`：腾讯云 Gradle 发行版
- `/root/.gradle/init.gradle`：settings 级镜像注入（不违反 `FAIL_ON_PROJECT_REPOS`）

## 构建 bootstrap APK（可安装、已签名）

```bash
export JAVA_HOME=/opt/jdk17
export ANDROID_HOME=/workspace/android-sdk
cd /workspace/WonderBotAndroid
./gradlew :app:assembleBootstrapRelease \
  -PappVersionName=beta0.1.0 -PappVersionCode=1
```

产物：`app/build/outputs/apk/bootstrap/release/app-bootstrap-release.apk`

签名配置在 `.local/signing/release.properties`（已被 Git 忽略），使用自有证书 `release.p12`，别名 `release`。**请备份整个 `.local/signing/` 目录**：后续版本必须用相同包名、相同证书并提高 `appVersionCode` 才能覆盖升级。

## 构建 bundled 完整包

按顺序准备工程外输入，详见 `docs/构建指南.md`：

1. **运行环境镜像**：按 `runtime/provision-base.sh`、`runtime/provision-astrbot.py`、`runtime/provision-napcat.py` 在干净的 ARM64 环境中准备 Ubuntu + Python + WonderBot + NapCat，用 `runtime/export-runtime.sh` 导出到 `runtime/apk-assets/bootstrap/arm64-v8a/`，并生成匹配的 `manifest.json`。
   > 注意：WonderBot 的 Python 源码在本项目另一仓库 `2997669349awa-blip/wonderBot`。镜像内目录仍为 `/opt/astrbot`，这是 App 与运行脚本的接口契约，不要改名。
2. **原生程序**：用 NDK 编译 `native/process_runner.c`、`native/terminal_runner.c`，连同 PRoot 等库放入 `app/src/main/jniLibs/`。
3. **桌面依赖与许可材料**：`tools/build_support/prepare_optional_assets.py`、`prepare_desktop_snapshot.py`、`collect_distribution.py`。
4. **一键构建**：

```bash
python3 -B tools/build_release.py
```

脚本会校验签名身份、收集发行依赖与许可材料、构建 ARM64 Release、运行 lint，并校验 APK 的 v1/v2/v3 签名、包名、版本、架构与摘要，通过后导出到 `dist/`。

## 版本与更新

- 包名：`dev.wonderbot.android`（与上游 `io.astrbot.android` 不同，可共存，不能覆盖安装）
- 当前版本：`beta0.1.0`，versionCode `1`
- **在线更新默认关闭**：`updateCatalogUrl` / `napcatCatalogUrl` / `managerCatalogUrl` 均为空，不会连接上游官方通道。需要自有更新通道时，用 `-PupdateCatalogUrl=https://…` 指向你自己的签名目录，并配套 `-PupdatePublicKeyFile=`。

## 合规

分发前请阅读 `WONDERBOT.md`、`docs/legal/NOTICE` 与 `docs/legal/THIRD_PARTY_NOTICES.md`。上游 MIT 版权声明必须保留；若分发含 WonderBot（AGPL-3.0-or-later）的完整运行镜像，须履行 AGPL 相应义务（含 §13）。
