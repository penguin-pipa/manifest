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

Barista needs `vendor/aospa` at commit `e6ddfbf` or newer: older copies filter
the sync list with `repo list --path-only`, which on repo v2.x drops every
project that is not checked out yet, so the device tree is never downloaded.

### Variants

```bash
./rom-build.sh pipa              # GMS by default
./rom-build.sh pipa -l           # vanilla, without GMS
./rom-build.sh pipa -g           # GMS, explicit
./rom-build.sh pipa -j"$(nproc)" # explicit job count
./rom-build.sh pipa -r           # repo sync before building
```

### Updates

For updates, run `repo sync` before the build (or use
`./rom-build.sh pipa -r`) so direct repositories that are already checked out
are refreshed. Barista only syncs new or changed manifest entries:

```bash
repo sync -j"$(nproc)"
./rom-build.sh pipa
```

If YumeMichi force-pushes, use `repo sync --force-sync -j"$(nproc)"` instead.

### Validation

A manual barista dry-run is only useful for validation, never in the normal
build flow:

```bash
python3 vendor/aospa/build/tools/barista.py aospa_pipa true
cat .repo/local_manifests/baristablend.xml
```

After a sync, these checks must all hold:

```bash
ls device/xiaomi/pipa/device.mk
ls vendor/qcom/build/tasks/generate_extra_images.mk \
   vendor/qcom/build/tasks/kernel_definitions.mk
repo list --path-only --all | grep -x "device/xiaomi/pipa"
grep -n -A5 'framework_compatibility_matrix.7.xml' \
    hardware/interfaces/compatibility_matrices/Android.bp
```

The last check must show `kernel_config_t_4.19` next to `t_5.10` and `t_5.15`.

The default launcher is PenguinOS `Launcher3` (`Launcher3QuickStep`). Pixel
Launcher is not part of this product.
