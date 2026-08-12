# SM-S928N / S928NKSS6DZE1 port record

This file records the offline port of the Galaxy S24 Ultra Korean firmware to a
new payload profile `e3q-S928NKSS6DZE1`. The kernel is byte-identical to the
existing `e3q-S928USQS6DZF2` (SM-S928U) target apart from the head-section
packing and the build banner, so all symbol offsets are reused from the DZF2
record rather than re-derived.

## Firmware identity

```text
model: SM-S928N
device: e3q
region: KOO
AP/PDA: S928NKSS6DZE1
CSC:    S928NOKR6DZE1
CP:     S928NKSS6DZE1
kernel release: 6.1.145-android14-11-33419968-abS928NKSS6DZE1
```

## Extracted image hashes

```text
boot.img      size 100663296  SHA-256 68025B47044EB2C7B82CE1F6915E2A8D5F13B2CFA55637F29EBFD5BC4E4A272B
kernel        size 38005248   SHA-256 AA40D2A1E8EA37AF8931874AAC26F9E199E749853C8D91C8D3FF095D1A04D833
```

The ARM64 Image header matches DZF2 exactly: `text_offset = 0`,
`image_size = 0x26f0000`, `flags = 0xa`, magic `ARMd`.

## Kernel equivalence to DZF2

`vmlinux-to-elf` recovered 107254 symbols at image base
`0xffffffc008000000`. Every exploit-relevant symbol offset is identical to the
DZF2 `e3q-S928USQS6DZF2` target:

```text
call_usermodehelper_exec_work 0x000d39cc
noop_llseek                   0x003a14e4
generic_file_splice_read      0x003ef340
configfs_read_iter            0x004712a4
configfs_bin_write_iter       0x004717d4
ashmem_ioctl                  0x00d3a314
compat_ashmem_ioctl           0x00d3ac4c
ashmem_mmap                   0x00d3aca4
ashmem_open                   0x00d3aed0
ashmem_release                0x00d3af58
ashmem_show_fdinfo            0x00d3b078
anon_pipe_buf_ops             0x01219d90
ashmem_fops                   0x013d1140
kmalloc_caches                0x0176c6f8
system_unbound_wq             0x0223ae60
nfulnl_logger                 0x02242a20
init_task                     0x0224f8c0
ashmem_miscs (+0x10)          0x023bb5b0
root_task_group               0x0244cd80
selinux_state.enforcing       0x02521588
sysctl_bootid                 0x026046e8
```

The `target.h` is therefore a copy of the DZF2 target with only the build
label, build fingerprint, and P0 fingerprint path changed.

## Head-section mirror (inverse slide)

The DZE1 raw Image stores the first `0x1f0000` bytes in reverse chunk order
relative to DZF2: `DZE1_file[X] == DZF2_file[0x1f0000 - X]`. The generated
`p0_fingerprint.h` is therefore the mirror of the DZF2 table. Following the
DZE1/DZF3/DZDR precedent, `APP_P0_FINGERPRINT_INVERSE_SLIDE` is enabled:

```c
#define APP_P0_FINGERPRINT_INVERSE_SLIDE 1
```

This runtime slide-direction flag is a best-effort offline decision and must be
confirmed on hardware.

## Built artifacts

```text
artifacts/e3q-S928NKSS6DZE1/cve-2026-43499-app.so
  size 104128  SHA-256 B2A3D04176933107FAE507EDAE17B1463A5A8FE46E1A670D581CF3143DC62C30

kernelsu/android14-6.1_kernelsu-e3q-S928NKSS6DZE1-kdp.ko
  size 400200  SHA-256 2478422362FADE18D9A917C89D058621AB17B8DCF72BF0800F50FC0DF035DBF4
  vermagic 6.1.145-android14-11-33419968-abS928NKSS6DZE1 SMP preempt mod_unload modversions aarch64

kernelsu/ksud-e3q-S928NKSS6DZE1-kdp
  size 5090384  SHA-256 26888CB9D1532F62004FBDD204BB1EDEA422450893A9ABBFD8826E178E0F9DC5
```

The KernelSU module was built from KernelSU v3.2.5 (`b0bc817`) with the
Samsung KDP/RKP/DEFEX patch against the Android 14 6.1 DDK image, with the
DDK release replaced by the exact DZE1 release before building. `check_symbol`
passes against the recovered DZE1 `vmlinux.elf`.

## Status

Offline porting gates complete. The profile has not been executed on an
`SM-S928N` device; hardware validation (in particular the inverse-slide flag)
remains outstanding.
