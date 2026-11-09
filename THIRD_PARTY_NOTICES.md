# Third-Party Software Notices

[English](THIRD_PARTY_NOTICES.md) | [简体中文](THIRD_PARTY_NOTICES.zh_CN.md)

## Scope

This notice applies to the executable packages produced by ffmpeg-builder. The builder's
scripts, documentation, and automation are licensed under Apache License 2.0, but that
license does not apply to FFmpeg, x264, or the resulting FFmpeg executable.

## FFmpeg

- **Project**: FFmpeg
- **Pinned version**: n7.1.2
- **Commit**: `f893221c8d89cb798b829bebe71d55e1a3f242fd`
- **Source repository**: https://github.com/FFmpeg/FFmpeg
- **Exact revision**: https://github.com/FFmpeg/FFmpeg/commit/f893221c8d89cb798b829bebe71d55e1a3f242fd
- **License for this build**: GNU GPL version 2 or later, because the build uses
  `--enable-gpl` and links x264
- **License copy in executable packages**: `share/licenses/FFmpeg-LICENSE.md`

FFmpeg licensing depends on build configuration. No non-free component is enabled by
this repository. The exact configure arguments are recorded in each package's
`build-info.json`.

## x264

- **Project**: x264
- **Pinned branch name at synchronization time**: stable
- **Commit**: `b35605ace3ddf7c1a5d67a2eb553f034aef41d55`
- **Source repository**: https://code.videolan.org/videolan/x264
- **Exact revision**: https://code.videolan.org/videolan/x264/-/commit/b35605ace3ddf7c1a5d67a2eb553f034aef41d55
- **License**: GNU GPL version 2 or later
- **License copy in executable packages**: `share/licenses/x264-COPYING`

The generated FFmpeg executable statically links x264. It does not require a separate
x264 shared library at runtime.

## Corresponding source code

Every GitHub Release includes these corresponding source assets:

| Asset | Contents |
| --- | --- |
| `ffmpeg-source.tar.gz` | The exact FFmpeg commit used by all binaries in the Release |
| `x264-source.tar.gz` | The exact x264 commit used by all binaries in the Release |
| `ffmpeg-builder-source.tar.gz` | The builder sources, DEPS, scripts, documentation, and workflow for the same Release |

The release workflow verifies each checked-out dependency commit against `DEPS` and then
uses `git archive HEAD` to generate the upstream source archives. The current build does
not apply source patches to FFmpeg or x264. A generic upstream URL or a similar version
is not substituted for these exact source archives.

The same assets are available under `/releases/latest/download/` for the latest Release
and under `/releases/download/<tag>/` for a pinned Release. Each executable's
`build-info.json` records the source commits and configure arguments that connect it to
these archives.

## Process integration

The generated FFmpeg program is designed to run as a separate executable. Host applications
can communicate with it through command-line arguments, stdin, and output files without
linking x264 or the FFmpeg libraries. The host application and the GPL FFmpeg executable
retain their respective license boundaries.

This notice is provided for license clarity and does not constitute legal advice.
