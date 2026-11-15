<div align="center">

# WonderBot for Android

**把 WonderBot 机器人服务端搬进安卓手机的原生运行管理器 · 独立二创发行版**

[![Version](https://img.shields.io/badge/Version-beta0.1.0-5EE0C8?style=flat-square)](https://github.com/2997669349awa-blip/WonderBotAndroid/releases)
[![License](https://img.shields.io/badge/License-MIT-5EE0C8?style=flat-square)](./LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Android%208.0%2B%20%7C%20ARM64-5EE0C8?style=flat-square)](#)
[![Base](https://img.shields.io/badge/Base-astrbot--for--android%201.4.1-8A8A99?style=flat-square)](https://github.com/JunWan666/astrbot-for-android)

[下载 APK](https://github.com/2997669349awa-blip/WonderBotAndroid/releases/tag/beta0.1.0) ·
[主项目 wonderBot](https://github.com/2997669349awa-blip/wonderBot) ·
[官网](https://2997669349awa-blip.github.io/wonderBot/) ·
[二创说明](./WONDERBOT.md) ·
[构建指南](./BUILD_WONDERBOT.md) ·
[English](./docs/README.en.md)

</div>

---

## 这是什么

WonderBot 本体是一个用 Python 写的聊天机器人**服务端**（见主仓库 [`wonderBot`](https://github.com/2997669349awa-blip/wonderBot)，AGPL-3.0-or-later），正常跑在电脑 / 服务器 / NAS 上，**官方并不提供 APK**。

本仓库是它的**安卓端运行管理器**：基于社区第三方移植项目 [`JunWan666/astrbot-for-android`](https://github.com/JunWan666/astrbot-for-android)（MIT，v1.4.1）二次开发，用 Kotlin + Jetpack Compose 做原生界面，通过 **PRoot** 在手机里跑一整套 ARM64 Ubuntu 环境，把 Python 机器人、NapCat 协议端、Linux QQ 等一起塞进去，从而在安卓手机上本地运行 WonderBot。

> 简单说：**主仓库是机器人本身，本仓库是让机器人住进手机的壳 + 运行环境。**

## ⚠️ 先读这一段：两种安装包的区别

这是最容易被误解的地方，务必看清楚：

| | `bootstrap`（本次发布） | `bundled`（上游正式版） |
| --- | --- | --- |
| APK 体积 | **约 11 MB** | 约 1022 MB |
| 能否安装 | ✅ 能装、能打开原生管理界面 | ✅ 能装、能直接用 |
| 是否内置运行环境 | ❌ **不含** ARM64 Ubuntu 镜像 | ✅ 内置完整镜像 |
| 装完能否直接跑机器人 | ❌ 需另行准备运行环境 | ✅ 初始化后即可用 |
| 本仓库是否提供 | ✅ `beta0.1.0` 提供 | 需自行构建（见下） |

**本次 `beta0.1.0` 发布的是 `bootstrap` 变体**：一个真正用自有证书签名、可直接安装、包名 `dev.wonderbot.android` 的 APK，能打开深黑主题的原生管理界面。但它**不含那约 1 GiB 的 ARM64 Ubuntu 完整运行镜像**，所以装完还不能立刻在手机上把机器人跑起来。

完整镜像不进 Git（体积过大，且需 ARM64 root + `qemu-user-static` + `dpkg-deb` 现场生成），上游也只在 Release 里发成品、不发可复现的构建输入。生成 `bundled` 完整包的**完整步骤、所需输入、一键命令与签名要求**都写在 [`BUILD_WONDERBOT.md`](./BUILD_WONDERBOT.md)。这一点如实说明，不假装已经做到。

## 本版本做了哪些二创改动

1. **品牌**：应用名「AstrBot 运行管理器」→「WonderBot 运行管理器」，界面内所有面向用户的品牌文案同步替换。
2. **应用标识**：包名 `io.astrbot.android` → **`dev.wonderbot.android`**（含 `namespace`、`applicationId`、Kotlin/aidl 源码目录）。因此与上游版本**不能覆盖安装、可以共存**。
3. **图标**：全套替换为 WonderBot 自有图标（各密度启动图标 + 自适应图标前景），背景色改深黑 `#08080B`。
4. **配色**：蓝色 / 亮色主题 → **深黑主题**（背景 `#08080B`、面板 `#121218`、强调色 `#5EE0C8`），不跟随系统明暗，始终深黑。
5. **更新通道**：三个在线更新目录**默认留空 = 默认不检查在线更新**，避免误连上游官方通道被检测到官方更新；`channelBase` 指向本仓库，需启用时以 `-PupdateCatalogUrl=` 等参数指定自有签名目录。上游更新公钥不再内置。
6. **版本**：`beta0.1.0`（versionCode 1）。
7. **签名**：使用本项目自有应用签名证书（PKCS12，别名 `release`）。签名文件放在被 Git 忽略的 `.local/signing/`，**不进入源码仓库**。
8. **国内源**：Gradle 发行版、Maven 依赖、Android SDK 组件、JDK 17 均可走清华 / 阿里 / 腾讯云镜像配置，见 [`BUILD_WONDERBOT.md`](./BUILD_WONDERBOT.md)。

完整的上游使用说明、界面预览、SnowLuma 部署与常见问题，见 [`docs/README.upstream.md`](./docs/README.upstream.md)、[`docs/使用指南.md`](./docs/使用指南.md) 与 [`docs/构建指南.md`](./docs/构建指南.md)。

## 刻意保留、没有改动的内容

改动会导致运行失败或违反许可，因此以下内容**原样保留**：

- **PRoot 内部路径与环境变量契约**：`/opt/astrbot`、`persistent/astrbot/data`、`opt/astrbot-venv`、`launch_astrbot.py`、`ASTRBOT_ROOT`、`ASTRBOT_DESKTOP_*`、`ServiceId.ASTRBOT` 等。它们是 App 与 Ubuntu 环境内脚本之间的**接口**，不是品牌名。
- **上游 MIT 版权声明**：`LICENSE`、`docs/legal/NOTICE`、`docs/legal/AUTHORS.md`、`docs/legal/THIRD_PARTY_NOTICES.md` 及若干 Kotlin 文件头的 `Copyright (c) 2026 JunWan666`。MIT 要求分发时保留版权声明与许可文本。
- **上游来源标识与锁文件**：`AstrBotDevs`、`astrbot.app` 及 `runtime/` 下所有来源锁（含 SHA-256 摘要），是构建期完整性校验输入，改动会导致 `preBuild` 校验失败。
- **第三方组件名称与许可**：NapCat、Shizuku、Ubuntu、PRoot、cpolar、SnowLuma、xterm.js、DeepSeek Harness 等。

完整清单见 [`WONDERBOT.md`](./WONDERBOT.md)。

## 下载与安装

从 [Releases · beta0.1.0](https://github.com/2997669349awa-blip/WonderBotAndroid/releases/tag/beta0.1.0) 下载 `WonderBot-Android-beta0.1.0.apk`：

- 要求 **Android 8.0+（minSdk 26）、ARM64 设备**；targetSdk 35。
- 首次安装请在系统设置里允许「安装未知来源应用」。
- 用 `SHA256SUMS` 校验完整性。
- 由于签名与包名不同于上游，**可与上游 AstrBot for Android 同时安装**。

验证签名（可选）：

```bash
apksigner verify --print-certs WonderBot-Android-beta0.1.0.apk
# 证书 SHA-256: 6e:b8:e4:60:8a:c1:c2:d4:97:37:4d:6d:14:76:d9:ac:c2:d4:aa:cd:59:56:8b:c3:f4:6f:67:dc:73:16:27:49
```

## 从源码构建

环境：JDK 17、Android SDK（platform-35 + build-tools 35.0.0）、Gradle 8.11.1。国内网络建议按 [`BUILD_WONDERBOT.md`](./BUILD_WONDERBOT.md) 配置镜像。

```bash
# 1) 生成本地签名（首次，文件不进 Git）
mkdir -p .local/signing
#   放入 release.p12 与 release.properties（别名 release）

# 2) 指定 SDK 路径
echo "sdk.dir=/path/to/android-sdk" > local.properties

# 3) 构建 bootstrap Release APK
gradle :app:assembleBootstrapRelease \
  -PappVersionName=beta0.1.0 -PappVersionCode=1
# 产物：app/build/outputs/apk/bootstrap/release/app-bootstrap-release.apk
```

`bundled` 完整包需要额外准备 ARM64 运行环境资产，命令与校验流程见构建指南。

## 项目结构

```
WonderBotAndroid/
├── app/                     # Kotlin + Compose 原生管理器（包名 dev.wonderbot.android）
│   └── src/main/
│       ├── java/dev/wonderbot/android/   # 身份常量、界面、运行时控制、WebUI 容器
│       ├── res/                          # 深黑主题、WonderBot 图标、多语言文案
│       └── assets/runtime/harness/       # 运行环境安装配方
├── core/                    # 原生公共模块
├── runtime/                 # 来源锁文件、品牌资产定义、provision 脚本
├── tools/                   # 构建 / 校验 / 发行脚本
├── docs/                    # 使用、部署、更新与分发文档 + 合规声明
├── WONDERBOT.md             # 二创与许可说明
├── BUILD_WONDERBOT.md       # 完整包构建与国内镜像配置
└── LICENSE / NOTICE         # MIT 许可与第三方声明
```

## 许可与致谢

| 组件 | 来源 | 许可 |
| --- | --- | --- |
| 上游 Android 管理器 | [`JunWan666/astrbot-for-android`](https://github.com/JunWan666/astrbot-for-android) | MIT · Copyright (c) 2026 JunWan666 |
| 机器人程序 | [`AstrBotDevs/AstrBot`](https://github.com/AstrBotDevs/AstrBot) | AGPL-3.0-or-later |
| QQ 协议端 | [`NapNeko/NapCatQQ`](https://github.com/NapNeko/NapCatQQ) | 见上游仓库 |
| 本衍生版本修改部分 | 本仓库 | MIT · Copyright (c) 2026 WonderBot Team |

- 上游 Android 管理器原创代码与文档仍为 MIT，按许可证要求保留声明；本仓库新增/修改部分同为 MIT。
- 机器人程序派生自 AstrBot（AGPL-3.0-or-later）。**若分发含该程序的完整运行镜像，须遵守 AGPL 相应义务，包括 §13 网络交互源码提供义务**。
- 本仓库的 MIT 许可**不重新授权**任何第三方组件，也不授予分发第三方二进制的许可。
- 感谢 [AstrBot](https://github.com/AstrBotDevs/AstrBot)、[astrbot-for-android](https://github.com/JunWan666/astrbot-for-android) 及所有上游贡献者。

## 相关链接

- 主项目（Python 服务端）：<https://github.com/2997669349awa-blip/wonderBot>
- 官网：<https://2997669349awa-blip.github.io/wonderBot/>
- 下载：<https://github.com/2997669349awa-blip/WonderBotAndroid/releases>
- 问题反馈：<https://github.com/2997669349awa-blip/WonderBotAndroid/issues>

---

<div align="center">

**WonderBot for Android** · beta0.1.0 · MIT（衍生修改部分）

上游原创代码 Copyright © 2026 JunWan666 · 衍生修改 Copyright © 2026 WonderBot Team

</div>
