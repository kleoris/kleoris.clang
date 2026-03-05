# kleoris.clang

[![Build CLANG 21](https://github.com/kleoris/kleoris.clang/actions/workflows/build_clang.yml/badge.svg)](https://github.com/kleoris/kleoris.clang/actions/workflows/build_clang.yml)
![C++26](https://img.shields.io/badge/C%2B%2B-26-blue)
![CLANG 21](https://img.shields.io/badge/CLANG-21)
![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey)
![License](https://img.shields.io/badge/license-GPLv3-green)

CI toolchain builder for **CLANG 21 (with C++26 P2996 static-reflection support)** (`-freflection`).

This repository contains nothing but a GitHub Actions workflow that builds CLANG 21 from source for all three supported platforms and publishes the result as a downloadable artifact. It is a companion to [kleoris](https://github.com/kleoris/kleoris_cpp), which requires a C++26 compiler with P2996 support.

## What gets built

- Build system: CMake + Ninja (single-stage Release build)
- Enabled projects: `clang` only
- Targets: `host` only (no cross-compilation artefacts)
- Key flag: `-freflection` (P2996 compile-time static reflection)
- Assertions, tests, and benchmarks: disabled (CI speed)

## Install locations

| Platform | Prefix |
|---|---|
| macOS | `/opt/homebrew/clang-21` |
| Linux | `/usr/local/clang-21` |
| Windows | `C:/Dev/clang-21` |

Binaries are installed as `clang` / `clang++`.

## Platforms

| Artifact | Runner | Source |
|---|---|---|
| `clang-macos-clang-21` | `macos-latest` (Apple Silicon) | [bloomberg/clang-p2996](https://github.com/bloomberg/clang-p2996) |
| `clang-linux-clang-21` | `ubuntu-latest` | [bloomberg/clang-p2996](https://github.com/bloomberg/clang-p2996) |
| `clang-win-clang-21` | `windows-latest` | [bloomberg/clang-p2996](https://github.com/bloomberg/clang-p2996) |

## Github workflow usage

1. Go to **Actions → Build CLANG 21 with P2996 support**.
2. Click **Run workflow**.
3. Select a build target (see [Selecting a single platform](#selecting-a-single-platform)).
4. Optionally override the source branch/tag (`clang_ref_default`, defaults to `p2996`).
5. Optionally provide a `release_tag` (e.g. `v21.0.0-p2996-1`) to publish a GitHub release with all platform tarballs attached. Leave blank to produce artifact-only builds.
6. Once complete, download the `.tar.xz` artifact for your platform from the run summary.

## Selecting a single platform

The `platform` input accepts `all` (default) or any single platform name:

```
all              — builds all three matrix entries in parallel
macos-clang-21   — macOS (Apple Silicon) only
linux-clang-21   — Linux only
win-clang-21     — Windows only
```

Internally a lightweight `prepare` job runs first and uses `jq` to filter the full platform list down to the requested entry. The `main` build job then consumes this filtered JSON as its dynamic strategy matrix via `fromJSON(needs.prepare.outputs.matrix)`. This sidesteps the GitHub Actions limitation that the `matrix` context is unavailable in a job-level `if` condition.

## All-platforms note

All three platforms check out the same [bloomberg/clang-p2996](https://github.com/bloomberg/clang-p2996) repository. Dependencies differ per platform:

- **macOS**: `cmake`, `ninja`, `python3` via Homebrew
- **Linux**: `cmake`, `ninja-build`, `python3`, `build-essential` via apt
- **Windows**: `ninja` via Chocolatey (MSVC toolchain provided by the runner)

## Build notes

* `LLVM_ENABLE_PROJECTS=clang` — only the Clang front-end is compiled; no `lld`, `lldb`, or extra runtimes, keeping build times manageable in CI.
* `LLVM_TARGETS_TO_BUILD=host` — only the native target backend is included, avoiding cross-compilation overhead.
* `LLVM_INCLUDE_TESTS=OFF` / `DCLANG_INCLUDE_TESTS=OFF` — test infrastructure is excluded to further reduce build time and artefact size.
* `DLLVM_INCLUDE_BENCHMARKS=OFF` — only useful for dev.
* `DLLVM_ENABLE_ASSERTIONS=ON` — the assertions are ON despite the overhead because this is a very experimental compiler
* `DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind"` — using the clang unwinder rather than the system's
