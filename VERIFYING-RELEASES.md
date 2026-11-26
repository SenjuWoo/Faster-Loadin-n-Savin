# Verifying releases

This project ships a native SKSE DLL. Release verification should combine source review, reproducible packaging, hashes, PE-section comparison, and static-import inspection rather than relying on any single signal.

The tools in this repository use only Python's standard library.

## What 1.7.1 is

Version 1.7.1 fixes the DLL version resource (1.7.0 carried the literal text `1.7.0.0\0`) and restamps the version; the
save/load implementation is unchanged from 1.7.0. Comparing the published `ae1170` DLLs of both releases:

```text
python tools/verify-release-binaries.py compare <1.7.0.dll> <1.7.1.dll>
```

reports 360 differing bytes: 29 in `.rdata` (version strings, error-dialog title), 321 in `.rsrc` (version resource), 1 in
`.text`, 3 in `OptionalHeader.CheckSum` and 6 in header padding. The single `.text` byte is the immediate of
`GetReleaseVersion`'s `mov eax, imm32; ret` (`b8 00 00 07 01 c3` in 1.7.0, `b8 10 00 07 01 c3` in 1.7.1, that is
`0x01070000` to `0x01070010`). Because `.text` differs, the tool prints "EXECUTABLE CODE DIFFERS"; that is the
expected result for a version bump of this code base.

All 18 DLL pairs (nine game builds, Universal and High-End) show the same picture: exactly one differing `.text` byte
each. The 18 INI files are byte-identical, and in `fomod/` only `info.xml` differs (the version and the Website link).

## What 1.5.1 is

Version 1.5.1 contains no save/load implementation changes from 1.4.0 or 1.5.0. It corrects the installer, packaging scripts, documentation, licensing guidance, and release validation. The sixteen DLLs are version-stamped `1.5.1.0`.

For each matching DLL, comparison against 1.5.0 must report:

- no differences in `.text`;
- differences only in `.rdata` release metadata (including the sanitized CodeView PDB path), `.rsrc` version resources, and `OptionalHeader.CheckSum`.

```text
python tools/verify-release-binaries.py compare <1.5.0.dll> <1.5.1.dll> --strict
```

## Manifest

Each release archive includes `RELEASE-MANIFEST.txt`, generated from the staged payload:

```text
python tools/verify-release-binaries.py manifest <extracted-release>
```

The archive SHA-256 is distributed beside the ZIP as a separate `.sha256` file. It cannot be embedded inside the archive without changing the archive hash.

## Static import inspection

```text
python tools/verify-release-binaries.py imports <dll>
```

This lists the DLL's static import table and flags APIs associated with networking, process creation, remote-memory writes, and persistence. The 1.5.1 DLLs have the same imports as 1.5.0.

Static imports are **not proof of every capability** a native binary could exercise. APIs can be resolved dynamically, and this codebase performs limited dynamic symbol resolution against modules already loaded by Skyrim. The stronger evidence is the combination of published source, reviewable call sites, import inspection, hashes, and section-level binary comparison.

## Reproducing the package

On Windows with the documented compiler toolchain:

```powershell
./scripts/update-version-number.ps1 1.5.1.0
./package-release.ps1 -Version 1.5.1
```

The packager builds both CPU profiles for all eight supported game versions, validates the FOMOD, rejects development artifacts, generates `RELEASE-MANIFEST.txt`, creates the ZIP, and writes a SHA-256 sidecar.

## Source provenance

- Original upstream: https://github.com/just-harry/save-load-accelerator-for-skse-cosaves
- Maintenance fork: https://github.com/SenjuWoo/Faster-Loadin-n-Savin
- License: 0BSD
