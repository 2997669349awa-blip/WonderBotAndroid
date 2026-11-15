# WonderBot for Android 二创说明

本文档说明本仓库与上游项目的关系，以及本衍生版本做了哪些改动。

## 来源

| 项目 | 地址 | 许可 |
| --- | --- | --- |
| 上游 Android 管理器 | `JunWan666/astrbot-for-android` | MIT，Copyright (c) 2026 JunWan666 |
| 上游机器人程序 | `AstrBotDevs/AstrBot` | AGPL-3.0-or-later |
| 上游 QQ 协议端 | `NapNeko/NapCatQQ` | 见上游仓库 |
| 本衍生版本（Android） | `2997669349awa-blip/WonderBotAndroid` | 见各组件许可 |

上游 Android 管理器是**社区第三方移植项目**，不是 AstrBot 官方产物。它的作用是把 Python 写的机器人服务端与 NapCat 一起塞进手机的 PRoot Ubuntu 环境里运行，并提供原生管理界面。本仓库基于它的源码做二次开发。

## 本版本改动

1. **品牌**：应用名由「AstrBot 运行管理器 / AstrBot for Android」改为「WonderBot 运行管理器 / WonderBot for Android」，界面内所有面向用户的品牌文案同步替换。
2. **应用标识**：包名由 `io.astrbot.android` 改为 `dev.wonderbot.android`（含 `namespace`、`applicationId`、Kotlin/aidl 源码目录）。因此不能与上游版本覆盖安装，两者可共存。
3. **图标**：全部替换为 WonderBot 自有图标，启动图标各密度与自适应图标前景重新生成，背景色改为深黑 `#08080B`。
4. **配色**：由蓝色 / 亮色主题改为深黑主题（背景 `#08080B`、面板 `#121218`、强调色 `#5EE0C8`），不跟随系统明暗，始终深黑。
5. **更新通道**：三个在线更新目录默认留空，即**默认不检查在线更新**，避免误连上游官方通道。需要启用时通过 `-PupdateCatalogUrl=` 等参数指向自有签名目录；`channelBase` 已指向本项目仓库。上游更新公钥不再内置。
6. **版本**：`beta0.1.0`（versionCode 1）。
7. **签名**：使用本项目自有应用签名证书（PKCS12，别名 `release`），签名配置放在被 Git 忽略的 `.local/signing/`，不进入源码仓库。

## 刻意保留的内容

以下内容**没有改动**，改动会导致运行失败或违反许可：

- **Ubuntu rootfs 内部路径与环境变量契约**：`/opt/astrbot`、`persistent/astrbot/data`、`opt/astrbot-venv`、`launch_astrbot.py`、`ASTRBOT_ROOT`、`ASTRBOT_DESKTOP_*`、`ServiceId.ASTRBOT`、`__ASTRBOT_BLOB_READER_KEY__` 等。它们是 App 与 PRoot 环境内脚本之间的接口，不是品牌名。
- **上游 MIT 版权声明**：`LICENSE`、`docs/legal/NOTICE`、`docs/legal/AUTHORS.md`、`docs/legal/THIRD_PARTY_NOTICES.md` 以及 6 个 Kotlin 文件头的 `Copyright (c) 2026 JunWan666` 全部原样保留。MIT 要求分发时保留版权声明与许可文本。
- **上游来源标识**：`AstrBotDevs`、`astrbot.app`，以及 `runtime/` 下所有来源锁文件（含 SHA-256 摘要）。这些是构建期完整性校验的输入，改动会导致 `preBuild` 校验失败。
- **第三方组件名称与许可**：NapCat、Shizuku、Ubuntu、PRoot、cpolar、SnowLuma、xterm.js、DeepSeek Harness 等。

## 许可说明

- 上游 Android 管理器的原创代码与文档仍为 MIT，版权归 JunWan666，按许可证要求保留声明。
- 本衍生版本新增/修改部分同为 MIT。
- 机器人程序 WonderBot 派生自 AstrBot（AGPL-3.0-or-later），若分发含该程序的完整运行镜像，须遵守 AGPL 相应义务（含 §13 网络交互源码提供义务）。
- 本仓库的 MIT 许可**不重新授权**任何第三方组件，也不授予分发第三方二进制的许可。

## 构建

见 `docs/构建指南.md`。国内网络可在 `settings.gradle.kts` 与 `gradle/wrapper/gradle-wrapper.properties` 使用已配置的阿里云 / 腾讯云镜像。

```bash
# 只编译应用源码（无需完整运行镜像）
./gradlew :core:compileKotlin :app:compileBootstrapReleaseKotlin

# 构建可安装的 bootstrap APK
./gradlew :app:assembleBootstrapRelease
```

> `bootstrap` 变体产出的是**不含 1 GiB 预置 Ubuntu 镜像**的可安装 APK。完整离线包（`bundled`）需要按构建指南另外准备 ARM64 运行镜像、原生程序与许可输入，在具备 root 与 qemu-user-static 的环境中生成。
