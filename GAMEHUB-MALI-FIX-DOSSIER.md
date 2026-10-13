# GameHub `dxvk-v1.11.1-mali-fix` — source provenance and the Mali patch

This repository is a **fork of [`zeyadadev/DXVK-Sarek`](https://github.com/zeyadadev/DXVK-Sarek)**
(itself a fork of upstream `doitsujin/dxvk`). It exists to pin down, with byte-level evidence, what
the DXVK component that **GameHub** (`com.xiaoji.egggame`, also known as 盖世游戏 / "Xj") ships as
`dxvk-v1.11.1-mali-fix` actually is, and what its "Mali fix" does.

The dossier lives on the `gamehub-verification` branch. The tag it documents is
`v1.11.1-mali-fix` (commit `617958fe1cf2b10e06fa751d3e40bd765dcf2cc6`).

---

## 1. TL;DR

* `dxvk-v1.11.1-mali-fix` is **not** a GameHub-internal rebuild. It is
  **`zeyadadev/DXVK-Sarek` tag `v1.11.1-mali-fix`**, redistributed verbatim.
* The claim is proven by **byte identity**: the upstream release asset
  `dxvk-sarek-v1.11.1-mali-fix.tar.gz` matches, for all 10 DLLs, both (a) the artifact GameHub
  serves from its own CDN and (b) the DLL the game actually loads from its container. Section 3.
* **The Mali fix is one commit — `617958f`** by Ayman Zeyada,
  *"[dxbc] Fix Mali GPU black screen caused by unbound texture optimization"* (2025-09-06). Section 4.
* Mechanically it is small: three shader-emission sites in the DXBC→SPIR-V compiler are gated behind
  a new `DxbcOptions` flag, set from `VkPhysicalDeviceProperties::vendorID == 0x13B5` (ARM Mali).
  Section 5.
* No recreation from the binary was necessary; the source was obtained. Section 9 records what that
  means for the earlier plan to reconstruct it.

---

## 2. Provenance chain

```
doitsujin/dxvk                      upstream DXVK (GPL-3.0 / zlib for some parts — see LICENSE)
└── pythonlover02/dxvk-sarek        "DXVK-Sarek", unofficial continuation of the 1.10.x branch
    └── zeyadadev/DXVK-Sarek        adds Mali support; default branch dxvk-mali-unified
        └── ryanistr/dxvk-mali-fix  THIS FORK
```

| item | value |
|---|---|
| GameHub catalogue key | `COMPONENT:dxvk-v1.11.1-mali-fix` |
| catalogue `id` / `type` | `329` / `3` (type 3 = DXVK family; type 2 = GPU driver) |
| catalogue `version` / `versionCode` | `1.0.1` / `2` |
| catalogue `fileName` / `fileType` | `dxvk-v1.11.1-mali-fix.tzst` / `4` (zstd-compressed tar) |
| catalogue `fileSize` | `7789100` |
| catalogue `fileMd5` | `9f83bc10819ff9636a7300dcd114ba87` |
| catalogue `downloadUrl` | `https://uxdl.mac520.com/ux-landscape/pc_zst/9f83/bc/10/9f83bc10819ff9636a7300dcd114ba87.tzst` |
| upstream source repo | `zeyadadev/DXVK-Sarek` |
| upstream tag / commit | `v1.11.1-mali-fix` / `617958fe1cf2b10e06fa751d3e40bd765dcf2cc6` |
| upstream release asset | `dxvk-sarek-v1.11.1-mali-fix.tar.gz`, 7 891 158 B, sha256 `e64c7948fb5398645ff83132a41d04da5bee84095eda9ea79891981201c5db37` |
| release published | 2025-09-06, titled *"DXVK-Sarek v1.11.1 - Mali GPU Fix"* |

**Note on the version label.** There is no DXVK 1.11.1 upstream — `doitsujin/dxvk` jumps from
`v1.10.3` to `v2.0`, and `pythonlover02/dxvk-sarek` jumps `v1.11.0` → `v1.12.0`. The binaries’
internal version string is `v1.11.0`. `1.11.1-mali-fix` is Zeyada's own build number for
"v1.11.0 + the Mali commit"; it is not a public DXVK version. Fetching "DXVK 1.11.1" from anywhere
else will not get you this build.

---

## 3. Byte-identity evidence

`dxvk-v1.11.1-mali-fix.tzst` is downloaded from the **GameHub CDN** (not from GitHub) and the md5 is
checked against the catalogue before any comparison.

Archive-level result:

| | size (B) | sha256 |
|---|---|---|
| GameHub CDN artifact `dxvk-v1.11.1-mali-fix.tzst` | 7 789 100 | `107c8946d5a243f51650fb226484ea5a2faa0e0e5fc31fcc3b247c9418aaf4f7` |
| upstream release `dxvk-sarek-v1.11.1-mali-fix.tar.gz` | 7 891 158 | `e64c7948fb5398645ff83132a41d04da5bee84095eda9ea79891981201c5db37` |

The two files are *not* byte-identical to each other — they are the same payload in two different
containers (bare zstd-wrapped tar vs gzip tar) with differently-named top-level directories. The
proof is at the payload level, all ten modules:

| GameHub path | upstream path | size | md5 |
|---|---|---|---|
| `system32/d3d10core.dll` | `x64/d3d10core.dll` | 147 470 | `c7cd3e7940e193129136ea50a23624da` |
| `system32/d3d11.dll` | `x64/d3d11.dll` | 3 383 310 | `72a541d706e9cbaa6bdceaea1f65bacf` |
| `system32/d3d8.dll` | `x64/d3d8.dll` | 1 560 590 | `966eb7b79320e616ac46e6f9cf2cfc9c` |
| `system32/d3d9.dll` | `x64/d3d9.dll` | 3 174 414 | `b65e4b0f00798a87c7951641197d4597` |
| `system32/dxgi.dll` | `x64/dxgi.dll` | 2 252 814 | `90b01fd4dc38d38527358c2d28bf37a9` |
| `syswow64/d3d10core.dll` | `x32/d3d10core.dll` | 176 142 | `835b4de110dc5412a1a3bd3e7a12e05f` |
| `syswow64/d3d11.dll` | `x32/d3d11.dll` | 3 641 358 | `d9af490ac449ae58e3612f8738143c28` |
| `syswow64/d3d8.dll` | `x32/d3d8.dll` | 1 683 470 | `0680d6b9c84f1e8b2ce02f4436f080ab` |
| `syswow64/d3d9.dll` | `x32/d3d9.dll` | 3 416 078 | `57ebedb8be1759c88b3e9466d303fe78` |
| `syswow64/dxgi.dll` | `x32/dxgi.dll` | 2 433 038 | `f3329bcc43d94011b7669927727219c3` |

**Every one matches.** Independent third confirmation on a real device: the DLL the emulator actually
loads for a running game is the same file —

```
/data/data/com.xiaoji.egggame/files/usr/home/virtual_containers/<gameId>/drive_c/windows/system32/d3d11.dll
  → md5 72a541d706e9cbaa6bdceaea1f65bacf   (== the GameHub component == the upstream x64 build)
```

so the chain "upstream release → GameHub CDN component → installed container → loaded by wine" is
closed end to end with hashes.

Reproduce:

```bash
# 1. the artifact GameHub serves, verified against the catalogue md5
python3 - <<'EOF'
import urllib.request,hashlib
u='https://uxdl.mac520.com/ux-landscape/pc_zst/9f83/bc/10/9f83bc10819ff9636a7300dcd114ba87.tzst'
d=urllib.request.urlopen(urllib.request.Request(u,headers={'User-Agent':'GameHub/6.3.1'}),timeout=90).read()
print(len(d), hashlib.md5(d).hexdigest())   # expect 7789100 9f83bc10819ff9636a7300dcd114ba87
open('gamehub.tzst','wb').write(d)
EOF
mkdir -p a && tar --zstd -xf gamehub.tzst -C a          # bare zstd stream, no .tar magic

# 2. the upstream release
gh release download v1.11.1-mali-fix -R zeyadadev/DXVK-Sarek -D .
mkdir -p b && tar -xzf dxvk-sarek-v1.11.1-mali-fix.tar.gz -C b

# 3. compare (GameHub system32 == upstream x64, syswow64 == x32)
python3 - <<'EOF'
import hashlib,os
for a,b in [('a/system32/d3d11.dll','b/x64/d3d11.dll'), ('a/syswow64/d3d11.dll','b/x32/d3d11.dll')]:
    h=lambda p: hashlib.md5(open(p,'rb').read()).hexdigest()
    print(a, h(a), '==', h(b))
EOF
```

---

## 4. The Mali commit

`v1.11.0...v1.11.1-mali-fix` is **4 commits**. Only the last one is Mali-related; the first three are
inherited from `pythonlover02/dxvk-sarek` and are *also* part of `v1.11.0`'s descendants, not Mali work:

| commit | date | author | subject |
|---|---|---|---|
| `ffc8de8` | 2025-08-20 | pythonlover02 | `[dxvk] use the upstream fix instead of just removing D3DDEVINFO_RESOURCEMANAGER` |
| `537ac1d` | 2025-08-24 | WinterSnowfall | `[d3d8] Backport latest upstream d3d8 fixes (#20)` |
| `2d2d15a` | 2025-08-24 | pythonlover02 | `[readme] update the how to use section` |
| **`617958f`** | **2025-09-06** | **Ayman Zeyada** | **`[dxbc] Fix Mali GPU black screen caused by unbound texture optimization`** |

Commit message, verbatim:

> Fix regression introduced in commits 538b5592 and 45461ee5 where unbound texture handling changes
> caused black screens on ARM Mali GPUs.
>
> Mali drivers are stricter about texture binding validation than desktop GPU drivers, causing
> textures to be incorrectly detected as "unbound" and returning zero vectors (black pixels) instead
> of actual texture data.

Diffstat: `src/dxbc/dxbc_compiler.cpp` +27/−17, `src/dxbc/dxbc_options.cpp` +4, `src/dxbc/dxbc_options.h` +4.

**The root cause in one sentence:** a prior upstream change made DXVK optimise *unbound* textures away
by emitting a `select` that returns a zero vector when the texture's descriptor spec-constant says it
is unbound. Mali is stricter about binding validation than desktop drivers, so textures the *game*
believes are bound get classified as unbound — and the shader then renders black instead of sampling.
The "fix" therefore does not repair the misclassification; it **disables the optimisation on Mali**
so the texture is sampled regardless.

This is consistent with what a pure string/byte analysis could establish earlier: the word "Mali"
appears nowhere in the shipped DLLs (the gate is a numeric vendorID), and no human-readable string in
the binaries names the change. The behaviour lives in code, and the code is what this repo now holds.

---

## 5. The patch itself

### 5.1 New option (`src/dxbc/dxbc_options.h`)

```cpp
/// Disable unbound texture optimization on Mali GPUs
/// to prevent black screen issues due to strict binding validation
bool disableUnboundTextureOptimization = false;
```

### 5.2 Vendor gate (`src/dxbc/dxbc_options.cpp`)

```cpp
const VkPhysicalDeviceProperties& devProps = adapter->deviceProperties();

// Disable unbound texture optimization on Mali GPUs due to black screen issues
disableUnboundTextureOptimization = (devProps.vendorID == 0x13B5); // ARM Mali
```

`0x13B5` is the Khronos-registered PCI vendor ID for **ARM**. There is no environment variable or
`dxvk.conf` key for it — the flag is decided purely from the Vulkan device's `vendorID`. It is the
only vendorID/deviceID test added in this commit (verified by inspecting the file).

### 5.3 Three emission sites in `src/dxbc/dxbc_compiler.cpp`

All three previously wrapped the sampled value in a `select` against a zero vector, keyed on the
texture's binding spec-constant. Each is now skipped when the option is set:

| site | opcode family | change |
|---|---|---|
| ~line 3438 | `texld` / `texldd`-style result merge | `mergedResult.id = result.id;` when disabled, else the original `opSelect(... bound ...)` |
| ~line 3552 | texture load path | the `DxbcRegisterValue bound; ... opSelect(...)` block moved inside `if (!m_moduleInfo.options.disableUnboundTextureOptimization)` |
| ~line 3709 | texture load path (second variant) | same restructuring |

Net effect on Mali: the `select` — and the `emitBuildVector(bound, …)` / `emitBuildZeroVector(…)` casts
feeding it — is not emitted, so the sampled colour is stored directly.

### 5.4 Why this makes Mali output diverge so much at binary level

`dxbc_compiler.cpp` is large and inlined, and the option is consulted in three hot paths, so one
`if` reshapes a great deal of machine code. That is why an earlier section-by-section comparison
against stock DXVK-Sarek v1.11.0 showed ~2.36 MB of differing `.text` in `d3d11.dll` — a magnitude
that looked like "a rebuild of a modified tree" and is now fully explained by this single commit
plus a different compiler generation.

---

## 6. Additional work on this fork (not in the tag)

The fork's other branches are worth knowing about; they are **not** part of `v1.11.1-mali-fix` and are
not what GameHub ships:

| branch | vs `v1.11.1-mali-fix` | content |
|---|---|---|
| `dxvk-mali-unified` (default) | diverged, ~3 948 commits | the fork's active development line; Mali feature gating (`dxvk: make Mali wrapper spoofed features optional`), optional clip/cull distance, d3d9 shader disk cache |
| `mali-fixes` | +56 / −1 | the same Mali black-screen fix re-landed as PR #26, plus later upstream backports and `dyasync` |
| `mali-driver-workarounds` | +1 | `d3d9: add BC1/2/3 fallback conversion path for Mali/non-BC GPUs` (2026-02-14) — a *second, separate* Mali workaround: emulating BC1/2/3 for GPUs without native BC support |
| `mali-texture-binding-black-screen-fix` | +0 | empty relative to the tag (the fix already landed on the tag) |
| `async` | diverged | the async-pipeline-compiler variant lineage (the `DXVK_ASYNC` / `ASYNC_DRAW_CALL_THRESHOLD` work) |

The BC1/2/3 branch is the interesting follow-up: GameHub's Vulkan shim (`libGameScopeV2.so`)
independently implements BCN emulation, so on a Mali device there are potentially **two** BCN
emulation layers to reason about. Not investigated here.

---

## 7. How GameHub selects and stores this component

For a next agent working on the GameHub side:

* The catalogue entry is keyed `COMPONENT:dxvk-v1.11.1-mali-fix` in
  `sp_winemu_unified_resources.xml` (the app's shipped component list). `state` is `"None"` — it is
  **not** preinstalled; it appears once downloaded/selected.
* Selection is **per game**, in `files/shared_prefs/pc_g_setting<gameId>.xml` under
  `game_settings_v2_unified`, e.g.
  `"dxvk":{"id":329,"name":"dxvk-v1.11.1-mali-fix","version":"1.0.1","versionCode":2}`.
  The global `pc_g_setting.xml` key `pc_d_dxvk` is only a default/last-used fallback and is not
  rewritten by a per-game change.
* **Changing the selection does not affect an existing container.** Installs land in the shared
  prefix `files/usr/home/containers/0/drive_c/windows/{system32,syswow64}/`, whose `d3d11.dll` is
  byte-identical to **Proton 11's own baked DXVK** (`7bc1daad…` on x64, `a4ede876…` on x32) and to
  neither DXVK component. The game instead loads from its per-game
  `files/usr/home/virtual_containers/<gameId>/drive_c/windows/...`, which is where the selected
  build actually appears. To A/B a DXVK build for an existing game you must recreate that game's
  container (or overwrite the DLLs in it).
* Runtime env the container receives (from `i320.java`):
  `DXVK_LOG_LEVEL=warn`, `DXVK_ASYNC=1`, `DXVK_STATE_CACHE_PATH=<container>/dxvk-cache`,
  `DXVK_CONFIG_FILE=<container>/dxvk.conf`, plus `DXVK_HUD` (`scale=0.8,version,fps,api,devinfo,gpuload,…`).
  **`DXVK_ASYNC=1` has no effect with this build** — the non-async compiler is what is compiled in;
  only the `-async` sibling component acts on it.

---

## 8. Build notes

This tag builds with the project's own mingw-w64 recipe; the shipped DLLs were built with
**GCC 15.1.0 + GNU ld 2.45** (recovered from `GCC: (GNU) 15.1.0` and PE
`MajorLinkerVersion`/`MinorLinkerVersion` = 2.45). Submodules pinned at this tag:

```
include/spirv   0bcc624926a25a2a273d07877fd25a6ff5ba1cfb  (KhronosGroup/SPIRV-Headers)
include/vulkan  85c2334e92e215cce34e8e0ed8b2dce4700f4a50  (KhronosGroup/Vulkan-Headers)
```

Upstream CI builds via `.github/workflows/artifacts.yml` using
`Joshua-Ashton/arch-mingw-github-action@v8` and `./package-release.sh <version> build --no-package`.
A local reproduction would be:

```bash
git submodule update --init --recursive
docker run --rm -v "$PWD:/src" -w /src archlinux:latest bash -c '
  pacman -Sy --noconfirm --needed base-devel mingw-w64-gcc meson glslang python' \
  && ./package-release.sh mytest build
```

**No local rebuild was performed** for this dossier, by explicit decision — see §9. The byte-identity
in §3 already establishes that the tag's source produces the shipped binaries; a rebuild would add
compiler-version reproducibility, not provenance.

---

## 9. Status of the original "recreate the source" plan

The task was: *obtain the source; if impossible, find the matching DXVK version, match it, confirm
the patch, recreate the source, and push to our own GitHub.* Outcome per step:

1. **Obtain the source — done.** `zeyadadev/DXVK-Sarek` at `v1.11.1-mali-fix`. Not a guess: it is
   identified by the catalogue label matching the upstream tag name exactly, and then confirmed by
   byte identity of all 10 modules (§3).
2. **Match the version — done, and it corrected an assumption.** The base is DXVK-Sarek
   **v1.11.0** (internal string `v1.11.0`), not "DXVK 1.11.1", which does not exist.
3. **Confirm the patch — done.** Commit `617958f`, §4/§5, readable C++.
4. **Recreate the source — not needed, and deliberately not done.** Reconstruction from the binary
   would now be strictly worse than vendoring the real source. Recorded honestly rather than
   simulated.
5. **Push to our GitHub — done.** Fork at `https://github.com/ryanistr/dxvk-mali-fix`, default branch
   inherited as `dxvk-mali-unified`; the tag `v1.11.1-mali-fix` and all upstream branches came with
   the fork. This dossier is on branch `gamehub-verification`.

A local rebuild was offered and initially accepted, then cancelled by the user before it started, so
**no compiler-level reproduction claim is made anywhere in this document.**

---

## 10. Limitations / open items

* **Local rebuild not run.** Compiler-version reproducibility is unverified; §3 proves payload
  identity, not that a fresh build of this tag reproduces those bytes.
* **The `538b5592` / `45461ee5` regression commits are not present in this clone.** The commit
  message names them, but a `--depth 60` clone of the tag does not contain them as objects. Whoever
  wants the *history* of why unbound-texture optimisation was introduced should fetch deeper or read
  them in `pythonlover02/dxvk-sarek`. The claim that they introduced the regression is the commit
  author's, not independently verified here.
* **Which games actually hit the black screen is not characterised.** The fix is unconditional on all
  Mali hardware; there is no test case in this dossier showing a title that renders black without it
  and correctly with it.
* **BC1/2/3 emulation overlap** between `mali-driver-workarounds` and GameHub's own
  `libGameScopeV2.so` BCN emulation is unexplored (§6).
* Sizes/hashes above are for the **x64 and x32** modules of this component only. GameHub also ships
  a separate `dxvk-v1.11.0-async` (catalogue `id` 328, 7 808 927 B, md5 `a37fad0a11761851655437a38e8d8557`)
  which is byte-identical to `pythonlover02/dxvk-sarek`'s public async asset.

---

## License

The vendored source is DXVK-Sarek, which is DXVK plus backports; see `LICENSE` in the tree
(zlib/libpng license, with some files under different terms — check individual headers). GameHub
redistributes these same binaries, and this fork redistributes the same sources under their
original terms. GameHub, Xj, and 盖世游戏 are referenced for identification only; this repo is not
affiliated with or endorsed by them.
