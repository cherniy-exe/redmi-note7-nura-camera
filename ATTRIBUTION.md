# Attribution

Unofficial repackaging; not affiliated with Nura, postmarketOS, sdm660-mainline
or Xiaomi.

- Kernel patch (`patches/0001-lavender-camera-stack.patch`) is derived from
  `mest2011/redmi-note7-nura`, directory `kernel-7.2/`, patches 0003-0007:
  https://github.com/mest2011/redmi-note7-nura/tree/main/kernel-7.2
  Original authors and commit trailers are preserved in the patch.
  License: GPL-2.0-only (Linux kernel code).
- `config/cameras.config`, `modprobe.d/redmi-cameras.conf`, `tuning/*.yaml`
  come from the same upstream repository (`kernel-7.2/cameras.config`,
  `scripts/`, `libcamera/tuning/`), Copyright (c) 2026 mest2011, MIT.
  The tuning files are modifications for soft-ISP; sensor gain defaults live
  in the register arrays of the GPL drivers.
- libcamera autofocus series referenced in README is by Milan Zamazal
  (Red Hat), RFC "Autofocus basics for software ISP" v2, October 2026.
- Build/install notes adapted from the `redmi-note7-linux` and
  `postmarketos-kernel-build` Hermes skills.
