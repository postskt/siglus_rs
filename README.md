<div align="center">

# Siglus 引擎 Switch 移植版 · SiglusEngine Switch Port

**本仓库是 [xmoezzz/siglus_rs](https://github.com/xmoezzz/siglus_rs)（MPL-2.0）的 fork，面向 Nintendo Switch（Atmosphere CFW / aarch64）维护打包流水线与真机修复。**

[![Download latest](https://img.shields.io/badge/⬇️_下载最新版_v0.1.0-e60012?logo=nintendo3ds&logoColor=white)](https://github.com/postskt/siglus_rs/releases/latest)
[![Upstream](https://img.shields.io/badge/上游-xmoezzz%2Fsiglus_rs-6aa84f)](https://github.com/xmoezzz/siglus_rs)
[![License](https://img.shields.io/badge/License-MPL--2.0-blue)](LICENSE-MPL-2.0)
![Platform](https://img.shields.io/badge/Platform-Nintendo_Switch_(CFW)-e60012)

<img src="images/switch.png" width="58%" alt="SiglusEngine on Nintendo Switch" />

</div>

## 🕹️ 一键下载 | One-click Download

| 文件 | 用途 |
|---|---|
| [`siglus_switch.nro`](https://github.com/postskt/siglus_rs/releases/latest/download/siglus_switch.nro) | 放入 `sdmc:/switch/siglus_rs/`，homebrew 菜单直接运行 |
| [`siglus_switch.nso`](https://github.com/postskt/siglus_rs/releases/latest/download/siglus_switch.nso) | NSP 打包用 ExeFS `main`（配合 `main.npdm` 与 RomFS） |

> 链接永远指向最新 Release。**本项目不分发任何游戏资源**，请使用合法拥有的游戏数据。

## 🧭 工作流程 | Build & Packaging Pipeline

1. **引擎构建**：锁定 nightly + 自定义 target `aarch64-switch.json`（`env=newlib`）+ `-Zbuild-std=std,panic_abort` → `libsiglus_scene_vm.a`（crates.io 依赖零改动，含完整 std）
2. **着色器**：与桌面渲染共用同一份 WGSL → naga 译为 GLSL 4.60 → devkitPro `uam` 编成 DKSH → RomFS
3. **链接**：libnx C shell（`main.c` 生命周期/输入/audren，`gpu.c` deko3d）+ Rust staticlib → ELF
4. **打包**：`elf2nro` → NRO；或 `elf2nso` + `main.npdm` + `hacpack` → NSP
5. 本地一键构建：`构建Switch.bat`（Windows，经 msys2 bash）；CI 同命令

## 📦 Switch 使用方法 | Usage

1. **NRO 部署**：`siglus_switch.nro` → `sdmc:/switch/siglus_rs/`；游戏目录（`Scene.pck`、`Gameexe.dat`、资源包）→ `sdmc:/switch/siglus_rs/game/`
2. **NSP 部署**：单游戏打包后安装（DBI / Tinfoil / Awoo），从主菜单启动；需要 sigpatches
3. **加密游戏**：先在桌面端运行一次生成 `key.toml`，随游戏目录放置（Switch 上不进行密钥恢复）
4. **存档**：`sdmc:/switch/siglus_rs/game/savedata/`——内嵌 RomFS 的载荷自动重定向到这里（PR #108），NRO / NSP 两种部署存档互通
5. **按键**：A 确认 / B 取消 / X 空格 / Y Tab / L·R Shift·Ctrl / ± 菜单 / 十字键与摇杆方向；触屏直接点按

## 🔧 本 fork 相对上游的改动 | Fork Changes

- [#108](https://github.com/xmoezzz/siglus_rs/pull/108) — 只读 RomFS 载荷的存档重定向到 SD 卡；游戏根目录大小写回退探测
- [#109](https://github.com/xmoezzz/siglus_rs/pull/109) — NSO 构建目标（ExeFS 打包用）
- Windows 下经 msys2 bash 的本地一键构建脚本与 NSP 打包脚本（`platform/switch/package_siglus_nsp.py`）

## 📄 许可 | License

引擎代码遵循上游的 **MPL-2.0**，本 fork 的修改同样以 MPL-2.0 提供；游戏数据（脚本 / 音频 / 美术）版权归属原权利方，本仓库不包含、不分发。

---

# siglus_rs

<img src="./icon/Icon.png" alt="icon" align="left" width="100" style="margin-right: 10px;" />

**siglus_rs** is an unofficial full Rust reimplementation of **UK2, AVG32, RealLive, and SiglusEngine**, with a primary focus on SiglusEngine.

This project is non-commercial and intended for research purposes.

Architectural decisions are recorded in [docs/adr](docs/adr/README.md).
Accepted decisions describe the intended design, not necessarily implemented features.

Siglus applies an **unofficial Unicode cell-spacing extension** automatically.
It requires no configuration file or activation step on any host.

The extension uses Unicode widths rather than a language whitelist. It handles
accented alphabetic text, font-measured Greek and Cyrillic, wide CJK, East Asian
ambiguous punctuation such as Japanese ellipses and dashes, and canonical
composition of decomposed characters in display runs. Newly handled alphabetic
characters use their resolved font advances, including fallback fonts, so wide
glyphs are not forced into overlapping half cells. Messages, names, ruby,
text objects, choices, backlog, reconstructed choices, and editbox geometry
share the policy. Editbox movement and deletion use grapheme boundaries.
Script strings and native UTF-16/width commands retain their original semantics.
Existing positioned save glyphs retain their coordinates.

This is the cell-spacing stage of [ADR 0001](docs/adr/0001-unicode-text-layout.md),
not complete Unicode typography. Contextual shaping, bidi, Unicode word/line
breaking, complex mark attachment, joined emoji, cluster-aware font fallback,
and lossless save reconstruction for future shaped layouts are still pending.
Font availability still determines glyph coverage.

<br clear="left"/>

## Example screenshots

### UK2 Engine
**1993–1997 · AyPio**

<table>
  <tr>
    <td align="center" width="33%">
      <img src="./images/uk2_1.png" width="100%"><br>
      <b>Sorcer Kingdom</b><br>
      <sub>Title screen.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/uk2_2.jpeg" width="100%"><br>
      <b>Sorcer Kingdom</b><br>
      <sub>Dialogue scene.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/uk2_3.png" width="100%"><br>
      <b>Sorcer Kingdom</b><br>
      <sub>Map exploration.</sub>
    </td>
  </tr>
</table>


### AVG32
**1996–2001 · VisualArt's**

<table>
  <tr>
    <td align="center" width="33%">
      <img src="./images/avg32_air_title.png" width="100%"><br>
      <b>AIR</b><br>
      <sub>The title of AIR.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/avg32_air_op.png" width="100%"><br>
      <b>AIR</b><br>
      <sub>Air opening.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/avg32_kanon.png" width="100%"><br>
      <b>Kanon</b><br>
      <sub>Kanon opening.</sub>
    </td>
  </tr>
</table>


### RealLive
**2001–2022 · VisualArt's**

<table>
  <tr>
    <td align="center" width="33%">
      <img src="./images/rl_lb.jpeg" width="100%"><br>
      <b>Little Busters!</b><br>
      <sub>A screenshot of Little Busters!</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/rl_game.jpeg" width="100%"><br>
      <b>Little Busters!</b><br>
      <sub>The baseball minigame in Little Busters! (implemented in bt00.dll)</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/rl_tomoyo_dt.jpeg" width="100%"><br>
      <b>Tomoyo After: It's a Wonderful Life</b><br>
      <sub>The Dungeons & Takafumis minigame in Tomoyo After ~It's a Wonderful Life~ (dt00.dll)</sub>
    </td>
  </tr>
</table>


### SiglusEngine
**2010–now · VisualArt's**

<table>
  <tr>
    <td align="center" width="33%">
      <img src="./images/screenshot.png" width="100%"><br>
      <b>macOS</b><br>
      <sub>SiglusEngine running natively on macOS.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/ios-screenshot.png" width="100%"><br>
      <b>iOS</b><br>
      <sub>SiglusEngine running on iOS.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/screenshot-wasm.png" width="100%"><br>
      <b>WebAssembly</b><br>
      <sub>SiglusEngine running in a web browser.</sub>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td align="center" width="33%">
      <img src="./images/sg_benchmark.png" width="100%"><br>
      <b>Chihaya Rolling WE</b><br>
      <sub>Official SiglusEngine benchmark released by Key to measure Rewrite performance.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/sg_summer.png" width="100%"><br>
      <b>Summer Pockets REFLECTION BLUE</b><br>
      <sub>Key’s expanded version of Summer Pockets, adding new routes and a new heroine.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/sg_anemoi.png" width="100%"><br>
      <b>anemoi</b><br>
      <sub>Key’s 2026 romance adventure title.</sub>
    </td>
  </tr>
</table>

### On game consoles

<table>
  <tr>
    <td align="center" width="33%">
      <img src="./images/psvita.png" width="100%"><br>
      <b>PS Vita</b><br>
      <sub>Summer Pockets REFLECTION BLUE for PS Vita.</sub>
    </td>
    <td align="center" width="33%">
      <img src="./images/switch.png" width="100%"><br>
      <b>Nintendo Switch</b><br>
      <sub>Anemoi for Nintendo Switch.</sub>
    </td>
  </tr>
</table>

* siglus_rs works on a wide range of platforms, including Windows, Linux, macOS, iOS, Android, and WebAssembly.

| Platform | Targets |
|---|---|
| Linux | x86_64, aarch64 |
| FreeBSD | x86_64 |
| Windows | x86_64, ARM64 |
| macOS | aarch64 app, x86_64 app, universal DMG app bundle |
| iOS | arm64 device, arm64 simulator, x86_64 simulator |
| Android | arm64-v8a, x86_64 |
| WebAssembly | wasm32-unknown-unknown |
| PS Vita | armv7 |
| Nintendo Switch | aarch64 |

The app launchers (macOS bundle, iOS, Android, WebAssembly) import one game or
many games at once, detect the engine (SiglusEngine, RealLive, AVG32, UK2),
show cover art or the game icon, and let you pick the text encoding for
non-Siglus games. See [crates/game_launcher](crates/game_launcher/README.md).

## Pre-built binaries
* See preview releases on [GitHub Releases](https://github.com/xmoezzz/siglus_rs/releases)

Android APKs from non-fork pushes and all tag builds use a persistent signing
key. The build aligns the APK, signs it, and verifies its signature before
uploading the `siglus-android` artifact.
Repository secrets `ANDROID_KEY_BASE64` (base64-encoded keystore),
`ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`, and `ANDROID_KEYSTORE_PASSWORD` are
required. Missing signing configuration fails the Android job. Keep the same
key for future releases so installed apps can be updated.

Pull requests and branch pushes in forks only check compilation. They upload
`siglus-android-unsigned/app-release-unsigned.apk`, which is not directly
installable and is never published as a release or rolling pre-release. No
release signing secrets are required for these checks. Forks that publish
tagged releases must configure their own persistent signing key and secrets.

## Documentation Availability
* API documentation is available at [docs](https://xmoezzz.github.io/siglus_rs/)
* [PS Vita port](platform/vita/README.md) — experimental SiglusEngine player (`siglus-psvita.vpk` on the releases page); installing it and a game is described there. [Roadmap](platform/vita/ROADMAP.md).

## Run

```bash
cargo run --release -p siglus_scene_vm --bin siglus_engine -- --project-dir ~/Documents/siglus_rs-main/testcase
```

Closing the desktop game window prefers the game's own exit confirmation.
The engine looks for a unique, parameterless exit action in shared script
commands, the configured cancel-menu scene, and the active title menu's local
button actions. Title actions are available only after the title buttons are
ready. Scripts that require EXCALL menu storage, unsupported or ambiguous
scripts, and already active system menus use the built-in confirmation instead.
An optional `#CLOSE_SCENE = "scene_name", label` entry in `Gameexe.ini` overrides
discovery: that scene must handle confirmation and call `syscom.end_game` on
acceptance; returning resumes the game.

Desktop windows use `icon.png`, `icon.ico`, or the first readable `.ico` file
(in filename order) from the game directory, falling back to the Siglus icon.
The engine uses winit `0.31.0-beta.3` to send window icons directly through
`xdg_toplevel_icon_v1` on Wayland compositors that support it. No desktop entry
is generated. Each game retains a separate Wayland application ID.

## Community
If you want to join the development and discussion of this project, you can join the following Discord server:
* Discord: [https://discord.gg/g4rXucPZz3](https://discord.gg/g4rXucPZz3)
* Personally, I only able to speak English, Chinese, Japanese, and very limited French. 


## The `key.toml` configuration file
The `key.toml` file is used to specify different configuration options for siglus_rs. The file should be placed in the root directory of the game project.

### Resource decryption key
* This configuration key for this element is `key`. It is an array of 16 bytes (128 bits) that represents the secondary key.

* SiglusEngine games require a secondary key to decrypt protected resources.

siglus_rs can automatically brute-force the secondary key, but if you want specify the key manually, you can create a `key.toml` file in the game root directory.

There are several practical ways to obtain the key:

1. Static extraction, when the game executable is not encrypted or obfuscated (recommended). The general idea can be found in this repository:

   https://github.com/xmoezzz/siglus_static_key_tool

2. Dynamic extraction. The general idea can be found in this older repository:

   https://github.com/xmoezzz/SiglusExtract

3. Known-key databases maintained by some extractor tools.

Brute-force will be attempted in the following situations:
1. If the key is not specified in the `key.toml` file, siglus_rs will try to brute-force the key.
2. Users specify a wrong key in the `key.toml` file. siglus_rs will try to override the key.
3. If siglus_rs fails to save or overwrite the `key.toml` file, the engine will still execute.

Trial games may not require a secondary key, and in that case, you can specify all-zero key in the `key.toml` file. Here is an example of `key.toml`:

```toml
key = [
  0x00, 0x00, 0x00, 0x00,
  0x00, 0x00, 0x00, 0x00,
  0x00, 0x00, 0x00, 0x00,
  0x00, 0x00, 0x00, 0x00,
]
```

### String Encryption
* This configuration key for this element is `override_string_encryption`. The value of this key can be `xor`, `none`, or `mdl`. The default value is `xor`. 
* In very earlier versions of SiglusEngine, string encryption was not used. 
* However, in later versions (for the most cases), string encryption is enabled by default. 

Explanation of each value:
* `xor`: Enbales the string encryption. This is the default value even if the `override_string_encryption` key is not specified in the `key.toml` file.
* `none`: Disables the string encryption. If you are sure that the game does not use string encryption.
* `mdl`: Automatically detects the string encryption method by using the MDL approach (also see the paper: [https://arxiv.org/abs/cs/0312044](https://arxiv.org/abs/cs/0312044)). It does introduce a performance overhead, but IMO, it's minor.

## Claims
<p align="center">
  <img src="icon/crysirror.svg" width="160" height="160" alt="CrySirror logo">
</p>

This project belongs to `CrySirror` [project](https://github.com/xmoezzz/CrySirror), which is a unified engine backend for visual novels and 3D games. 

1. Bug fixes and improvements for the siglus engine is acceptable in this repository. 

2. If you want to support a new platform (e.g. game console), please create an issue or pull request in the `CrySirror` repository instead of this one. 


## License
This project is licensed under the MPL-2.0 License. See [LICENSE](./LICENSE-MPL-2.0) for details.
