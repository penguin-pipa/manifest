# PenguinOS for Xiaomi Pad 6 (`pipa`)

Local manifest overlay for Project PenguinOS `celerity` (AOSPA `calcite` / Android 17).

## Repositories

This overlay replaces only the two repositories that contain pipa-specific
changes:

| Path | Repository | Base |
| --- | --- | --- |
| `vendor/aospa` | `penguin-pipa/android_vendor_aospa` | `Project-PenguinOS/android_vendor_aospa` |
| `hardware/interfaces` | `penguin-pipa/android_hardware_interfaces` | `Project-PenguinOS/hardware_interfaces` |

Everything else is synced by `vendor/aospa/products/pipa/beans.xml` directly
from YumeMichi's trees:

- `YumeMichi/device_xiaomi_pipa`, `YumeMichi/vendor_xiaomi_pipa`,
  `YumeMichi/kernel_xiaomi_pipa`
- `aospa-pipa/android_kernel_configs`
- `aospa-pipa/android_hardware_xiaomi`
- `aospa-pipa/android_hardware_dolby`
- `aospa-pipa/android_packages_apps_FlipFlap`
- `aospa-pipa/android_device_qcom_common`
- `aospa-pipa/android_device_qcom_sepolicy_vndr`
- `aospa-pipa/android_kernel_build`
- `aospa-pipa/android_vendor_qcom-opensource_power`
- `aospa-pipa/android_vendor_qcom-opensource_usb`
- `aospa-pipa/android_hardware_qcom_audio` (`calcite-865`)
- `aospa-pipa/android_hardware_qcom_display` (`calcite-865`)
- `aospa-pipa/android_hardware_qcom_media` (`calcite-865`)

The three `calcite-865` HALs are mapped by
`vendor/aospa/products/pipa/kona.xml`; they are not forked into this org.

## Build

```bash
repo init -u https://github.com/Project-PenguinOS/manifest -b celerity
mkdir -p .repo/local_manifests
curl -L https://raw.githubusercontent.com/penguin-pipa/manifest/celerity/pipa.xml \
    -o .repo/local_manifests/pipa.xml
repo sync -j"$(nproc)"

./rom-build.sh pipa
```

`rom-build.sh` runs `lunch aospa_pipa-userdebug` and `m otapackage`. `lunch`
automatically invokes `vendor/aospa/build/tools/barista.py`, which creates
`.repo/local_manifests/baristablend.xml` and syncs the pipa device tree,
YumeMichi's device-common forks and the `calcite-865` HALs. Do not invoke
barista manually in the normal build flow.

For updates, run `repo sync` before the build (or use
`./rom-build.sh pipa -r`) so direct repositories that are already checked out
are refreshed. Barista only syncs new or changed manifest entries.

For a vanilla build use `./rom-build.sh pipa -l`, for GMS use
`./rom-build.sh pipa -g`. A manual barista dry-run is only useful for
validation:

```bash
python3 vendor/aospa/build/tools/barista.py aospa_pipa true
cat .repo/local_manifests/baristablend.xml
```

The default launcher is PenguinOS `Launcher3` (`Launcher3QuickStep`). Pixel
Launcher is not part of this product.
