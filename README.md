# Android Manifests for Nothing Phone (3a) (asteroids)

Custom local manifests for building AOSP 17 (YAAP) and LineageOS 24 on Nothing Phone (3a) / Phone (2a) (`asteroids`, SM7635 SoC).

## Included Manifests
- **`asteroids.xml`**: Device, proprietary vendor, kernel, hardware, and Glyph packages for Nothing Phone (3a) (`asteroids`).
- **`lineage_24.xml`**: LineageOS 24.0 HAL, telephony, and Qualcomm common project overrides.

## Setup Instructions

### 1. Place in `.repo/local_manifests/`
```bash
mkdir -p .repo/local_manifests
curl -o .repo/local_manifests/asteroids.xml https://raw.githubusercontent.com/fuzailmansuri/android-manifest/seventeen/asteroids.xml
curl -o .repo/local_manifests/lineage_24.xml https://raw.githubusercontent.com/fuzailmansuri/android-manifest/seventeen/lineage_24.xml
```

### 2. Sync Source Tree
```bash
repo sync -c --no-clone-bundle --optimized-fetch --prune -j$(nproc --all)
```

### 3. Build Environment
```bash
source build/envsetup.sh
lunch yaap_asteroids-user
m yaap
```
