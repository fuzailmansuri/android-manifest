# Android Manifests for Nothing Phone (3a) (asteroids)

Custom local manifests for building AOSP 17 (YAAP) and LineageOS 24 on Nothing Phone (3a) / Phone (3a) (`asteroids`, SM7635 SoC).

## Included Manifests
- **`asteroids.xml`**: Device, proprietary vendor, kernel, hardware and shared UI source for Nothing Phone (3a) (`asteroids`).
- **`lineage_24.xml`**: LineageOS 24.0 HAL, telephony, and Qualcomm common project overrides.

## Setup Instructions

### 1. Place in `.repo/local_manifests/`
```bash
mkdir -p .repo/local_manifests
git clone --branch yaap-17 --single-branch --depth 1 \
    https://github.com/fuzailmansuri/android-manifest.git _personal/android-manifest
cp _personal/android-manifest/asteroids.xml .repo/local_manifests/
cp _personal/android-manifest/lineage_24.xml .repo/local_manifests/
```

### 2. Sync Source Tree
```bash
repo sync -c --no-clone-bundle --optimized-fetch --prune -j$(nproc --all)
```

### 3. Apply Runtime Patches

Keep the official YAAP and LineageOS sources. Apply the ordered compatibility,
policy, PowerHAL and netlink patches using the checked command in
[patches/runtime/README.md](patches/runtime/README.md) before UI integration or
building. It skips already matching trees and rejects changed baselines. Patches
preserve the existing fixes without selecting additional forks in the manifest.

### 4. Integrate Shared UI Source
```bash
python3 packages/apps/Customizations/tools/integrate.py "$PWD" --rom yaap-17
python3 packages/apps/Customizations/tools/integrate.py "$PWD" --rom yaap-17 --apply
```

The UI source and framework, SystemUI, Settings and Launcher adapters live in
[android_packages_apps_Customizations](https://github.com/fuzailmansuri/android_packages_apps_Customizations).
The first command checks compatibility; the second applies the checked files and
hooks. Existing conflicting edits stop integration before writing. Current build
and runtime validation limits are recorded in that repository's `VALIDATION.md`.

### 5. Build Environment
```bash
source build/envsetup.sh
lunch yaap_asteroids cp2a user
m yaap
```

## Patches

Custom patches for the YAAP A17 (cp2a) tree live in [`patches/`](patches/). Each
topic is one directory with a git patch (verified clean against the synced tree)
and per-topic notes. `patches/README.md` has the full apply instructions.

| # | Topic | Repo |
|---|-------|------|
| 04 | Classic (A14-style) status bar icons | `frameworks/base` |
| 05 | Icon sizing: 17.6sp set + spacing | `frameworks/base` |
| 06 | Frosted volume / lockscreen / notifications | `build/release` |

### Quick apply
```bash
cd <repo-from-table>
git apply ../../patches/<NN-topic>/<patch-file>   # see patches/README.md for exact paths
```
Apply 04 before 05 (sizing assumes the classic icon layout). Patch 06 is
additive flag pins copied into `build/release/aconfig/cp2a/com.android.systemui/`.
