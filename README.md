<p align="center">
  <img src="docs/images/logo.svg" width="72" height="72" alt="Faster Loadin and Savin mark">
</p>

<h1 align="center">Faster Loadin' 'n' Savin'</h1>

<p align="center"><strong>SKSE cosaves, faster — and not truncated mid-crash.</strong></p>

<p align="center">
  Maintenance fork of <a href="https://github.com/just-harry/save-load-accelerator-for-skse-cosaves">S.L.A.C.K.</a>
  by Harry Gillanders (<code>just-harry</code>).<br>
  Accelerates SKSE cosave serialize/load. Does not touch Skyrim's <code>.ess</code> format.
</p>

<p align="center">
  <a href="https://github.com/ShugokiFable/Faster-Loadin-n-Savin/actions/workflows/ci.yml"><img src="https://github.com/ShugokiFable/Faster-Loadin-n-Savin/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-0BSD-5eead4?labelColor=0d0f11" alt="0BSD License"></a>
  <a href="https://github.com/ShugokiFable/Faster-Loadin-n-Savin/releases/tag/v1.7.0"><img src="https://img.shields.io/badge/release-v1.7.0-5eead4?labelColor=0d0f11" alt="v1.7.0"></a>
  <img src="https://img.shields.io/badge/D-LDC%20betterC-8f9aa6?labelColor=0d0f11" alt="D LDC betterC">
</p>

<p align="center">
  <a href="#install">Install</a>
  ·
  <a href="#build">Build</a>
  ·
  <a href="#honest-status">Honest status</a>
  ·
  <a href="changelog.md">Changelog</a>
  ·
  <a href="VERIFYING-RELEASES.md">Verifying releases</a>
</p>

## Why it exists

SKSE plugins serialize into a **cosave** beside the `.ess`. Upstream S.L.A.C.K. already made that path fast. This fork keeps that work and stops the previous cosave from being truncated before the replacement has been fully written.

The public name is **Faster Loadin' 'n' Savin'** (requested by the original author). Runtime filenames stay `Save&LoadAcceleratorForSKSECosaves.dll` and `Save&LoadAcceleratorForSKSECosaves.ini` because the compiled plugin expects those names.

## What the fork adds

On top of upstream S.L.A.C.K.:

- Sibling `.tmp` writes, then `MoveFileExA(..., MOVEFILE_REPLACE_EXISTING | MOVEFILE_WRITE_THROUGH)` replacement
- Buffered, sequential, exact-length cosave loading
- Corrected committed-buffer sizing for padded writes
- Universal **x86-64-v2** FOMOD default, optional **x86-64-v3** High-End build
- Reproducible packaging with hashes and manifests

**1.6.0** rebased on upstream 1.3.3: SKSE 2.2.8 required for Skyrim 1.6.1170; SKSE 2.3.0 for AE 1.7.99; `SKSEPlugin_Preload` / `SKSEPlugin_Version` on those builds so SSE Engine Fixes' SKSE64 Preloader is no longer required there; inherited INI use-after-free fix.

**1.7.0** rebased on upstream 1.3.4: `SKSEPlugin_Preload` moved to ordinal 1 so SKSE 2.2.6 can load far enough to show a friendly mismatch dialog (with download link) instead of failing silently. Still requires **SKSE 2.2.8+** to *run* on AE 1.6.1170. Shipped DLLs self-report their true version.

Atomic writes and buffered exact-length reads remain unique to this fork. Upstream 1.3.3 still truncates cosaves in place.

## Safety scope

Atomic replacement means the previous cosave is not truncated before the new file has been fully written and closed. That **reduces the risk of a destroyed or truncated SKSE cosave when saving is interrupted**.

It is not a transaction spanning both the `.ess` and the SKSE cosave. It cannot guarantee that every third-party SKSE serialization callback produced logically complete data. Experimental parallel saving stays **disabled by default** — third-party callbacks cannot be assumed thread-safe.

No save cleaning or conversion is required when installing, updating, or uninstalling.

## Install

Install the archive with Mod Organizer 2, Vortex, or another FOMOD-capable manager. Current release: [v1.7.0](https://github.com/ShugokiFable/Faster-Loadin-n-Savin/releases/tag/v1.7.0).

1. Select the detected Skyrim runtime.
2. Keep **Universal (Recommended)** unless you deliberately want High-End.
3. Confirm the DLL is at `Data\DLLPlugins\Save&LoadAcceleratorForSKSECosaves.dll`.

High-End x86-64-v3 needs an AVX2-class CPU (AMD Zen 2+ or Intel Haswell+). It may provide little or no measurable improvement.

Manual install: copy the DLL and INI for **exactly one** game version to `Data\DLLPlugins\`. Universal files are in the archive's game-version folders. High-End files are under `Optional\High-End (x86-64-v3)\<game-version>\DLLPlugins\`.

Remove the old version before switching CPU builds so two copies cannot deploy. See [`INSTALL-NOTES.txt`](INSTALL-NOTES.txt).

## Compatibility

Standalone apart from SKSE. It does **not** require NextGen Disk Cache.

- 1.6.1170 and 1.7.99 builds use SKSE's builtin preloader (`SKSEPlugin_Preload`).
- Other variants still load from `Data\DLLPlugins\` and need a compatible DLL preloader.
- Parallel saving should stay disabled unless every participating SKSE plugin is known to be thread-safe during serialization.

Packaged game variants: `ae7_99`, `ae1170`, `ae1130`, `ae640`, `ae353`, `se`, `vr`, `gog`, `gog659`.

## Verifying releases

`tools/verify-release-binaries.py` produces DLL manifests, inspects static imports, and compares PE sections:

```text
python tools/verify-release-binaries.py manifest <extracted-release>
python tools/verify-release-binaries.py imports <installed-dll>
python tools/verify-release-binaries.py compare <old-dll> <new-dll> --strict
```

Static imports are evidence, not proof of every capability a native binary could exercise. Historical 1.5.1 `.text` identity vs 1.5.0 is documented in [`VERIFYING-RELEASES.md`](VERIFYING-RELEASES.md); that file has not been restated for 1.6.0 / 1.7.0.

## Build

Requirements:

- Windows PowerShell 5.1 or PowerShell 7+
- LDC (`ldc2`)
- Windows SDK resource compiler (`rc.exe`)
- Clang C++ or MSVC `cl.exe`
- `lld-link` or MSVC `link.exe`
- Python 3.8+ for release verification

```powershell
./scripts/update-version-number.ps1 1.7.0.0
./package-release.ps1 -Version 1.7.0
```

`package-release.ps1` defaults to 1.7.0 and builds both CPU profiles for all nine game variants unless `-SkipBuild` is passed. CI runs:

```powershell
./build.ps1 -BuildTag x86-64-v2 -TargetCPU x86-64-v2
./build.ps1 -BuildTag x86-64-v3 -TargetCPU x86-64-v3
./package-release.ps1
```

`package.ps1` is a compatibility wrapper around `package-release.ps1`. It no longer creates PDB-bearing legacy packages.

## Project map

```text
mod/slack_mod/     plugin (save_load, setup, entrypoint, INI)
mod/skse64/        SKSE bindings + runtime patch offsets
mod/game/          Skyrim SE/AE/VR/GOG archetype detection
mod/slack_common/  betterC support library (no D runtime, no CRT)
mod/*.ini          default plugin INI
scripts/           version stamp
tools/             release binary verification
build.ps1          LDC + rc + link
package-release.ps1  both CPU profiles, FOMOD, hashes
```

## Honest status

Verified in this tree:

- `VERSION.txt` **1.7.0.0**, changelog **1.7.0**, packager default **1.7.0**
- Atomic `.tmp` + `MoveFileExA` replacement and buffered exact-length reads in source
- GitHub Actions CI that builds x86-64-v2 and x86-64-v3

Not claimed:

- Acceleration of the main `.ess` save
- An atomic `.ess` + cosave transaction
- In-game load-time benchmarks from this README pass
- That High-End x86-64-v3 is faster on any given load order

## Credits

**Save and Load Accelerator for SKSE Cosaves** was created by **Harry Gillanders (`just-harry`)**.

- Upstream: https://github.com/just-harry/save-load-accelerator-for-skse-cosaves
- This fork: https://github.com/ShugokiFable/Faster-Loadin-n-Savin

See [`CREDITS.md`](CREDITS.md) and [`LICENSE`](LICENSE). 0BSD permits use, copying, modification, and redistribution with or without fee. Nexus permissions should mirror those terms.

## License

[0BSD](LICENSE)

---

## Hitchhiker's Guide to the Codebase

This software has been written using D in the style of a better C—but the abominable C runtime is not used—and the D runtime also is not used, and the D standard-library is mostly avoided.  
Thus, a great deal of this repository is my own non-standard-library, found in `mod/slack_common`.

### Map

- [`mod/slack_common`](mod/slack_common): This module deals with functionality that's more-or-less generic.
	- [`mod/slack_common/algorithms.d`](mod/slack_common/algorithms.d): Vaguely algorithmic logic.
	- [`mod/slack_common/bindings.d`](mod/slack_common/bindings.d): Bindings for the compilation target: in this case, Windows.
	- [`mod/slack_common/byte_sizes.d`](mod/slack_common/byte_sizes.d): Because `64.MB` is clearer than `64 << 20`.
	- [`mod/slack_common/cpp.d`](mod/slack_common/cpp.d): Some bindings for Visual C++'s STL.
	- [`mod/slack_common/dynamic_linking.d`](mod/slack_common/dynamic_linking.d): Some convenience functions for dynamically linking with exports.
	- [`mod/slack_common/dynamically_linked.d`](mod/slack_common/dynamically_linked.d): A bit of a bodge to make dynamically-linked symbols available to `slack_common` without presupposing a storage method/layout for the pointers.
	- [`mod/slack_common/file_handling.d`](mod/slack_common/file_handling.d): Does what it says on the tin.
	- [`mod/slack_common/ini.d`](mod/slack_common/ini.d): Simple INI file parsing (lexing, really).
	- [`mod/slack_common/integers.d`](mod/slack_common/integers.d): Functions and whatnot for dealing with integer values.
	- [`mod/slack_common/large_low_overhead_buffer.d`](mod/slack_common/large_low_overhead_buffer.d): Buffers optimised for low-overhead writing into large and re-used regions of memory.
	- [`mod/slack_common/parsing.d`](mod/slack_common/parsing.d): Straightforward parsing and lexing routines.
	- [`mod/slack_common/patching.d`](mod/slack_common/patching.d): Utilities for examining, generating, and rewriting x86 machine code.
	- [`mod/slack_common/pe.d`](mod/slack_common/pe.d): Functionality for inspecting Portable Executable images.
	- [`mod/slack_common/peb_access.d`](mod/slack_common/peb_access.d): Support for peeking inside the process's Process Environment Block.
	- [`mod/slack_common/simd.d`](mod/slack_common/simd.d): The basics for x86 SIMD usage.
	- [`mod/slack_common/sorting.d`](mod/slack_common/sorting.d): Functions which can sort any array so long as its length is three.
	- [`mod/slack_common/text.d`](mod/slack_common/text.d): Simple text manipulation routines, including UTF-8<->UTF-16 conversion.
	- [`mod/slack_common/threading.d`](mod/slack_common/threading.d): Functions for controlling and synchronising threads.
	- [`mod/slack_common/tib_access.d`](mod/slack_common/tib_access.d): Support for reading and writing from and to a thread's Thread Information Block.
	- [`mod/slack_common/timing.d`](mod/slack_common/timing.d): Time matters.
	- [`mod/slack_common/user_interface.d`](mod/slack_common/user_interface.d): Frankly, this is just for popping up an error-message box when something invariably goes pear-shaped.
	- [`mod/slack_common/version.d`](mod/slack_common/version.d): A bunch of `enum bool` definitions so that we can use `static if` instead of `version`.
- [`mod/game`](mod/game): This module provides some basic infrastructure for introspecting which version/archetype of Skyrim SE we're targeting.
- [`mod/skse64`](mod/skse64): This module provides a minimal set of bindings for SKSE.
	- [`mod/skse64/hacks`](mod/skse64/hacks): This module isn't for bindings but instead for data and functions that assist in patching SKSE at runtime.
		- [`mod/skse64/hacks/versioning.d`](mod/skse64/hacks/versioning.d): Very simple: version-numbers and file-names for the latest version of SKSE for a given archetype of Skyrim SE.
		- [`mod/skse64/hacks/offsets.d`](mod/skse64/hacks/offsets.d): A table of offsets for places to patch SKSE at runtime.
- [`mod/slack_mod`](mod/slack_mod): This module implements the actual plugin proper.
	- [`mod/slack_mod/configuration.d`](mod/slack_mod/configuration.d): Defines the configurable state for the plugin.
	- [`mod/slack_mod/entrypoint.d`](mod/slack_mod/entrypoint.d): The entrypoint for the plugin's DLL, as well as any DLL exports, and the assert handler for debug builds.
	- [`mod/slack_mod/exception_wrapper.cpp`](mod/slack_mod/exception_wrapper.cpp): A small wrapper for calling a function and catching any exceptions it may throw.
	- [`mod/slack_mod/exception_wrapper.d`](mod/slack_mod/exception_wrapper.d): D bindings for the C++ exception wrapper.
	- [`mod/slack_mod/global.d`](mod/slack_mod/global.d): All the global state for the plugin, traceable from once place, and the vectored-exception-handler.
	- [`mod/slack_mod/limits.d`](mod/slack_mod/limits.d): Provides the definitions for any hardcoded limits for the plugin's functionality.
	- [`mod/slack_mod/Save&LoadAcceleratorForSKSECosaves.def`](mod/slack_mod/Save&LoadAcceleratorForSKSECosaves.def): Defines the exports of the plugin's DLL.
	- [`mod/slack_mod/Save&LoadAcceleratorForSKSECosaves.rc`](mod/slack_mod/Save&LoadAcceleratorForSKSECosaves.rc): Defines the product-version metadata for the plugin's DLL.
	- [`mod/slack_mod/save_load.d`](mod/slack_mod/save_load.d): The actual implementation of optimised saving and loading for SKSE cosave files.
	- [`mod/slack_mod/setup.d`](mod/slack_mod/setup.d): This file is responsible for installing the optimised saving/loading hooks at runtime, and generally setting up all the fiddly global state.
- [`mod/Save&LoadAcceleratorForSKSECosaves.ini`](mod/Save&LoadAcceleratorForSKSECosaves.ini): The default configuration file for the plugin.
