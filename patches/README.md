# YAAP asteroids — custom patches

Custom patches for the YAAP Android 17 (cp2a) tree, device `nothing/asteroids` (SM7635).
All `git apply` commands are run **from inside the target repo** unless noted.
Verify before applying: `git apply --check <patch>`.

---

## 06 — Frosted volume / lockscreen / notifications (EvX-style)

**Repo:** `build/release` (its own git repo; files are additive, no patch needed)
**Files:** 4 flag pins dropped into the existing `aconfig_values` glob slot
(`cp2a/com.android.systemui/*_flag_values.textproto`)

```
cd build/release
cp ../../patches/06-frost-pins/*.textproto aconfig/cp2a/com.android.systemui/
git add aconfig/cp2a/com.android.systemui/
```

Pins (all ENABLED + READ_ONLY, cp2a):
- `blur_on_more_surfaces` — volume/power dialog frost (what EvX pins)
- `enable_lockscreen_blur` — keyguard blur surfaces
- `lockscreen_blur_for_notifications` — frosted lockscreen notifications
- `notification_row_transparency` — **the missing gate**: all notif-frost code
  early-returns without it; this is why pinning blur flags alone does nothing

Verified end-to-end: framework `config_enableBlurExpansion=true` (no device
overlay), `ro.surface_flinger.supports_background_blur=1` in
`device/nothing/asteroids/vendor.prop`, runtime gates clear. Knobs:
`volume_dialog_background_surface_blur_radius` (23dp),
`notification_background_blur_radius`, `volume_dialog_view_background_blur`.

---

## 07 — DynamicBar SystemUI Integration

**Repo:** `frameworks/base`
**Files:** SystemUI DynamicBar controllers, Compose island components, Monet accenting, and hardware-accelerated blur surfaces.

```bash
cd frameworks/base
git apply ../../patches/07-dynamicbar-systemui/frameworks-base-dynamicbar.patch
```

## 08 — DynamicBar Settings Configuration

**Repo:** `packages/apps/Settings`
**Files:** DynamicBar preferences controller under `Settings → Display` with native search indexing (`BaseSearchIndexProvider`).

```bash
cd packages/apps/Settings
git apply ../../patches/08-dynamicbar-settings/packages-apps-settings-dynamicbar.patch
```

## 09 — Lineage AConfig Notification & Lockscreen Blur

**Repo:** `vendor/lineage`
**Files:** Trunk stable AConfig flag pins enabling SystemUI notification and lockscreen background blur.

```bash
cd vendor/lineage
git apply ../../patches/09-lineage-aconfig-blur/vendor-lineage-aconfig-blur.patch
```

## 11 — Quickstep App Launch Blur Suppression

**Repo:** `device/nothing/asteroids`
**Files:** `device.mk`, `vendor.prop` (`ro.launcher.blur.appLaunch=0`) to eliminate SurfaceFlinger dual-Kawase frame drops during app launch.

```bash
cd device/nothing/asteroids
git apply ../../patches/11-launcher-blur/device-nothing-asteroids-launcher-blur.patch
```
