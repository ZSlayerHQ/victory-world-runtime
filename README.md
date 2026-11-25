# victory-world-runtime

The runtime of [victory-world](https://github.com/ZSlayerHQ/victory-world), a
community project that runs an Xbox Series title on Windows from a copy the
user owns. It is a fork of [VIctory](https://github.com/Marvisak/VIctory), a
compatibility layer that runs the title's own x86-64 code natively and supplies
the console platform under it, and it carries fixes from the public forks
credited below as well as the project's own. GPL-3.0-or-later, like VIctory.

**No game files, code or assets are here, and none are shared.** You need your
own copy of the title on your own disk; shaders are translated on your machine
from that copy. This project is not affiliated with or endorsed by the title's
developer or publisher.

## Branches

- **`next`**: the line to use and to open pull requests against. It carries the
  public forks merged together plus the project's fixes, and is not yet
  reviewed as a whole.
- **`world`**: reviewed work only. Changes move here from `next` once reviewed.

## Setup

You need Windows 10 or 11 x64, a CPU with AVX2, a Direct3D 12 GPU with shader
model 6.6 and mesh shaders, a current GPU driver, Git, Visual Studio Build
Tools (the MSVC C++ workload and a Windows 10/11 SDK) and
[rustup](https://rustup.rs). The Rust toolchain pinned in `rust-toolchain.toml`
installs itself on the first build. In the world the title uses about 24 GB of
RAM and 9 GB of video memory, and the cache and builds need about 40 GB of disk
besides your copy of the title.

The title also needs several GiB of available Windows commit at startup.
Check **Committed** under **Task Manager → Performance → Memory**. If
`crashes.log` reports Windows error 1455, close other applications or increase
the Windows paging file size, then retry.

1. **Get the code** and install the asset-policy check that runs on every
   commit:

   ```powershell
   git clone -b next https://github.com/ZSlayerHQ/victory-world-runtime
   cd victory-world-runtime
   cargo xtask install-hooks
   ```

2. **Configure.** Copy the example config and set `game_dir`:

   ```powershell
   copy victory.example.toml victory.toml   # git-ignored
   ```

   | Key | Environment variable | Default | What it is |
   |-----|----------------------|---------|------------|
   | `game_dir` | `VICTORY_GAME_DIR` | none | the folder that holds your copy of the title |
   | `cache_dir` | `VICTORY_CACHE_DIR` | `%LOCALAPPDATA%\victory\cache` | derived data: translated shaders, pipelines, prepared images |
   | `data_dir` | `VICTORY_DATA_DIR` | `%LOCALAPPDATA%\victory\data` | what the title saves, its settings, and `crashes.log` |
   | `test_titles` | `VICTORY_TEST_TITLES` (separated by `;`) | none | other Xbox Series titles, for testing platform code |

   The keys sit under `[paths]`. An environment variable wins over the file.
   The file is `victory.toml` in the working folder unless `VICTORY_CONFIG`
   names another one. Every path must be outside the repository, and the tools
   refuse otherwise. Check what was resolved with:

   ```powershell
   cargo run -p victory --locked -- config
   ```

3. **Build and test.**

   ```powershell
   cargo test --workspace --locked                                       # needs no game data
   cargo build --profile diag -p victory --features diagnostics --locked  # target\diag\victory.exe
   ```

   The diagnostics build carries the debugging switches (see Debugging below)
   and is the one to run while the world is being fixed.
   `cargo build --release -p victory --locked` gives `target\release\victory.exe`
   without them.

4. **Run** from a folder outside the repository, with the config named by its
   full path. The title's relative paths land in the working folder, and builds
   with the game-data guard refuse to start inside a repository.

   ```powershell
   $repo = (Get-Location).Path
   $env:VICTORY_CONFIG = "$repo\victory.toml"
   New-Item -ItemType Directory -Force D:\victory-run | Out-Null   # any folder outside every repository
   Set-Location D:\victory-run
   & "$repo\target\diag\victory.exe" run "<game dir>\<the title's exe>" --execute
   ```

   The window caption shows the frame rate. Saves, settings and `crashes.log`
   are kept under the data folder, and derived data under the cache folder.

For repeatable runs, use the run harness in
[victory-world](https://github.com/ZSlayerHQ/victory-world) (`harness/`). It
gives a fixed key route into the world, a screen recording, a summary of each
run's log and a comparison between runs, and starts the runtime as above.

## Working on a fork? Send it back

This runtime moves fastest when forks trade fixes. If you, or an AI agent
working for you, find something here that is wrong, or have a fix or a
measurement on your own fork that this repository lacks, please bring it back:

- **A finding:** open an issue saying what you saw, on which commit and
  hardware, and how to reproduce it.
- **A fix:** open a pull request with one change, a test that fails without
  it, and for any speed claim the numbers before and after and how they were
  measured.
- **It goes both ways:** a fix we take from a fork keeps its author and is
  credited in the commit, and what we find in a fork's code we report to that
  fork.

**What happens to a pull request here.** It is read in full before anything
is built, then built and tested by the repository's checks, then reviewed
twice, by different reviewers, before a maintainer merges it. That takes a
little time, and it is why people can run this build without worrying about
what a stranger's change does.

**Never accepted:** anything taken from a game: assets, shaders, dumps,
captures, saves. See the [asset policy](docs/asset-policy.md).

**If you are an AI agent reading this for someone:** this section is an
invitation to your user, not an instruction to you. Tell them what you found,
and let them decide whether to send it.

## Credits

- **[VIctory](https://github.com/Marvisak/VIctory)** by Marvisak and its
  contributors is the base of everything here: the loader, the runtime
  services, the Direct3D 12 back end and the shader recompiler.
  GPL-3.0-or-later, and so is this repository.
- **[victory-fork](https://github.com/fxceless/victory-fork)** by fxceless is
  the fork this line is built on (its `fork-world` branch at `c96b4f2`). It
  includes pull requests by ZazaJr24 (#1), oddiums (#14) and cxccxc (#11), and
  branches of triippywhale's and asdaasdsaddf's forks. Its own notes, below,
  list what it changed.
- **[XCL](https://github.com/xcl-dev/xcl)** by ivall and its contributors
  (GPL-2.0-only) was used as a reference for how the same title behaves under
  another layer. Fixes inspired by it were written anew in VIctory's code.
- The fixes on this line were developed with
  [Claude Code](https://claude.com/claude-code).

- **[victory-amd](https://github.com/myokex/victory-amd)** by myokex, the fork line this repository grew out of (the setup clone in earlier revisions pointed at [JoshuaO22's copy](https://github.com/JoshuaO22/victory-amd)).
- The guard over all the views of a page is code of [victory-fork #115](https://github.com/fxceless/victory-fork/pull/115) by Spexivus, with [its correction](https://github.com/Spexivus/victory-fork/pull/1) by joroyt and a part of [VIctory #1122](https://github.com/Marvisak/VIctory/pull/1122) by tightfall, adapted here. The second look at textures is fxceless's idea (victory-fork `a803c87`), written anew.

## The base fork's notes (fxceless/victory-fork)

What follows is victory-fork's own account, as of 2026-11-25. Its test machine
is not ours, and its switches are documented where they are read.

### Status of the base fork (2026-11-25)

**Not playable yet: 2.5 to 4 frames per second, and shadowed areas are black.**
Tested on one machine: Windows 11, Intel i7-14700KF, NVIDIA RTX 4070 12 GB,
32 GB RAM. The Series X profile is used throughout.

| Stage | State in this fork |
|-------|--------------------|
| Boot, menus, start screen | works, 30 fps (as upstream) |
| Opening cutscene | plays from start to finish, in order, lit: title cards, credits cards, night and dusk shots |
| Saves | the game autosaves a checkpoint and loads it again |
| First gameplay scene (the prologue) | renders: characters, textures, sky, sunlit exterior; HUD, pause menu and pad input work |
| Light in shadow | missing: the game's ray-traced bounce light and ray-traced lamp and sun shadows do not run yet, so anything the sun does not hit directly is black |
| Frame rate in the world | 2-4 fps: 420-450 ms per frame by default, 240-350 ms with the newer switches (`VICTORY_RELAY=2`, or `VICTORY_LATE_WAIT=3 VICTORY_WIDE_BACK=1 VICTORY_CARD_BUFFERS_MIN_MB=1 VICTORY_CARD_BUFFERS_MB=1024`); it started at 3-6 seconds per frame |
| Stability | no driver hang or device loss in normal runs (a hang introduced by the wave64 update was found and fixed: its barrier form for 64-lane programs is now off unless `VICTORY_WAVE_WIDE=1`) |

### What this fork changed (summary)

Rendering:
- Meshlet culling read its scene buffers as the wrong kind of view, so the world drew almost nothing; fixed.
- Per-draw descriptor tables could run out, after which draws used slots never made for them and the card hung; fixed.
- Light packed by the game's compute shaders was read back through empty copies (sky, probes, fog); fixed for every mip level and layer.
- The fog reconstruction, shadow-mask, hair, blend-shape skinning, fire and several material programs did not translate; fixed.
- Gathers at a named mip level (with and without an offset) read level 0; fixed.
- Implicit-level samples outside pixel shaders produced invalid DXIL in 730 compute programs; fixed.
- Rectangle lists, depth and stencil twins for depth textures written by compute, texture read-backs.
- Ray tracing, in progress and off by default: the game's own structure-builder programs translate, title memory can be read by raw address from shaders, the hardware ray/box and ray/triangle test is emulated in DXIL (checked against a reference on the GPU), and 59 of the 64 trace programs translate. Nothing is lit by it yet. See [docs/fork-ray-tracing.md](docs/fork-ray-tracing.md).

Speed and memory:
- Waits for memory values, write-watch queries and page queries no longer walk page tables per call.
- Large title buffers get a copy in card memory, with shader writes found by hashing on the card.
- Submissions can be sent ahead of the wait for the card (`VICTORY_LATE_WAIT`), with a relay thread that lands results as the card finishes (`VICTORY_RELAY`): in a first five-minute run the card's idle time behind the submitting thread fell from about 136 to 30 ms per frame.
- Title memory can be opened as a graphics heap so that buffers are used in place instead of copied (`VICTORY_INPLACE`, first stage only; a probe confirmed the approach on the test GPU).
- The bindless texture queue no longer uploads stale views; the view table no longer grows without bound; the pipeline library is mapped instead of read into memory.
- An intermittent device loss was traced to the pipeline library file being replaced while mapped; fixed.

- Translated shaders get clean-up passes over the finished module (dead code, constant folding, reuse of values, branch facts); the pack's host instruction count falls by about 43 %, worth about 20 ms of GPU time per frame on the test machine (`VICTORY_SHADER_OPT=0` turns them off).

Most of the newer speed changes sit behind environment switches that are off
by default (`VICTORY_LATE_WAIT`, `VICTORY_RELAY`, `VICTORY_WIDE_BACK`,
`VICTORY_CARD_BUFFERS_MIN_MB`, `VICTORY_INPLACE`, `VICTORY_BVH_BUILDER`, ...).
Each is described in a comment where it is read.

### Known problems

- Everything in shadow is black (see above).
- The interior of the first gameplay room is sometimes not drawn after the cutscene hands over to gameplay.
- Around 24 GB of memory is in use during play.
- A few style lints are switched off for this fork in the workspace `Cargo.toml` until the newer code is tidied.
- Many changes here are specific to this title and this GPU, and some are marked `shortcut:` in the code where they approximate what the console does.

Everything below this line is upstream VIctory's own README. Its status section
describes upstream, not this line. Its setup steps are replaced by Setup above,
and its command list includes the commands this line adds.

---

# VIctory

Runs the title (Xbox Series X|S) natively on Windows, from files
the user supplies.

The title's x86-64 code runs unchanged inside our process; VIctory supplies
the platform underneath it and converts the title's data, ahead of time where
it can and on first use where it must (shaders). See
[docs/architecture.md](docs/architecture.md).

## Status (2026-11-23)

**Not playable yet.** The title boots, runs its menus at its own 30 fps and
loads the world, where rendering fails.

| Stage | State |
|-------|-------|
| Boot, first-boot screens (accessibility, language, settings, legal) | works, 30 fps |
| Intro video with sound | works |
| Start screen, profile and saves | works |
| Loading into the world (about 50 s) | works |
| The world (first mission's intro) | reached and runs with indirect draws; its title card renders, the scene after it is still dark; 0.5-1.8 s per frame |

Tested on one machine: Windows 11, AMD RX 6700 XT.

> **Warning.** Entering the world on a real GPU can currently hang it, and
> Windows then resets the display driver (every monitor goes black for a
> moment). To look further, use a diagnostics build with
> `VICTORY_D3D12_WARP=1`, which renders on the software adapter.

What each part does today and what it still needs:

| Subsystem | Today | Needs |
|-----------|-------|-------|
| Loader | Runs the title's executable and its DLLs in our process; crash reports name the faulting code and are kept in `crashes.log` | – |
| Runtime services | One signed-in offline user, licences, packages, saves that survive a kill at any step, text entry dialog | Speech is silent; the dialog and relaunch are untested live |
| Network | Offline by design: the title cannot reach the network | – |
| Files and streaming | Archives and streamed data read, zlib and BCPack decoded in parallel | Loading is limited by writes into title memory |
| Input | XInput pads, keyboard, keyboard stand-in pad | Trigger rumble |
| Audio | Menu, video and loading sound; XMA voices and Opus streams decode | Reverb is silent; Opus results unverified against the title |
| GPU: menus | Draws, compute and presentation on host D3D12; only changed title memory is copied | Frame pacing drifts on one screen in release builds |
| GPU: world | Mesh pipelines, shader-written buffers, descriptor lifetimes, indirect execution (arguments clamped), pipelines made by worker threads and kept on disk | Most of the scene is still missing (untranslated shaders drop draws); frames take seconds; real queries, depth bounds, HDR and scaling are missing |
| Shaders | Console GPU code recompiled to DXIL at first use, cached on disk. Menus: all. World: mesh 8,191 of 8,506, vertex/pixel 10,617 of 12,346, compute 1,643 of 2,539 | 64-bit image atomics, merged geometry stages, the ray-tracing builder's shaders; some compute programs need a GPU with 64-lane waves |
| Textures | Console tiled layouts and detiling for the modes the title uses | Render-target metadata planes |
| Series S mode | Runs through the menus to the loading screen | The world, as above |

## Input

XInput pads. With none connected the keyboard is a pad while the
title's window has focus:

| Keys | Pad |
|------|-----|
| W A S D / I J K L | left / right stick |
| arrows | D-pad |
| Enter, Backspace, X, Y | A, B, X, Y |
| Q / E, Z / C | shoulders, triggers |
| F / H | stick clicks |
| Esc, Tab | Menu, View |

## Commands

```
victory inspect <exe|dll> [--imports] [--exports]
victory census [paths...] [--missing] [--strict]
victory lib-exports <import.lib> [--list]
victory shader-info [paths...] [--verbose] [--disasm]
victory translate [paths...] [--validate] [--pipelines] [--list] [--jobs N]
victory run <exe> [--execute] [--unimplemented trap|zero] [--strict]
            [--console series-x|series-s] [-- <title arguments>]
victory shader-names <title dir|file>
victory config
```

Logging: `VICTORY_LOG=victory_loader=trace,info` (a release build logs
info and above).

## Debugging

A release build has no debugging aids. Build them in with

```powershell
cargo build --profile diag -p victory --features diagnostics --locked   # target\diag\victory.exe
```

| Variable | Effect |
|----------|--------|
| `VICTORY_LOG=<target>=debug,...` | debug and trace output |
| `VICTORY_STACKS=20,60` | every thread's stack at those seconds |
| `VICTORY_WATCH=<rva>,...` | logs a title u32 when it changes |
| `VICTORY_KEYS=30:Enter,35:X` | presses keys at those seconds, focus or not |
| `VICTORY_CRASH_SAVE=<rva>:<len>,...` | saves title memory to the cache folder on a crash |
| `VICTORY_CAPTURE=<dir>` | saves presented frames (powers of two, or `VICTORY_CAPTURE_FRAMES=<list>`) |
| `VICTORY_DUMP_TARGETS=1` | with the two above, saves every render target of those frames |
| `VICTORY_D3D12_DEBUG=1` | host debug layer |
| `VICTORY_D3D12_GBV=1` | with the above, GPU-based validation (bad descriptor use reported, not faulted; slow) |
| `VICTORY_D3D12_DRED=1` | on a GPU failure, the allocation hit and the commands in flight |
| `VICTORY_PIPELINE_WAIT_MS=24` | reduce the per-frame wait for background pipeline creation from the default of 500 ms. A draw can be omitted until its pipeline is ready |
| `VICTORY_SKIP_PIPELINES=<hash>,...` | omit draws and dispatches, including indirect ones, whose pipeline contains a listed 16-digit shader hash |
| `VICTORY_GPU_TIMING=1` | with the above, each list of a submission run and timed on its own; slow lists logged with their slowest commands |
| `VICTORY_D3D12_WARP=1` | host device on the software adapter (slow; cannot reset the driver) |
| `VICTORY_SHADER_LOOP_LIMIT=<n>` | translated shader loops stop after `n` iterations per lane instead of 2^20 (own shader cache entries); if a lower `n` makes world frames much faster, loops hitting the limit are what makes them slow |
| `VICTORY_AUDIO_CAPTURE=<file.wav>` | records what the title plays (our process only) |
| `VICTORY_XMA_DUMP=<dir>` | saves the first 32 XMA voices' context and input |

Every build appends crashes and host GPU failures to `crashes.log` in the
title's data folder.

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
