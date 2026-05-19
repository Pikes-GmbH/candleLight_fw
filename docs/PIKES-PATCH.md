# Pikes: Host-Reboot / USB-Zustand (FYSETC UCAN)

Patch für [candleLight_fw issue #46](https://github.com/candle-usb/candleLight_fw/issues/46): Adapter hängt nach Host- oder Container-Reboot ohne Power-Cycle (z. B. Proxmox USB-Passthrough).

## Branch

`fix/usb-host-reboot-state`

## Änderungen

1. **`to_host_buf` zurücksetzen** bei USB-Suspend, Resume und DeInit (Bus-Reset) – Ersatz für das frühere `TxState = 0` aus [PR #94](https://github.com/candle-usb/candleLight_fw/pull/94).
2. **SOF-Watchdog** (200 ms ohne USB Start-of-Frame): CAN wird abgeschaltet, wenn der Host verschwindet **ohne** USB-Suspend (typisch bei Reboot ohne Stecker ziehen).

## Bauen

```bash
mkdir -p build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/arm-none-eabi-gcc.cmake
make FYSETC_UCAN_fw
```

Firmware: `build/FYSETC_UCAN_fw.bin` (exakter Pfad je nach CMake-Target).

## Flashen (DFU)

Siehe [README.md](../README.md) – Fail-Safe-Methode mit BOOT-Pins oder `make flash-FYSETC_UCAN_fw`.

## Test

- Host oder LXC rebooten (ohne `ip link set can0 down`).
- Nach Boot: `ip link set can0 up type can bitrate <rate>` und CAN-Traffic prüfen.

## Fork auf GitHub ([Pikes-GmbH](https://github.com/Pikes-GmbH))

1. Als Org-Mitglied einloggen (Account mit Rechten auf `Pikes-GmbH`, z. B. `EKR-Pikes`).
2. [candle-usb/candleLight_fw](https://github.com/candle-usb/candleLight_fw) → **Fork** → Owner **Pikes-GmbH** wählen.
3. Lokal pushen:

```bash
git remote add pikes https://github.com/Pikes-GmbH/candleLight_fw.git
git push -u pikes fix/usb-host-reboot-state
```

Kollegen: Branch klonen oder `git fetch` + `git checkout fix/usb-host-reboot-state`.

Alternativ nur den Patch anwenden: `git am patches/0001-*.patch` (Ordner `patches/`).

## Upstream

Geplanter Pull Request von `Pikes-GmbH/candleLight_fw:fix/usb-host-reboot-state` nach `candle-usb/candleLight_fw`.
