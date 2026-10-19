# ps4_core — Shared Core for Native PS4 PC Ports

> **Local-only repository.** A shared compatibility layer and runtime core that allows PlayStation 4 games to run natively on PC without CPU emulation. Powers native ports including:
>
> - [gravityrush_pc](https://github.com/Prathewsh/gravityrush_pc) (_Gravity Rush 2_)

Based on a fork of [HayatoG/ps4_core](https://github.com/HayatoG/ps4_core).

**No Sony proprietary files are included** (no `eboot.bin`, `prx`, `sprx`, `psarc`, `sfo`, or official artwork).

---

## What It Is

Native ports run the game's original executable directly on the PC's x86-64 CPU at full native speed without CPU emulation. The core provides:

1. **Loader & Dynamic Linker (`port/src/probe.c`, `port/scripts/link_*.py`)**
   - Maps decrypted PS4 ELF/PRX modules (`eboot.bin`, `libc.prx`, `libSceFios2.prx`, etc.).
   - Applies relocations and patches host/platform differences (e.g. segment register handling).
   - Resolves `NID#library` imports to host functions.

2. **PS4 System Library Reimplementations (HLE) (`port/src/runtime_*.c`)**
   - Clean C implementations of PS4 OS APIs:
     - **libkernel**: Threads, synchronization primitives, memory management, files, timers.
     - **Input & AV**: Controller input (`Pad`), audio output (`AudioOut`), media decoding (`Ajm`, `Videodec`, `AvPlayer`).
     - **OS Services**: `SaveData`, `UserService`, `SystemService`, `PlayGo`, `Rtc`, `Sysmodule`.
     - **Network/Trophies**: Offline trophy system (`NpManager`), network/HTTP stubs.
   - Full function-level implementation status: [docs/LIBRARIES.md](docs/LIBRARIES.md).

3. **GPU Layer (`port/gpu/`)**
   - A specialized fork of **shadPS4** translating PS4 GCN command buffers (`libSceGnmDriver`) and shaders to Vulkan.
   - SDL3-based window management, in-game overlay menu, upscaling (FSR, CAS), and post-processing (SMAA).

---

## Directory Layout

```
ps4_core/
├── docs/            # Documentation: LIBRARIES.md, KNOWN_ISSUES.md, CHANGELOG.md
├── port/            # Shared port codebase (mirrors port repositories)
│   ├── src/         # Loader (probe.c), runtime resolver, and HLE libraries
│   ├── gpu/         # Vulkan GPU translation layer (shadPS4 fork) and SDL3 shim
│   ├── scripts/     # Linkers, module preparation, and patch utilities
│   └── third_party/ # Submodules (LibAtrac9, Vulkan headers, ImGui, etc.)
├── scripts/         # General helper tools (import extractors, TOC tools)
├── tools/           # Core synchronization (core_sync.py) and export list generator
├── CORE_MANIFEST    # Canonical file list tracking core vs game-specific files
└── CORE_SOURCE      # Tracks upstream commit hashes for each imported component
```

---

## Core vs. Game-Specific Code

| Shared Core (`ps4_core`)                        | Game-Specific (Kept in Game Ports)                      |
| ----------------------------------------------- | ------------------------------------------------------- |
| Loader (`probe.c`) & runtime linker             | Game resolvers (`runtime_tlg.c`, `runtime_gr2.c`, etc.) |
| GPU translation layer (Vulkan) & SDL3 shim      | Game-specific patches, FPS uncaps, memory hacks         |
| Universal NID lookup table (`import_names.inc`) | Packaging configs (`port.env`), launcher scripts, icons |
| Common system libraries (`runtime_*.c`)         | Game profiles (`profile.ini`) and user settings         |

---

## Building & Dependencies

- **Platform Requirements:** Linux (x86-64) or Windows (MSYS2 CLANG64 environment).
- **Key Dependencies:** Vulkan SDK, SDL3, FFmpeg (`libavcodec`, `libavutil`, `libswscale`), FreeType, Zydis.
- **Submodules:** Initialize before building:
  ```bash
  git submodule update --init --recursive
  ```
- **Shader Cache:** Stored under `<saves>/cache/<TITLE_ID>-<hash>` per game.

---

## Management Tools

- **`tools/core_sync.py`**: Syncs changes between `ps4_core` and individual port checkouts:
  ```bash
  python3 tools/core_sync.py --status-all                 # Check status against all ports
  python3 tools/core_sync.py --diff --ref origin/main <path_to_port> # View file diffs
  python3 tools/core_sync.py --pull --ref origin/main <path_to_port> # Pull latest port updates
  ```
- **`tools/list_exports.py`**: Regenerates [docs/LIBRARIES.md](docs/LIBRARIES.md) from the runtime tables:
  ```bash
  python3 tools/list_exports.py -o docs/LIBRARIES.md
  ```
