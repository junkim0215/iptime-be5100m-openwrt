# OpenWrt for ipTIME BE5100M

[English](README.md) | **한국어**

ipTIME BE5100M(Airoha AN7563PT, 512 MiB RAM, 128 MiB SPI-NAND, MT7992 Wi-Fi, EN8811H 2.5G WAN)에 OpenWrt를 설치하는 비공식 포팅 펌웨어입니다. 제조사 U-Boot를 그대로 쓰며 부트로더를 건드리지 않습니다.

> **주의:** 비공식 펌웨어입니다. 설치하면 제조사 보증이 사라질 수 있고, 플래시 중 문제가 생기면 UART 연결이 필요할 수 있습니다. 모든 책임은 사용자 본인에게 있습니다.

## 상태

| 기능 | 상태 |
|---|---|
| LAN 4포트, WAN(DHCP) | 동작 (WAN은 100 Mbps/1 Gbps 환경에서 확인) |
| Wi-Fi 2.4 / 5 GHz | 동작 (KR) |
| LED(CPU, 2.4G, 5G, WAN, LAN) | 동작 |
| USB 3.0 저장장치 (exFAT, vfat, ext4, ntfs3) | 동작 |
| LuCI, sysupgrade(설정 유지) | 동작 |
| 2.5G WAN 협상, USB 2.0 장치, 장시간 안정성, 처리량 | **미확인** |
| critical 온도 트립 | 없음(데이터시트 없음) |

한 대의 기기, 제조사 펌웨어 16.006에서만 확인했습니다.

## 다운로드

[Releases](../../releases)에서 받습니다.

| 파일 | 용도 |
|---|---|
| `openwrt-airoha-an7563-iptime_be5100m-factory_16_006.bin` (r113) | **제조사 웹 UI로 처음 설치할 때만** 사용 |
| `*-sysupgrade.bin` (r115 이후) | OpenWrt 안에서 업그레이드 |
| `*-slot.bin` | UART + TFTP 설치/복구용 (고급) |

## 설치 (UART 없이, 제조사 웹 UI)

1. 제조사 펌웨어 관리 페이지(`192.168.0.1`) → 관리도구 → 시스템 관리 → 펌웨어 업그레이드.
2. `openwrt-airoha-an7563-iptime_be5100m-factory_16_006.bin`을 올립니다. 파일 이름에 `_16_006`이 들어 있어야 합니다.
3. 전원을 끄지 말고 기다립니다. 자동 재부팅 후 `192.168.1.1`로 OpenWrt가 올라옵니다. root 비밀번호를 설정하세요.
4. **5분 안에 OpenWrt가 안 열리고 제조사 펌웨어가 다시 올라오면**(중간에 한 번 스스로 재부팅됩니다) 제조사 펌웨어가 두 번째 슬롯에서 돌고 있었던 것입니다. 제조사 펌웨어를 웹 UI로 **한 번 더** 올리면 제조사 펌웨어가 메인 슬롯으로 돌아오니, 그 뒤 위 1~3을 다시 합니다.

> 웹 설치에는 **r113 파일만** 쓰세요. 이후 버전의 factory 이미지는 웹 설치 시 두 번째 부팅에서 제조사 펌웨어로 되돌아갑니다(Stock U-Boot의 `boot_try` 동작). r113은 이를 막는 수정이 들어 있습니다.

## 업그레이드

OpenWrt 안에서 LuCI → 시스템 → 백업/펌웨어 업그레이드로 `sysupgrade.bin`을 올립니다(설정 유지 가능). `-F`(강제)는 쓰지 마세요.

## 제조사 펌웨어로 되돌리기 (UART 없이)

OpenWrt에서 아래를 실행하면 부팅 시 U-Boot가 제조사 펌웨어로 전환합니다. 제조사 펌웨어가 메인 슬롯에 남아 있어야 합니다(정상 설치 상태). 설정 백업을 먼저 받으세요.

```
mtd erase kernel
reboot
```

## 알려진 제한 사항

- 패키지 저장소는 공식 `base`, `luci`, `packages`, `routing`, `telephony`, `video`만 남깁니다. 이미지에 없는 커널 모듈은 `apk add kmod-…`로 설치할 수 없습니다.
- 슬롯 구조와 부트 동작의 자세한 설명은 [`docs/install-format-and-procedure.ko.md`](docs/install-format-and-procedure.ko.md) (한국어, 영문판 예정)를 보세요.

## 소스와 빌드

소스는 [junkim0215/openwrt](https://github.com/junkim0215/openwrt)의 `be5100m` 브랜치입니다. 릴리스마다 빌드한 커밋에 태그가 있습니다.

| 릴리스 | 태그 | 커밋 |
|---|---|---|
| r113 (웹 설치용) | `be5100m-r113` | `b6e0f16590` |
| r115 (sysupgrade) | `be5100m-r115` | `de8de5506c` |

```
git clone https://github.com/junkim0215/openwrt.git && cd openwrt
git checkout be5100m-r115
./scripts/feeds update -a && ./scripts/feeds install -a
cp <diffconfig> .config && make defconfig && make -j$(nproc)
```

`diffconfig`와 `feeds.conf.default`는 릴리스에 첨부되어 있습니다. r113의 설정은 r115 diffconfig에 `CONFIG_PACKAGE_uboot-envtools=y`를 더한 것입니다.

## 크레딧

- **xinyooo**: AN7563 포팅과 [openwrt/openwrt#23960](https://github.com/openwrt/openwrt/pull/23960)
- **pabragin**: Xiaomi BE5000용 OpenWrt 빌드 시리즈([xiaomi-be5000-openwrt](https://github.com/pabragin/xiaomi-be5000-openwrt)). 이 저장소는 그 릴리스 `be5000-openwrt-20261105` 위에 BE5100M 지원을 얹은 것입니다.
- OpenWrt 프로젝트

이 포팅은 Claude(Anthropic)의 도움을 받아 작성했고, 변경 사항은 실기에서 시험했습니다.

라이선스는 GPL-2.0입니다.
