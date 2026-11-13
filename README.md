# OpenWrt for ipTIME BE5100M

**English** | [한국어](README.ko.md)

An unofficial OpenWrt port for the ipTIME BE5100M (Airoha AN7563PT, 512 MiB RAM, 128 MiB SPI-NAND, MT7992 Wi-Fi, EN8811H 2.5G WAN). It keeps the stock U-Boot and never touches the bootloader.

> [!WARNING]
> This is unofficial firmware. Installing it may void your warranty, and if a flash goes wrong you may need a UART connection to recover. Use it at your own risk.

## Status

| Feature | Status |
|---|---|
| 4 LAN ports, WAN (DHCP) | Working (WAN verified on a 100 Mbps link only) |
| Wi-Fi 2.4 / 5 GHz | Working (KR regulatory domain) |
| LEDs (CPU, 2.4G, 5G, WAN, LAN) | Working |
| USB 3.0 storage (exFAT, vfat, ext4, ntfs3) | Working |
| LuCI, sysupgrade (keeping settings) | Working |
| 2.5G WAN negotiation | **Untested**, but 1G working |
| USB 2.0 devices, long-term stability, throughput | **Untested** |
| Critical temperature trip | None defined (no datasheet) |

Tested on a single unit running stock firmware 16.006 only.

## Downloads

Get the files from [Releases](../../releases).

| File | Use |
|---|---|
| `…-squashfs-factory_16_006.bin` | install from the stock web UI |
| `…-squashfs-sysupgrade.bin` | upgrade from OpenWrt (keep settings) |
| `…-squashfs-slot.bin` | U-Boot TFTP install (UART) |

## Installing (no UART, via the stock web UI)

1. Open the stock admin page (`192.168.0.1`) → 관리도구 (Management) → 시스템 관리 (System) → 펌웨어 업그레이드 (Firmware upgrade).
2. Upload `openwrt-airoha-an7563-iptime_be5100m-squashfs-factory_16_006.bin`. The file name must contain `_16_006`.
3. Do not power off; the router reboots by itself. Check whether LuCI comes up at `http://192.168.1.1` (user `root`, no password).
   - If LuCI comes up, set a root password.
   - **If OpenWrt (LuCI) does not come up within 5 minutes and the stock firmware returns** (the router reboots once by itself in between), the stock firmware was running from the second slot. Upload the official stock firmware (from the [ipTIME download page](https://iptime.com/support/downloads?dfsid=21&dftid=903)) through the web UI, which brings it back to the main slot, then repeat steps 1–3.

## Installing or recovering with UART (advanced)

Use this if the web install is not possible or OpenWrt no longer boots. You need a 3.3 V USB-UART adapter (connect GND, TX and RX only; never VCC; the UART pin header is populated and labeled on the board), a TFTP server on a PC at `192.168.0.100`, and the `*-slot.bin` image.

1. Connect the UART (115200 8N1), power on and press any key at `Hit any key to stop autoboot` to get the U-Boot `ECNT>` prompt (the countdown is short).
2. Set the PC to a static IP `192.168.0.100/24`, connect it to a LAN port (e.g. LAN1) and start the TFTP server (on Windows, allow TFTP in the firewall). Then run (replace `<len>` with the file size rounded up to a multiple of `0x20000`):

```
tftpboot 0x88000000 openwrt-airoha-an7563-iptime_be5100m-squashfs-slot.bin
crc32 0x88000000 <len>
flash erase 0x2D00000 <len>
flash write 0x2D00000 <len> 0x88000000
flash read 0x2D00000 <len> 0x89000000
crc32 0x89000000 <len>
flash erase <0x2D00000+len> <0x2800000-len>
bootflag read
reset
```

3. Both `crc32` values must match the CRC of the file. If `bootflag read` shows `0`, run `bootflag swap` before `reset` so the second slot (OpenWrt) boots.

> [!CAUTION]
> Write only to `0x2D00000`–`0x54FFFFF` (the second slot). Do not run `saveenv` and do not write to any other address. To boot the stock firmware again from U-Boot, run `bootflag swap` so that the flag is `0`.

## Upgrading

From inside OpenWrt, use LuCI → System → Backup / Flash Firmware and upload a `sysupgrade.bin` (keeping settings is fine). Do not use `-F` (force).

## Returning to stock (no UART, r132 or later)

Run the following in OpenWrt; U-Boot then switches to the stock firmware on the next boot. The stock firmware must still be in the main slot (the normal installed state). Back up your settings first.

```
fw_setenv boot_try 0 && reboot
```

## Known limitations

- The stock reset-button TFTP recovery mode has not been tested. ipTIME Firmware Wizard does not support BE models.
- The reboot after `sysupgrade` sometimes hangs. If the router does not come back after a few minutes, power-cycle it. The same problem is reported for the Xiaomi BE5000, which this build is based on.
- Heavy CPU-routed traffic may jam the Ethernet frame engine. This bug was found on the Xiaomi BE5000, which this build is based on, so it may affect this router too. It has not been tested here.
- Only the official `base`, `luci`, `packages`, `routing`, `telephony` and `video` package feeds are kept. Kernel modules that are not in the image cannot be installed with `apk add kmod-…`.

## Source and building

The source is the `be5100m` branch of junkim0215/openwrt. Each release is tagged
at the commit it was built from (`be5100m-rNNN`); the tag and commit are listed
in the release notes.

```
git clone https://github.com/junkim0215/openwrt.git && cd openwrt
git checkout be5100m-rNNN    # e.g. be5100m-r132
./scripts/feeds update -a && ./scripts/feeds install -a
cp <diffconfig> .config && make defconfig && make -j$(nproc)
```

## Credits

- **xinyooo**: the AN7563 port and [openwrt/openwrt#23960](https://github.com/openwrt/openwrt/pull/23960)
- **pabragin**: the OpenWrt build series for the Xiaomi BE5000 ([xiaomi-be5000-openwrt](https://github.com/pabragin/xiaomi-be5000-openwrt)). This repository adds BE5100M support on top of the release `be5000-openwrt-20261109`.
- The OpenWrt project

This port was written with the help of Claude (Anthropic), and the changes were tested on real hardware.

Licensed under GPL-2.0.

The image also contains binary firmware for Wi-Fi and the NPU. See the license information of the corresponding OpenWrt packages.
