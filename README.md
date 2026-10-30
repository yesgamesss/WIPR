# Wipr

**A tiny disk tool that boots straight from a USB stick. No Linux, no Windows.**
Wipe, clone, partition and check drives from a colorful terminal. Made by **Rainbow Tech**.

> ## ⚠️ WARNING
> Wipr can **permanently erase drives**. `wipe`, `clone`, `mbr`, `gpt` and `fakecheck`
> destroy data and **cannot be undone**. Always check the drive number with `disks`
> before you type `YES`. You use this at your own risk.
>
> Wipr is young software. It has been tested in QEMU, and real-hardware testing is
> still in progress. Try it on drives you don't care about first.

## Features

- Own UEFI program (written in Rust), about 100 KB. Boots in seconds, runs from RAM
- 64-bit **and** 32-bit UEFI in one image
- Colored terminal, a progress bar with speed and ETA, scrollback (Up / Down / PgUp / PgDn)
- Full-screen help menu (type `help` or press **F1**)
- Refuses to touch the disk it booted from, and asks you to type `YES` before anything destructive

| Command | What it does |
|---|---|
| `disks` | List drives: size, type, MBR / GPT |
| `rescan` | Find USB drives plugged in after boot |
| `disk N` (or `desk N`) | Full report on drive N: details + partitions |
| `parts N` | List the partitions on drive N |
| `health` | SMART health of SATA / IDE / NVMe drives, and USB adapters that allow it (HDD and SSD) |
| `wipe N` | Zero the whole drive N |
| `wipe N quick` | Clear the partition tables only (seconds) |
| `check N` | Verify drive N is all zeros |
| `clone A B C-D` | Copy drive A onto B, C..D (up to 12 targets at once) |
| `verify A B` | Compare two drives |
| `fakecheck N` | Test a USB stick for fake capacity (erases it) |
| `mbr N` / `gpt N` | New empty partition table (erases) |
| `clear` `reboot` `off` | Screen / restart / shut down |

### Drive health

- **HDD:** green **HEALTHY**, yellow **CAUTION**, red **BAD** (probably dying, back up now)
- **SSD / NVMe:** a wear percentage and the same colors

## Supported drives

| Drive | List / wipe / clone | Health |
|---|---|---|
| SATA / IDE (internal) | yes | yes |
| NVMe (internal) | yes | yes |
| USB sticks and USB drives | yes | no |
| USB-to-SATA adapters | yes | yes, if the adapter passes ATA commands through (experimental) |
| USB-to-NVMe adapters | yes | no |

USB health is experimental: Wipr sends standard ATA commands (SAT) to the adapter.
Many adapters (JMicron, ASMedia) allow this and some block it; Wipr tells you when
an adapter blocks it. USB sticks have no health data. The drive must be visible to
your firmware (very old firmware may not see NVMe drives).

## Requirements

- A PC with **UEFI** firmware (most PCs from about 2013 on; many older ones in UEFI mode)
- **Secure Boot turned off** (Wipr is not signed)
- **Legacy BIOS-only PCs are not supported.** Wipr is a UEFI program.

## Starting Wipr

1. Write `wipr.img` to a USB stick with [Rufus](https://rufus.ie) (choose **DD Image mode**)
   or [balenaEtcher](https://etcher.balena.io). **This erases the stick.**
2. Turn Secure Boot off in the BIOS.
3. Boot the stick from the boot menu (F12 / F10 / Esc).
4. Wipr starts on its own.

### Landed in a UEFI shell instead?

Run it by hand:

```
fs0:
cd EFI\BOOT
BOOTX64.EFI
```

If `fs0:` isn't the right drive, type `map` to list the drives and try `fs1:`, `fs2:`
and so on. Use `ls` to check you can see the `EFI` folder.
On 32-bit firmware, run `BOOTIA32.EFI` instead.

## Building it yourself

Needs [Rust](https://rustup.rs) and the two UEFI targets:

```
rustup target add x86_64-unknown-uefi i686-unknown-uefi
cargo build --release
cargo build --release --target i686-unknown-uefi
```

Test in QEMU (Windows paths shown):

```
mkdir esp\EFI\BOOT
copy target\x86_64-unknown-uefi\release\wipr.efi esp\EFI\BOOT\BOOTX64.EFI
qemu-system-x86_64 -machine q35 -drive if=pflash,format=raw,readonly=on,file=edk2-x86_64-code.fd -drive format=raw,file=fat:rw:esp
```

`make-img.ps1` (Windows) builds the 64 MB `wipr.img` from the two release builds.

## Credits

Made by **Rainbow Tech**. Built with the [uefi](https://crates.io/crates/uefi) Rust crate.

## License

MIT. See [LICENSE](LICENSE).
