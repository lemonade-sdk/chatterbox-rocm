# chatterbox-rocm

Distribution repository for the self-contained **chatterbox-server** bundles
consumed by [Lemonade](https://github.com/lemonade-sdk/lemonade)'s `chatterbox`
text-to-speech backend.

**No Chatterbox code is forked or vendored here.** At build time:

- [`chatterbox-tts`](https://pypi.org/project/chatterbox-tts/) is installed from
  PyPI (Resemble AI's open-source TTS library),
- the device-appropriate **PyTorch** wheel is installed over it (CUDA, ROCm, or
  the default CPU/MPS wheel),
- the thin `main.py` server wrapper is checked out from
  [`lemonade-sdk/lemonade`](https://github.com/lemonade-sdk/lemonade/tree/main/tools/chatterbox-server),

and the result is frozen into a PyInstaller onedir bundle with an embedded
Python 3.11 runtime — no system Python is required (or touched) on user
machines.

> Despite the `-rocm` name (kept consistent with the sibling
> [`moonshine-server-rocm`](https://github.com/lemonade-sdk/moonshine-server-rocm)
> repo), this repo publishes bundles for **all** devices: CUDA, ROCm, Metal, and
> CPU.

## Releases

The [build workflow](.github/workflows/build-release.yml) polls PyPI daily. When
a new `chatterbox-tts` version appears, it builds and publishes a release tagged
`chatterbox<version>` (e.g. `chatterbox0.1.7`) with one asset per
platform/device:

| Platform | Device | Asset |
|----------|--------|-------|
| Linux x64 | CUDA | `chatterbox-server-<tag>-linux-x64-cuda.tar.gz` |
| Linux x64 | ROCm | `chatterbox-server-<tag>-linux-x64-rocm.tar.gz` |
| Linux x64 | CPU | `chatterbox-server-<tag>-linux-x64-cpu.tar.gz` |
| Windows x64 | CUDA | `chatterbox-server-<tag>-windows-x64-cuda.zip` |
| Windows x64 | CPU | `chatterbox-server-<tag>-windows-x64-cpu.zip` |
| macOS arm64 | Metal | `chatterbox-server-<tag>-macos-arm64-metal.tar.gz` |
| macOS arm64 | CPU | `chatterbox-server-<tag>-macos-arm64-cpu.tar.gz` |

There is no Windows-ROCm asset because PyTorch publishes ROCm wheels for Linux
only. A single PyTorch wheel covers all GPU architectures of its vendor, so
there is no per-`sm_`/per-`gfx` split — device is the only axis.

Lemonade pins the release tag it consumes in
[`backend_versions.json`](https://github.com/lemonade-sdk/lemonade/blob/main/src/cpp/resources/backend_versions.json)
(`chatterbox.{cuda,rocm,metal,cpu}`). New upstream versions are built here
automatically, but Lemonade only switches when those pins are bumped via PR.

Builds can also be triggered manually from the Actions tab (with an optional
explicit `chatterbox-tts` version, a `force` rebuild flag, and a lemonade ref
for the wrapper).
