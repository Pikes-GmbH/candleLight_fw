# Pikes patch: host reboot / USB state (FYSETC UCAN)

Fix for [candleLight_fw issue #46](https://github.com/candle-usb/candleLight_fw/issues/46): the adapter stops responding after a host or container reboot without a power cycle (e.g. Proxmox USB passthrough to LXC).

**Base:** current `candle-usb/candleLight_fw` `master` (not the legacy [smalik bugfix branch](https://github.com/smalik007/candleLight_fw/tree/bugfix%2346/fix-can-hang-on-host-reboot)).

## Branch

`fix/usb-host-reboot-state` on [Pikes-GmbH/candleLight_fw](https://github.com/Pikes-GmbH/candleLight_fw).

## What changed

| Area | Change |
|------|--------|
| `src/usbd_gs_can.c` | New `USBD_GS_CAN_ReleaseToHostBuf()` — returns a stuck `to_host_buf` to the frame pool (replaces legacy `TxState = 0` from [PR #94](https://github.com/candle-usb/candleLight_fw/pull/94)). |
| Suspend / Resume / DeInit | Call `ReleaseToHostBuf` on USB suspend, resume, and bus reset (`DeInit`). |
| SOF watchdog | If no USB Start-of-Frame for **200 ms** while configured and not suspended, disable CAN and release `to_host_buf` (covers host reboot without USB suspend). |
| `src/main.c` | Call `USBD_GS_CAN_CheckHostPresence()` in the main loop. |
| `include/usbd_gs_can.h` | Declare `USBD_GS_CAN_CheckHostPresence()`. |

## Build

```bash
mkdir -p build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/arm-none-eabi-gcc.cmake
make FYSETC_UCAN_fw
```

Output firmware: `build/FYSETC_UCAN_fw.bin` (exact path may vary with your CMake target name).

Requires `arm-none-eabi-gcc` with **newlib** — see the [main README](../README.md).

## Flash (DFU)

See [README.md](../README.md) — fail-safe method using BOOT pins, or:

```bash
make flash-FYSETC_UCAN_fw
```

## Test

1. Flash `FYSETC_UCAN_fw.bin` to the adapter.
2. Reboot the **host** or **LXC container** **without** running `ip link set can0 down` first.
3. After boot: `ip link set can0 up type can bitrate <rate>` and verify CAN traffic.

**Pass:** `can0` works after reboot without unplugging the adapter.

## Clone for colleagues

```bash
git clone https://github.com/Pikes-GmbH/candleLight_fw.git
cd candleLight_fw
git checkout fix/usb-host-reboot-state
```

Apply only the patch file:

```bash
git am patches/0001-*.patch
```

## Upstream

Planned pull request: `Pikes-GmbH/candleLight_fw:fix/usb-host-reboot-state` → `candle-usb/candleLight_fw`.

## Host workaround (without reflashing)

```bash
ip link set can0 down   # before reboot
```
