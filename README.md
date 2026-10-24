# kona kernel build (Lenovo TB-J716F)

Cloud build for the CODM device kernel. Produces a bootable `Image` with two
changes applied on top of the stock config:

| option | stock | patched | why |
|---|---|---|---|
| `CONFIG_MODULE_SIG_FORCE` | `y` | `n` | allow loading unsigned `.ko` |
| `CONFIG_KPROBES` | not set | `y` | BPF/kprobe fallback path |

## What this builds

- source: `lualua99/Realking_kernel_sm8250` (kona / SM8250, 4.19.x)
- config: `device.config` — a verbatim copy of the device's `/proc/config.gz`
  with only the two options above changed
- output: `Image.gz`, `Image`, and the kona `*.dtb`

## Device facts this is matched against

| item | value |
|---|---|
| model | Lenovo TB-J716F (kona / SM8250, sd870) |
| kernel | 4.19.152-perf+ |
| compiler used by vendor | clang 10.0.7 for Android NDK |
| device tree | `qcom,kona-mtp` (generic kona MTP, not vendor-custom) |
| page size | 4096 |
| boot header | v2 |
| kernel addr | 0x8000 |
| ramdisk addr | 0x1000000 |
| tags addr | 0x100 |
| slot | A/B, currently `a` |

## How to use

1. push this repo to GitHub
2. Actions -> "Build Kona Kernel" -> Run workflow
3. wait ~30-40 min
4. download the `kona-kernel` artifact
5. repack into `boot.img` with AnyKernel3 (already in the source tree) or
   `magiskboot`

## Rollback

`backup/bk_boot_a.img` is a full 96 MB dump of the working boot partition.
Flashing it back always restores a bootable device:

```sh
fastboot flash boot_a bk_boot_a.img
```

## Notes

- The source tree is 4.19.303, the device is 4.19.152. Same series, no
  structural ABI breaks; if `uname -r` matters for a check, edit `SUBLEVEL`
  in `kernel/Makefile` to 152 before building.
- `CONFIG_LOCALVERSION="-perf"` is already in the device config, so the
  resulting version string keeps the `-perf` suffix.
- Do not ship `device.config` with `MODULE_SIG_FORCE=y` re-enabled — that
  defeats the whole point.
