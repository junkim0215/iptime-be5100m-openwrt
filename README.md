# OpenWrt for ipTIME BE5100M

**English** | [한국어](README.ko.md)

An unofficial OpenWrt port for the ipTIME BE5100M (Airoha AN7563PT, 512 MiB RAM, 128 MiB SPI-NAND, MT7992 Wi-Fi, EN8811H 2.5G WAN). It keeps the stock U-Boot and never touches the bootloader.

> **Warning:** This is unofficial firmware. Installing it may void your warranty, and if a flash goes wrong you may need a UART connection to recover. Use it at your own risk.

## Status

| Feature | Status |
|---|---|
| 4 LAN ports, WAN (DHCP) | Working (WAN verified on 100 Mbps / 1 Gbps links) |
| Wi-Fi 2.4 / 5 GHz | Working (KR regulatory domain) |
| LEDs (CPU, 2.4G, 5G, WAN, LAN) | Working |
| USB 3.0 storage (exFAT, vfat, ext4, ntfs3) | Working |
| LuCI, sysupgrade (keeping settings) | Working |
| 2.5G WAN negotiation, USB 2.0 devices, long-term stability, throughput | **Untested** |
| Critical temperature trip | None defined (no datasheet) |

Tested on a single unit running stock firmware 16.006 only.

## Downloads

Get the files from [Releases](../../releases).

| File | Purpose |
|---|---|
| `openwrt-airoha-an7563-iptime_be5100m-factory_16_006.bin` (r113) | **First install through the stock web UI only** |
| `*-sysupgrade.bin` (r115 and later) | Upgrading from inside OpenWrt |
| `*-slot.bin` | UART + TFTP install/recovery (advanced) |

## Installing (no UART, via the stock web UI)

1. Open the stock admin page (`192.168.0.1`) → 관리도구 (Management) → 시스템 관리 (System) → 펌웨어 업그레이드 (Firmware upgrade).
2. Upload `openwrt-airoha-an7563-iptime_be5100m-factory_16_006.bin`. The file name must contain `_16_006`.
3. Do not power off; the router reboots by itself. OpenWrt comes up at `192.168.1.1`. Set a root password.
4. **If OpenWrt does not come up within 5 minutes and the stock firmware returns** (the router reboots once by itself in between), the stock firmware was running from the second slot. Upload the stock firmware through the web UI **once more**, which brings it back to the main slot, then repeat steps 1–3.

> Use **only the r113 file** for web installs. Factory images from later builds fall back to the stock firmware on the second boot when installed through the web UI (a stock U-Boot `boot_try` behavior). Only r113 contains the fix for this.

## Upgrading

From inside OpenWrt, use LuCI → System → Backup / Flash Firmware and upload a `sysupgrade.bin` (keeping settings is fine). Do not use `-F` (force).

## Returning to stock (no UART)

Run the following in OpenWrt; U-Boot then switches to the stock firmware on the next boot. The stock firmware must still be in the main slot (the normal installed state). Back up your settings first.

```
mtd erase kernel
reboot
```

## Known limitations

- Only the official `base`, `luci`, `packages`, `routing`, `telephony` and `video` package feeds are kept. Kernel modules that are not in the image cannot be installed with `apk add kmod-…`.
- For a detailed description of the slot layout and boot behavior, see [`docs/install-format-and-procedure.ko.md`](docs/install-format-and-procedure.ko.md) (Korean; an English version is planned).

## Source and building

The source is the `be5100m` branch of [junkim0215/openwrt](https://github.com/junkim0215/openwrt). Each release is tagged at the commit it was built from.

| Release | Tag | Commit |
|---|---|---|
| r113 (web install) | `be5100m-r113` | `b6e0f16590` |
| r115 (sysupgrade) | `be5100m-r115` | `de8de5506c` |

```
git clone https://github.com/junkim0215/openwrt.git && cd openwrt
git checkout be5100m-r115
./scripts/feeds update -a && ./scripts/feeds install -a
cp <diffconfig> .config && make defconfig && make -j$(nproc)
```

`diffconfig` and `feeds.conf.default` are attached to the release. The r113 configuration is the r115 diffconfig plus `CONFIG_PACKAGE_uboot-envtools=y`.

## Credits

- **xinyooo**: the AN7563 port and [openwrt/openwrt#23960](https://github.com/openwrt/openwrt/pull/23960)
- **pabragin**: the OpenWrt build series for the Xiaomi BE5000 ([xiaomi-be5000-openwrt](https://github.com/pabragin/xiaomi-be5000-openwrt)). This repository adds BE5100M support on top of the release `be5000-openwrt-20261105`.
- The OpenWrt project

This port was written with the help of Claude (Anthropic), and the changes were tested on real hardware.

Licensed under GPL-2.0.
