# kernel-sec-os-build

`live-build` configuration for the Kernel Sec OS "core" ISO: Debian
bookworm + XFCE + Calamares installer, booting the custom
`kernel-sec` kernel instead of Debian's stock kernel. Sibling repo to
[`kernel-sec`](../linux-6.12.41), which is where that kernel is built.

## Layout

- `auto/config`, `auto/build`, `auto/clean` -- standard live-build auto
  scripts. `lb config` options (bookworm, amd64, no debian-installer,
  Calamares does install-to-disk instead) live in `auto/config`.
- `config/package-lists/base.list.chroot` -- XFCE desktop, Calamares,
  network-manager, and other base-system packages. Arsenal category
  package lists (19 categories, curated core subset) will be added here
  as sibling `*.list.chroot` files.
- `config/packages.chroot/` -- drop the `kernel-sec` kernel `.deb`'s
  here before building (`linux-image-6.12.41-kernel-sec_*.deb`,
  `linux-headers-6.12.41-kernel-sec_*.deb`, `linux-libc-dev_*.deb`,
  built from the `linux-6.12.41` tree in WSL). live-build installs any
  `.deb` found here directly into the chroot, no package-list entry
  needed. Not committed to git (see `.gitignore`) -- copy them in
  fresh before each build.
- `config/includes.chroot/etc/sysctl.d/99-kernel-sec.conf` -- runtime
  policy that isn't a Kconfig option: `kernel.yama.ptrace_scope = 0`
  (axis 1, permissive capabilities). See
  `arch/x86/configs/kernel-sec-runtime-policy.md` in the kernel tree
  for the full rationale, including the per-binary `setcap` policy
  that still needs to land in the arsenal packaging step.

## Building

`live-build` is a Linux tool; run this from the same WSL2 (Ubuntu)
environment the kernel was built in, not from Windows directly.

```sh
sudo apt install live-build
cp /path/to/linux-image-6.12.41-kernel-sec_*.deb \
   /path/to/linux-headers-6.12.41-kernel-sec_*.deb \
   /path/to/linux-libc-dev_*.deb \
   config/packages.chroot/
lb clean
sudo lb build
```

Output is a hybrid ISO in the repo root, bootable from USB or in a VM.

## Status

Scaffold only -- not yet built or boot-tested. Next steps:

1. Copy the already-built kernel `.deb`'s into `config/packages.chroot/`
   and run a first `lb build` to validate the base pipeline (XFCE boots,
   Calamares installs, custom kernel loads).
2. Map the curated ~340-tool "core" arsenal list into per-category
   `config/package-lists/*.list.chroot` files.
