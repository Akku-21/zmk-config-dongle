# ZMK Build Reference for zmk-config-dongle

Last updated: 2026-06-22

## Architecture

- **ZMK**: `main` branch (Zephyr 4.1)
- **Workflow**: Custom `build.yml` (NOT the ZMK reusable workflow) -- patches Zephyr's kconfig to suppress upstream Kconfig warnings
- **Board approach**: `nice_nano//zmk` as board, `eyelash_corne_left/right` as shields (NOT standalone boards)
- **Dongle**: `xiao_ble//zmk` board + `corne_dongle_xiao prospector_adapter` shield
- **eyelash_corne module**: Upstream `a741725193/zmk-new_corne` provides the board/shield definitions. Local `boards/shields/eyelash_corne/` overrides the upstream's overlay files to remove `cormoran`-specific includes
- **Display**: Prospector LCD on dongle, nice_view_gem on peripherals
- **Modules**: urob (adaptive-key, auto-layer, helpers, leader-key, tri-state), nice-view-gem, prospector-zmk-module

## Key Files

| File | Purpose |
|---|---|
| `.github/workflows/build.yml` | Custom workflow with kconfig patch -- DO NOT replace with ZMK reusable workflow |
| `config/west.yml` | Module manifest -- ZMK main, upstream eyelash_corne, urob modules, prospector main |
| `build.yaml` | Build matrix -- board/shield combinations |
| `config/eyelash_corne.conf` | User config applied to peripheral builds |
| `config/eyelash_corne.keymap` | Keymap for the keyboard |
| `boards/shields/eyelash_corne/` | Local shield overrides (overlays, dtsi, layouts) |
| `boards/shields/eyelash_corne/corne_dongle_xiao.conf` | Dongle-specific config (Prospector display settings) |

## Why the Custom Workflow

The custom `build.yml` patches two Zephyr files before building:
1. `zephyr/scripts/kconfig/kconfig.py` line 127: `err("Aborting due to Kconfig warnings")` -> `pass`
2. `zephyr/cmake/modules/kconfig.cmake`: all `FATAL_ERROR` -> `WARNING`, remove `COMMAND_ERROR_IS_FATAL`

This is needed because Zephyr 4.1 has upstream Kconfig dependency bugs (BT_USER_PHY_UPDATE, BT_AUTO_PHY_UPDATE, NRFX_NVMC) that cause warnings treated as fatal errors for out-of-tree nRF52840 boards. ZMK's in-tree boards avoid this via the `//zmk` variant mechanism. When ZMK v0.4 releases, these may be fixed and the patches can potentially be removed.

## Deprecated Symbols Removed

These were in the original config but are incompatible with Zephyr 4.1:
- `CONFIG_WS2812_STRIP` (auto-enabled now)
- `CONFIG_EC11`, `CONFIG_EC11_TRIGGER_GLOBAL_THREAD` (moved to defconfig)
- `CONFIG_BT_CTLR_TX_PWR_PLUS_8` (choice symbol conflict)
- `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN` (triggers BT PHY warnings)
- `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING` (renamed/removed)
- `CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR` (removed from prospector module, replaced by `CONFIG_PROSPECTOR_BRIGHTNESS_FIXED`)

## Updating

To rebuild after keymap or config changes:
- Just push to `main` -- GitHub Actions triggers automatically
- Download `firmware` artifact from Actions tab

To update ZMK or modules:
- Modules are pinned to `main` branches -- they update automatically on each build
- If a module update breaks the build, pin it to a specific commit in `west.yml`

## Flash Order

1. Flash `settings_reset` on all devices first (if pairing issues)
2. Flash dongle (XIAO BLE): double-tap reset, drag `xiao_corne_dongle.uf2`
3. Flash left half: double-tap reset, drag left `.uf2`
4. Flash right half: double-tap reset, drag right `.uf2`
5. Power on: dongle first (USB), then left, then right

## Fork at Akku-21/zmk-new_corne

The fork was migrated to HWMv2 format but is **NOT used** in the current build. The upstream (`a741725193`) is used instead. The fork can be ignored unless the upstream eyelash_corne module needs custom changes in the future.
