# Cameras for Redmi Note 7 (lavender) on Nura / postmarketOS 7.2.x

Русский: три камеры (основная S5KGM1, фронтальная OV13855, глубины S5K5E8)
и фокусер DW9800K на mainline-ядре 7.2.x. Один патч с драйверами, исправлениями
CAMSS и device tree, плюс конфиг, порядок загрузки модулей и tuning-файлы для
libcamera с софтверным ISP и автофокусом.

English: enables the three cameras on Redmi Note 7 / lavender with a 7.2.x
sdm660-mainline kernel, using libcamera's software ISP and autofocus.

## Status / проверено

- Device: Redmi Note 7 (lavender), NT36672A **tianma**, 4 GB.
- OS: Nura edge, kernel `7.2.15-sdm660` (sdm660-mainline + local IPA wip2/rxfix);
  tested together with the MAX98937 speaker patches.
- Userspace: `libcamera`/`libcamera-ipa`/`libcamera-tools` package version
  `99990.8.0-r3` (upstream 0.8.0 + autofocus patch series by Milan Zamazal),
  tuning YAML in `/etc/libcamera/ipa/softisp/`.
- Verified 2026-11-21: all three sensors respond, `cam -l` lists three cameras,
  main camera `Rotation = 270`, `Location = 1 (Back)`, front `Rotation = 90`.

## What is inside

- `patches/0001-lavender-camera-stack.patch` — combined patch (generated with
  `git format-patch`): SDM660 CAMSS fixes (ISPIF creation, CSIPHY clock mux,
  VFE power-domain link), generated sensor drivers `s5kgm1`, `ov13855`,
  `s5k5e8`, `dw9719`, and the lavender DT nodes on `cci_i2c0/1`.
  Contains **no** ath10k/Wi-Fi changes.
- `config/cameras.config` — kernel options, all `=m`.
- `modprobe.d/redmi-cameras.conf` — module load order.
- `tuning/*.yaml` — soft-ISP tuning for the three sensors.

## Build (host)

```sh
KDIR=/path/to/linux-7.2.15
cd "$KDIR"
git apply /path/to/this/repo/patches/0001-lavender-camera-stack.patch
for s in I2C_QCOM_CCI V4L2_CCI V4L2_CCI_I2C VIDEO_S5KGM1 VIDEO_OV13855 VIDEO_S5K5E8 VIDEO_DW9719; do
  scripts/config --module "$s"
done
git add -A && git -c user.email=you@example.com -c user.name=You commit -m 'lavender: cameras'
git tag -f -a v7.2.15 -m v7.2.15 HEAD
make ARCH=arm64 LLVM=-19 olddefconfig
make ARCH=arm64 LLVM=-19 -j4 modules          # full pass: separate M= builds fail modpost on cci_* symbols
make ARCH=arm64 LLVM=-19 -j4 qcom/sdm660-xiaomi-lavender-tianma.dtb
```

If a newly enabled module is missing, `touch .config` and re-run `make modules`.
The annotated tag at HEAD keeps the release exactly `7.2.15-sdm660` (no `+`/`-dirty`).
Compress modules with `zstd -19 -q` on the host — the phone has no `zstd`.
The patch was generated on top of the speaker tree, but its DTS hunks are
independent; `git apply --check` on a clean 7.2.15 tree also succeeds.

## Install (on the phone, root)

1. Back up `/boot/*` and `/lib/modules/7.2.15-sdm660` first.
2. Modules:
   - `qcom-camss.ko.zst` → `/lib/modules/7.2.15-sdm660/updates/` (overrides the
     stock package module);
   - `i2c-qcom-cci.ko.zst` → `.../kernel/drivers/i2c/busses/`;
   - `v4l2-cci.ko.zst` → `.../kernel/drivers/media/v4l2-core/`;
   - `s5kgm1.ko.zst`, `ov13855.ko.zst`, `s5k5e8.ko.zst`, `dw9719.ko.zst`
     → `.../kernel/drivers/media/i2c/`.
3. `depmod 7.2.15-sdm660`.
4. Install `modprobe.d/redmi-cameras.conf` into `/etc/modprobe.d/`.
5. Install the DTB to both `/boot/dtbs/qcom/sdm660-xiaomi-lavender-tianma.dtb`
   and `/boot/sdm660-xiaomi-lavender-tianma.dtb`, run `mkinitfs` (retry once if
   it fails), then re-install the DTB and run
   `boot-deploy -i initramfs -d /tmp/bd -o /boot` as described in the speaker repo
   to avoid the DTB overwrite race.
6. Userspace: install `libcamera 99990.8.0-r3` (+ `-ipa`, `-tools`) with the
   autofocus patch series, copy `tuning/*.yaml` to `/etc/libcamera/ipa/softisp/`.
7. Reboot (user action). Verify:
   `dmesg | grep 'chip id'` → `0x8d1`, `0xd855`, `0x5e80`;
   `ls /dev/media* /dev/video*`; `cam -l`; `cam -c1 -p | grep -E 'Rotation|Location'`.

Focus check: `cam -c1 -C100 >/dev/null & sleep 6;
v4l2-ctl -d $(media-ctl -e "dw9719 3-000c") -C focus_absolute`.

## Attribution / лицензии

Sensor drivers, CAMSS fixes, DT and tuning come from
https://github.com/mest2011/redmi-note7-nura/tree/main/kernel-7.2 and
`.../libcamera/`, originally from sdm660-mainline and upstream authors;
Linux parts are GPL-2.0-only, tuning/docs follow the upstream repository's
MIT license. See `ATTRIBUTION.md`. Unofficial, unaffiliated port.
