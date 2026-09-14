# YAAP asteroids — custom patches

Custom patches for the YAAP Android 17 (cp2a) tree, device `nothing/asteroids` (SM7635).
All `git apply` commands are run **from inside the target repo** unless noted.
Verify before applying: `git apply --check <patch>`.

Apply 04 before 05 (sizing assumes the classic icon layout).

---

## 04 — Classic (A14-style) status bar icons, consistently forced

**Repo:** `frameworks/base`
**Files (13):** `NewStatusBarIcons.kt`, `SignalDrawable.java`, `TelephonyIcons.java`,
`WifiUtils.kt`, `WifiIcons.java`, `DarkIconDispatcherImpl.java`, `DualToneHandler.kt`,
`StackedMobileBindableIcon.kt`, `SystemUiCarrierConfig.kt`, `MobileIconInteractor.kt`,
`MobileIconInteractorKairos.kt`, `BatteryStatusEventComposeChip.kt`,
`ModernStatusBarMobileView.kt`

```
cd frameworks/base
git apply ../../patches/04-classic-statusbar-icons/frameworks-base-classic-icons.patch
```

Notes: bypasses the aconfig `new_status_bar_icons` gate in every consumer
(bp3a pins it ENABLED READ_ONLY, so the classic pipeline is forced at the call
sites). Also defuses `unsafeAssertInNewMode()` crash path and guards
`configureLayoutForNewStatusBarIcons()` so the 12sp `_updated` slots can never
reapply. This fixes the "small + broken battery tint" half-state from the
original community patch.

## 05 — Icon sizing: 17.6sp set + tightened spacing

**Repo:** `frameworks/base`
**Files:** `packages/SystemUI/res/values/dimens.xml`,
`packages/SystemUI/res/layout/mobile_signal_group.xml`,
`core/res/res/values/dimens.xml`

```
cd frameworks/base
git apply ../../patches/05-icon-sizing/frameworks-base-icon-sizing.patch
```

Values: mobile signal + wifi 17.6sp (equal weight, same 24dp canvas), system
icons (`status_bar_system_icon_size`) 17.6dp, battery 15.2sp / 9.1sp wide
(stock 0.867 ratio kept), battery extra spacing 2sp, system icon spacing 2.5sp,
RAT `paddingStart` 1dp -> 0.5dp (tightens signal<->4G/5G gap). Tablet buckets
(sw1500/sw1900) untouched. **Apply 04 first — 05 assumes the classic layout.**

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
