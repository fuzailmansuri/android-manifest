# Runtime fixes against official sources

Keep the official YAAP and LineageOS repositories selected by the manifests.
These ordered git-am patch series carry the existing compatibility, policy,
PowerHAL and netlink fixes without requiring additional forks. Original authors,
dates, commit messages and upstream mechanism references remain in the patches.

Run this block once from the ROM root after a fresh sync and before UI integration.
It checks all eight baselines, clean working trees and patch checksums before applying
any series. A tree already matching the recorded result is skipped. A changed baseline
stops application; adapt and validate the affected patches before updating it.

```bash
python3 - <<'PY'
from pathlib import Path
import hashlib, json, subprocess
rom = Path.cwd()
patch_root = rom / "_personal/android-manifest/patches/runtime"
series = json.loads((patch_root / "series.json").read_text())["repositories"]
pending = []
def git(repo, *args):
    return subprocess.run(["git", "-C", str(repo), *args], check=True,
                          capture_output=True, text=True).stdout.strip()
for entry in series:
    repo = rom / entry["path"]
    if git(repo, "rev-parse", "HEAD^{tree}") == entry["target_tree"]:
        continue
    if git(repo, "status", "--porcelain=v1"):
        raise SystemExit(f"Working tree has changes: {repo}")
    if git(repo, "rev-parse", "HEAD") != entry["baseline"]:
        raise SystemExit(f"Official baseline changed: {repo}")
    files = [patch_root / patch["file"] for patch in entry["patches"]]
    for file, patch in zip(files, entry["patches"]):
        if hashlib.sha256(file.read_bytes()).hexdigest() != patch["sha256"]:
            raise SystemExit(f"Patch checksum mismatch: {file}")
    pending.append((repo, entry, files))
for repo, entry, files in pending:
    subprocess.run(["git", "-C", str(repo), "am", *map(str, files)], check=True)
    if git(repo, "rev-parse", "HEAD^{tree}") != entry["target_tree"]:
        raise SystemExit(f"Applied tree mismatch: {repo}")
print(f"Applied {len(pending)} runtime patch series.")
PY
```

An unexpected git-am failure stops immediately. Earlier repositories remain patched;
resolve or abort the failed repository before proceeding. This command does not sync,
build, touch signing keys or change GitHub repositories.

| Official repository | ROM path | Baseline | Patches |
| --- | --- | --- | --- |
| [yaap/hardware_lineage_compat](https://github.com/yaap/hardware_lineage_compat) | `hardware/lineage/compat` | `df3745aa58b39e5288e423cbfea436041780defd` | 1 |
| [yaap/frameworks_native](https://github.com/yaap/frameworks_native) | `frameworks/native` | `cc7e9b60913c9b5c0b76462b45912f5815d98309` | 1 |
| [yaap/device_lineage_sepolicy](https://github.com/yaap/device_lineage_sepolicy) | `device/lineage/sepolicy` | `e31545ff1365ec77cf8a660ea034102bb46fdd07` | 4 |
| [yaap/system_sepolicy](https://github.com/yaap/system_sepolicy) | `system/sepolicy` | `98a168d264593282c13dc120dd5531e38f9cb0a7` | 1 |
| [LineageOS/android_vendor_qcom_opensource_power](https://github.com/LineageOS/android_vendor_qcom_opensource_power) | `vendor/qcom/opensource/power` | `2a237fa09a5f410e559ec8b7b3473969ac64d9bb` | 1 |
| [yaap/vendor_yaap](https://github.com/yaap/vendor_yaap) | `vendor/yaap` | `7ac9801a850fe3db9e08cadf23217a1e83495deb` | 2 |
| [yaap/system_core](https://github.com/yaap/system_core) | `system/core` | `35122337d18c4d086be1c0d821adc70145cec0ce` | 1 |
| [yaap/system_netd](https://github.com/yaap/system_netd) | `system/netd` | `6db396c7ebc06b6b4c30039826141661e3e1ebec` | 1 |

The exported series are replayed in isolated copies and compared with the complete
active source trees. That proves patch preservation, not compilation, boot or runtime
stability. No compilation was run for this publication correction.
