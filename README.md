# ffmpeg-builder

`ffmpeg-builder` builds a real, minimal FFmpeg executable with static x264 support for
`pagx export --format mp4`. It is self-contained and can be used as a standalone
repository without depending on the parent libpag repository.

The build is intentionally limited to the components used by pagx:

- raw RGBA video from a file or stdin pipe;
- H.264 encoding through libx264;
- scale and format conversion filters;
- MP4 muxing with fast-start support.

Network access, devices, other codecs, ffplay, ffprobe, documentation, and external
library auto-detection are disabled. The resulting executable uses no external x264
or FFmpeg shared libraries, but it may still use normal operating-system libraries.

## Pinned sources

`DEPS` fixes both source revisions. Do not replace them with moving branches.

| Project | Version | Commit |
| --- | --- | --- |
| FFmpeg | n7.1.2 | `f893221c8d89cb798b829bebe71d55e1a3f242fd` |
| x264 | stable | `b35605ace3ddf7c1a5d67a2eb553f034aef41d55` |

## Prerequisites

All platforms require Git, a C compiler, Make, pkg-config, Node.js, and depsync.
Install depsync with:

```bash
npm install -g depsync
```

x86 and x64 targets also require NASM. Cross-compilation is supported between
architectures on the same host OS, but not between operating systems. The supported
target matrix is:

| Host | Targets | Toolchain |
| --- | --- | --- |
| macOS x64 or ARM64 | macOS x64, macOS ARM64 | Xcode Clang with `-arch` |
| Linux x64 or ARM64 | Linux x64, Linux ARM64 | Native GCC or the matching GNU cross toolchain |
| Windows | Windows x86, x64, ARM64 | MSYS2 MINGW32, MINGW64/UCRT64, CLANGARM64 |

Set `CC`, `AR`, `RANLIB`, and `STRIP` when using a custom cross toolchain. `CC` must
name the compiler executable without additional arguments; use `CFLAGS` and `LDFLAGS`
for extra flags.

### macOS

```bash
xcode-select --install
brew install nasm pkg-config node
```

### Linux

On Debian or Ubuntu:

```bash
sudo apt-get install build-essential git nasm pkg-config nodejs npm
```

Other distributions need the equivalent development tools. For an ARM64 target on an
x64 Ubuntu host, also install `gcc-aarch64-linux-gnu` and `binutils-aarch64-linux-gnu`.
For an x64 target on an ARM64 Ubuntu host, install `gcc-x86-64-linux-gnu` and
`binutils-x86-64-linux-gnu`. The script selects the corresponding GNU cross-prefix
automatically when the host and target architectures differ.

### Windows

Install MSYS2, then install the x64 toolchain in an MSYS2 shell:

```bash
pacman -S --needed base-devel git make nasm pkgconf nodejs npm \
  mingw-w64-x86_64-toolchain
```

For Windows x86, install the 32-bit MinGW toolchain:

```bash
pacman -S --needed base-devel git make nasm pkgconf nodejs npm \
  mingw-w64-i686-toolchain
```

For Windows ARM64, install the LLVM-MinGW toolchain instead:

```bash
pacman -S --needed base-devel git make pkgconf nodejs npm \
  mingw-w64-clang-aarch64-toolchain
```

`build.ps1` selects `MINGW32` for x86, `MINGW64` for x64, and `CLANGARM64` for ARM64.
Set `MSYS2_ROOT` if MSYS2 is not installed at `C:\msys64`.

## Build

macOS and Linux:

```bash
./build.sh                         # Detect the host platform and architecture
./build.sh --platform mac --arch arm64 --jobs 8
./build.sh --platform mac --arch x64   # Cross-compile on an ARM64 Mac
./build.sh --platform linux --arch x64
./build.sh --platform linux --arch arm64
```

Windows PowerShell:

```powershell
./build.ps1 -Arch x64 -Jobs 8
./build.ps1 -Arch x86
./build.ps1 -Arch arm64
```

Windows can also use `build.sh --platform win` directly from the matching MSYS2
environment. Sources are synchronized before each normal build. To rebuild existing
sources without network access, pass `--skip-sync`.

The output layout is:

```text
out/<platform>-<arch>/
├── bin/ffmpeg[.exe]
├── build-info.json
└── share/licenses/
```

`build-info.json` records the actual source commits, configure arguments, host, target,
toolchain, validation level, and SHA-256 digest of the executable. Every build checks
the generated FFmpeg configuration for all required components. A native build also
checks component availability at runtime and encodes three generated RGBA frames into
a real MP4 through stdin. A cross-compiled build is marked `configuration-only` because
the target executable is not run on the host; run it on the target OS before release.

To remove one target's intermediate files and output:

```bash
./build.sh --platform mac --arch arm64 --clean
```

On Windows, use `./build.ps1 -Arch x64 -Clean`.

## GitHub Actions

`.github/workflows/build-release.yml` builds all seven desktop targets on native GitHub
runners. Pushes to `main`, pull requests, and manual runs upload temporary artifacts.
Pushing a tag matching `v*` additionally creates a GitHub Release after every target
succeeds, attaches all packages, and generates `SHA256SUMS`.

The Release job uses the workflow's `GITHUB_TOKEN` with `contents: write` permission;
no personal access token is required. Existing releases are not overwritten. To publish
a version:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The generated packages are named `ffmpeg-<platform>-<arch>.tar.gz` on Unix and
`ffmpeg-<platform>-<arch>.zip` on Windows. Windows x86 is built on an x64 runner and is
therefore marked `configuration-only`; test that executable on a compatible Windows
system before distribution. The other matrix entries run their native smoke tests.

## Download built executables

### Local builds

After running `build.sh` or `build.ps1`, use the executable directly from:

```text
out/<platform>-<arch>/bin/ffmpeg[.exe]
```

For example, a native Apple Silicon build produces `out/mac-arm64/bin/ffmpeg`.

### GitHub Actions artifacts

Open the repository's **Actions** page, select a successful **Build and release** run,
and download `ffmpeg-<platform>-<arch>` from its **Artifacts** section. These artifacts
are temporary, retained for seven days, and may require a GitHub login. They are intended
for CI verification rather than permanent download links.

### Release assets

Asset names do not contain a Release tag. Use the following stable URLs to download the
latest published Release without knowing its tag. These links become available after the
first Release is published:

| Target | Latest download URL |
| --- | --- |
| macOS ARM64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-mac-arm64.tar.gz |
| macOS x64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-mac-x64.tar.gz |
| Linux ARM64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-linux-arm64.tar.gz |
| Linux x64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-linux-x64.tar.gz |
| Windows x86 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-win-x86.zip |
| Windows x64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-win-x64.zip |
| Windows ARM64 | https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-win-arm64.zip |

For example:

```bash
curl -fL -o ffmpeg-mac-arm64.tar.gz \
  https://github.com/libpag/ffmpeg-builder/releases/latest/download/ffmpeg-mac-arm64.tar.gz
tar -xzf ffmpeg-mac-arm64.tar.gz
./bin/ffmpeg -version
```

The checksum file is available at:

```text
https://github.com/libpag/ffmpeg-builder/releases/latest/download/SHA256SUMS
```

To pin an exact build, replace `latest` with its Release tag:

```text
https://github.com/libpag/ffmpeg-builder/releases/download/<tag>/ffmpeg-<platform>-<arch>.<ext>
```

Release tags identify revisions of `DEPS`, build scripts, packaging logic, and the exact
FFmpeg/x264 commits used for the build. They do not imply that this repository patches
FFmpeg. Keeping the tag optional gives latest-download users a stable URL while allowing
reproducible builds to pin an exact snapshot even when the upstream FFmpeg version has
not changed.

## Use with pagx

Pass the executable explicitly:

```bash
pagx export input.pagx --format mp4 --ffmpeg /absolute/path/to/ffmpeg output.mp4
```

Or set `FFMPEG_BIN` to the absolute executable path. This directory does not alter the
pagx CLI, its PATH lookup, or the npm package layout.

## Licensing

x264 is licensed under the GNU GPL, and this build enables FFmpeg's GPL components.
No non-free component is enabled. The generated program is therefore subject to the
GPL; review the copied FFmpeg and x264 license files and satisfy source-code and
license obligations before redistributing any executable. This note is not legal advice.
