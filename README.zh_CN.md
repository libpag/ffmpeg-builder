# ffmpeg-builder

[English](README.md) | [简体中文](README.zh_CN.md)

`ffmpeg-builder` 用于构建真实、最小化且静态集成 x264 的 FFmpeg 可执行文件，服务于
`pagx export --format mp4`。本仓库可以独立使用，不依赖父级 libpag 仓库。

构建内容仅限 pagx 所需的组件：

- 从文件或标准输入管道读取 raw RGBA 视频；
- 通过 libx264 进行 H.264 编码；
- 使用 scale 和 format 滤镜转换图像；
- 封装支持 faststart 的 MP4 文件。

网络访问、设备、其他编解码器、ffplay、ffprobe、文档和外部库自动检测均被禁用。
生成的可执行文件不依赖外部 x264 或 FFmpeg 动态库，但仍可能依赖正常的操作系统库。

## 固定源码版本

`DEPS` 固定了两个源码项目的提交，不要改成持续移动的分支。

| 项目 | 版本 | 提交 |
| --- | --- | --- |
| FFmpeg | n7.1.2 | `f893221c8d89cb798b829bebe71d55e1a3f242fd` |
| x264 | stable | `b35605ace3ddf7c1a5d67a2eb553f034aef41d55` |

## 环境要求

所有平台都需要 Git、C 编译器、Make、pkg-config、Node.js 和 depsync。安装 depsync：

```bash
npm install -g depsync
```

x86 和 x64 目标还需要 NASM。支持在同一操作系统内跨架构编译，不支持跨操作系统
编译。支持的目标矩阵如下：

| 宿主机 | 构建目标 | 工具链 |
| --- | --- | --- |
| macOS x64 或 ARM64 | macOS x64、macOS ARM64 | 使用 `-arch` 的 Xcode Clang |
| Linux x64 或 ARM64 | Linux x64、Linux ARM64 | 原生 GCC 或对应的 GNU 交叉工具链 |
| Windows | Windows x86、x64、ARM64 | MSYS2 MINGW32、MINGW64/UCRT64、CLANGARM64 |

使用自定义交叉工具链时，可设置 `CC`、`AR`、`RANLIB` 和 `STRIP`。`CC` 只能指定
编译器可执行文件，不能附带额外参数；额外参数通过 `CFLAGS` 和 `LDFLAGS` 传入。

### macOS

```bash
xcode-select --install
brew install nasm pkg-config node
```

### Linux

Debian 或 Ubuntu：

```bash
sudo apt-get install build-essential git nasm pkg-config nodejs npm
```

其他发行版需要安装等价开发工具。在 x64 Ubuntu 宿主机上构建 ARM64 目标时，还需
安装 `gcc-aarch64-linux-gnu` 和 `binutils-aarch64-linux-gnu`；在 ARM64 Ubuntu
宿主机上构建 x64 目标时，还需安装 `gcc-x86-64-linux-gnu` 和
`binutils-x86-64-linux-gnu`。宿主机与目标架构不同时，脚本会自动选择对应的 GNU
cross-prefix。

### Windows

安装 MSYS2，然后在 MSYS2 Shell 中安装 x64 工具链：

```bash
pacman -S --needed base-devel git make nasm pkgconf nodejs npm \
  mingw-w64-x86_64-toolchain
```

构建 Windows x86 时，安装 32 位 MinGW 工具链：

```bash
pacman -S --needed base-devel git make nasm pkgconf nodejs npm \
  mingw-w64-i686-toolchain
```

构建 Windows ARM64 时，安装 LLVM-MinGW 工具链：

```bash
pacman -S --needed base-devel git make pkgconf nodejs npm \
  mingw-w64-clang-aarch64-toolchain
```

`build.ps1` 对 x86 选择 `MINGW32`，对 x64 选择 `MINGW64`，对 ARM64 选择
`CLANGARM64`。如果 MSYS2 未安装在 `C:\msys64`，请设置 `MSYS2_ROOT`。

## 构建

macOS 和 Linux：

```bash
./build.sh                         # Detect the host platform and architecture
./build.sh --platform mac --arch arm64 --jobs 8
./build.sh --platform mac --arch x64   # Cross-compile on an ARM64 Mac
./build.sh --platform linux --arch x64
./build.sh --platform linux --arch arm64
```

Windows PowerShell：

```powershell
./build.ps1 -Arch x64 -Jobs 8
./build.ps1 -Arch x86
./build.ps1 -Arch arm64
```

Windows 也可以在匹配的 MSYS2 环境中直接运行 `build.sh --platform win`。正常构建前
会同步源码；如需使用已有源码离线重建，请传入 `--skip-sync`。

产物目录结构：

```text
out/<platform>-<arch>/
├── bin/ffmpeg[.exe]
├── build-info.json
├── LICENSE.txt
├── THIRD_PARTY_NOTICES.md
├── THIRD_PARTY_NOTICES.zh_CN.md
└── share/licenses/
```

`build-info.json` 记录实际源码提交、configure 参数、宿主机、目标、工具链、验证级别
和可执行文件的 SHA-256。每次构建都会检查生成的 FFmpeg 配置是否包含全部必要组件。
原生构建还会在运行时检查组件可用性，并通过标准输入将三帧生成的 RGBA 图像编码为
真实 MP4。交叉构建标记为 `configuration-only`，因为宿主机不运行目标可执行文件；
发布前必须在目标操作系统上验证。

清理某个目标的中间文件和产物：

```bash
./build.sh --platform mac --arch arm64 --clean
```

Windows 使用 `./build.ps1 -Arch x64 -Clean`。

## GitHub Actions

`.github/workflows/build-release.yml` 在原生 GitHub Runner 上构建全部七个桌面目标。
推送到 `main`、Pull Request 和手动运行会上传临时 Artifact。推送匹配 `v*` 的标签时，
流水线会在全部目标成功后创建 GitHub Release，附加所有安装包并生成 `SHA256SUMS`。

Release 任务使用工作流自带且具有 `contents: write` 权限的 `GITHUB_TOKEN`，不需要
个人访问令牌，也不会覆盖已有 Release。发布版本：

```bash
git tag v1.0.0
git push origin v1.0.0
```

Unix 安装包命名为 `ffmpeg-<platform>-<arch>.tar.gz`，Windows 安装包命名为
`ffmpeg-<platform>-<arch>.zip`。Windows x86 在 x64 Runner 上构建，因此标记为
`configuration-only`；分发前需在兼容的 Windows 系统上测试。其他矩阵目标会执行各自
的原生冒烟测试。

## 下载构建产物

### 本地构建

运行 `build.sh` 或 `build.ps1` 后，可直接使用以下路径中的可执行文件：

```text
out/<platform>-<arch>/bin/ffmpeg[.exe]
```

例如，Apple Silicon 原生构建会生成 `out/mac-arm64/bin/ffmpeg`。

### GitHub Actions Artifact

打开仓库的 **Actions** 页面，选择一次成功的 **Build and release** 工作流运行，在
**Artifacts** 区域下载 `ffmpeg-<platform>-<arch>`。这些 Artifact 仅保留七天，并且
可能要求登录 GitHub，适用于 CI 验证，不适合作为永久下载链接。

### Release Assets

资产文件名不包含 Release 标签。首个 Release 发布后，可通过以下固定链接下载最新
Release，无需知道其标签：

| 目标 | 最新版本下载链接 |
| --- | --- |
| macOS ARM64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-mac-arm64.tar.gz |
| macOS x64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-mac-x64.tar.gz |
| Linux ARM64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-linux-arm64.tar.gz |
| Linux x64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-linux-x64.tar.gz |
| Windows x86 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-win-x86.zip |
| Windows x64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-win-x64.zip |
| Windows ARM64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-win-arm64.zip |

例如：

```bash
curl -fL -o ffmpeg-mac-arm64.tar.gz \
  https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-mac-arm64.tar.gz
tar -xzf ffmpeg-mac-arm64.tar.gz
./bin/ffmpeg -version
```

校验文件下载地址：

```text
https://github.com/libpag/ffmpeg-builder/releases/latest/download/SHA256SUMS
```

每个 Release 还提供精确的对应源码和许可声明：

| 资产 | 最新版本下载链接 |
| --- | --- |
| FFmpeg 源码 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-source.tar.gz |
| x264 源码 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/x264-source.tar.gz |
| ffmpeg-builder 源码 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-builder-source.tar.gz |
| 英文第三方声明 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/THIRD_PARTY_NOTICES.md |
| 中文第三方声明 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/THIRD_PARTY_NOTICES.zh_CN.md |

源码工作流会先根据 `DEPS` 校验每个依赖的实际提交，再通过 `git archive HEAD`
创建源码包。只有二进制构建和源码打包任务全部成功后，流水线才会创建 Release。

如需固定某个精确构建，将链接中的 `latest` 替换为 Release 标签：

```text
https://github.com/libpag/ffmpeg-builder/releases/download/<tag>/ffmpeg-<platform>-<arch>.<ext>
```

Release 标签用于标识 `DEPS`、构建脚本、打包逻辑以及实际使用的 FFmpeg/x264 提交，
并不表示本仓库修改了 FFmpeg。标签可选的设计让下载最新产物的用户拥有固定 URL，
同时让需要可复现构建的用户能够固定精确快照，即使上游 FFmpeg 版本没有变化也不受影响。

## 与 pagx 配合使用

显式传入可执行文件路径：

```bash
pagx export input.pagx --format mp4 --ffmpeg /absolute/path/to/ffmpeg output.mp4
```

也可以将 `FFMPEG_BIN` 设置为可执行文件的绝对路径。本仓库不会修改 pagx CLI、其
PATH 查找逻辑或 npm 包目录结构。

## 开源协议与第三方许可

本仓库中的构建脚本、文档和自动化配置采用 **Apache License 2.0**，完整协议见
[LICENSE.txt](LICENSE.txt)。该协议仅适用于 ffmpeg-builder 自身，不会改变 FFmpeg、
x264 或其他第三方项目的许可证。精确提交、对应源码包和进程边界详见
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) 或
[中文第三方声明](THIRD_PARTY_NOTICES.zh_CN.md)。

x264 采用 GNU GPL，当前构建同时启用了 FFmpeg 的 GPL 组件，未启用 non-free 组件。
因此生成的 FFmpeg 可执行文件需要按照 **GPL** 分发。每个产物的
`share/licenses/` 中包含对应的 FFmpeg 和 x264 许可证。分发任何可执行文件前，请审查
这些许可证并履行相应的源码和许可义务。本说明不构成法律意见。
