# 第三方软件许可声明

[English](THIRD_PARTY_NOTICES.md) | [简体中文](THIRD_PARTY_NOTICES.zh_CN.md)

## 适用范围

本声明适用于 ffmpeg-builder 生成的可执行文件包。构建脚本、文档和自动化配置采用
Apache License 2.0，但该协议不适用于 FFmpeg、x264 或最终生成的 FFmpeg 可执行文件。

## FFmpeg

- **项目**：FFmpeg
- **固定版本**：n7.1.2
- **提交**：`f893221c8d89cb798b829bebe71d55e1a3f242fd`
- **源码仓库**：https://github.com/FFmpeg/FFmpeg
- **精确提交**：https://github.com/FFmpeg/FFmpeg/commit/f893221c8d89cb798b829bebe71d55e1a3f242fd
- **当前构建的许可证**：GNU GPL 第 2 版或更新版本，因为构建使用了
  `--enable-gpl` 并链接 x264
- **可执行文件包中的协议副本**：`share/licenses/FFmpeg-LICENSE.md`

FFmpeg 的许可证取决于构建配置。本仓库未启用 non-free 组件，具体 configure 参数
记录在每个包的 `build-info.json` 中。

## x264

- **项目**：x264
- **同步时使用的分支名称**：stable
- **提交**：`b35605ace3ddf7c1a5d67a2eb553f034aef41d55`
- **源码仓库**：https://code.videolan.org/videolan/x264
- **精确提交**：https://code.videolan.org/videolan/x264/-/commit/b35605ace3ddf7c1a5d67a2eb553f034aef41d55
- **许可证**：GNU GPL 第 2 版或更新版本
- **可执行文件包中的协议副本**：`share/licenses/x264-COPYING`

生成的 FFmpeg 可执行文件静态链接 x264，运行时不需要额外的 x264 动态库。

## 对应源码

每个 GitHub Release 都包含以下对应源码资产：

| 资产 | 内容 |
| --- | --- |
| `ffmpeg-source.tar.gz` | 该 Release 中全部二进制实际使用的 FFmpeg 提交 |
| `x264-source.tar.gz` | 该 Release 中全部二进制实际使用的 x264 提交 |
| `ffmpeg-builder-source.tar.gz` | 同一 Release 对应的构建器源码、DEPS、脚本、文档和工作流 |

Release 工作流先根据 `DEPS` 校验每个依赖仓库的实际提交，再使用 `git archive HEAD`
生成上游源码包。当前构建不会对 FFmpeg 或 x264 应用源码补丁，不会用通用上游链接
或相近版本代替这些精确源码包。

最新 Release 的相同资产位于 `/releases/latest/download/`，固定版本位于
`/releases/download/<tag>/`。每个可执行文件的 `build-info.json` 都记录源码提交和
configure 参数，用于建立二进制与这些源码包之间的对应关系。

## 进程集成方式

生成的 FFmpeg 程序作为独立可执行文件运行。宿主应用可以通过命令行参数、标准输入
和输出文件与其通信，无需链接 x264 或 FFmpeg 库。宿主应用与 GPL 的 FFmpeg 可执行
文件保持各自的许可证边界。

本声明用于澄清许可范围，不构成法律意见。
