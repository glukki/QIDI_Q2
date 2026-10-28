# AGENTS.md

Personal fork of QIDI's firmware/config repo for the **QIDI Q2** 3D printer (Klipper-based). The work here is tuning one physical printer, so the "product" is the config in `config/`, and changes are verified on the printer itself. The repo has no build, lint or test suite.

## Repository layout

- `config/` — Klipper/Moonraker config. Mirrors `~/printer_data/config` on the printer. **This is where almost all changes happen.**
- `klipper/klippy/` — QIDI's modified Klipper (Python host side only). Reference for what config options QIDI-specific modules accept.
- `moonraker/moonraker/` — QIDI's modified Moonraker.
- `fluidd/` — prebuilt Fluidd web UI bundle.
- `other/` — README images.

On the printer, `~/klipper`, `~/moonraker`, `~/fluidd` are plain copies (no `.git`); QIDI ships updates as USB packages, see `README.md`.

### Config structure

`config/printer.cfg` is the entry point. It `[include]`s:

- `KAMP_Settings.cfg` — Klipper Adaptive Meshing & Purging settings.
- `gcode_macro.cfg` — QIDI's macros: `PRINT_START`, `PRINT_END`, `CANCEL_PRINT`, `PAUSE`/`RESUME`, homing (`_HOME_X/Y/XY`), nozzle cleaning (`CLEAR_NOZZLE`, `CUT_FILAMENT_1`), and many M-code overrides (`M106`, `M109`, `M141`, `M191`, `M603`/`M604`, `M4029`–`M4031`, …) that the touchscreen firmware calls.
- `MCU_ID.cfg` — main MCU serial path; unique per board, exists only on the printer.
- `timelapse.cfg` — symlink on the printer to `~/moonraker-timelapse/klipper_macro/timelapse.cfg`.
- `plr.cfg` — power-loss recovery (shell commands + macros).
- `box.cfg` — includes `box1..4.cfg` for the QIDI multi-material Box (all commented out = no Box attached). `box*.cfg` define `T0`–`T15` / `UNLOAD_T*`.

Other config files: `moonraker.conf`, `drying.conf` (Box filament drying presets; CRLF line endings, keep them), `officiall_filas_list.cfg` (filament list for the touchscreen).

QIDI-specific Klipper modules referenced by the config live in `klipper/klippy/extras/`: `probe_air.py` (`[probe_air]`, the bed probe), `motor_constants.py`, `duplicate_pin_override.py`, `lis2dw.py` (accelerometer), `aht10.py`, `box_heater_fan.py`. The Box modules (`box_detect`, `box_extras`, `box_rfid`, `box_stepper`, `aht20_f`) are shipped only as compiled `.so` files — no source.

Comments in QIDI's files are often in Chinese.

### SAVE_CONFIG block

The `#*#` section at the bottom of `printer.cfg` is written by Klipper's `SAVE_CONFIG` (bed meshes, `[probe_air] z_offset`, extruder PID, input shaper). A value saved there must not also be set in the main body of the file, so calibrated options get commented out above (e.g. PID, `z_offset` on the printer). Treat the `#*#` block as printer-owned data.

## Printer access

- Local secrets live in git-ignored `.env` (`KEY=value` lines). `PRINTER` holds the SSH target (`user@address`). Keep the printer address and other secrets out of committed files; reference `$PRINTER` instead.
- `source .env && ssh "$PRINTER"` — key auth works non-interactively.
- `sudo` needs a password (QIDI default `makerbase`) and a TTY; ask the user to run sudo commands via `ssh -t "$PRINTER" '...'`.
- OS: Debian 11 (bullseye), aarch64. apt sources point to `archive.debian.org`. QIDI holds `xserver-common`, `xserver-xorg-core`, `xserver-xorg-legacy` — keep them held; leave `apt autoremove` suggestions alone.
- Services: `klipper`, `moonraker`, `crowsnest`, `klipper-mcu`.
- Logs: `~/printer_data/logs/` (`klippy.log`, `moonraker.log`).
- Other notable dirs in `/home/mks`: `printer_data`, `Klipper-Adaptive-Meshing-Purging` (target of the `config/KAMP` symlink), `moonraker-timelapse`, `crowsnest`, `kiauh`, `fluidd-config` (target of `config/fluidd.cfg` symlink).

Confirm with the user before running anything that changes the printer.

### Printer-only files in `~/printer_data/config`

Not in the repo: `MCU_ID.cfg`, `crowsnest.conf`, `Adaptive_Mesh.cfg`, `saved_variables.cfg` (rewritten at runtime by `[save_variables]`), `shapers/` (input shaper graphs), `*.bak` and timestamped `printer-*.cfg` backups, and the symlinks `KAMP`, `fluidd.cfg`, `timelapse.cfg`.

Before committing printer-side edits, diff the printer's config against `config/`:

```sh
rsync -a "$PRINTER":printer_data/config/ <scratch dir>/
diff -rq --no-dereference config <scratch dir>
```

## Applying and verifying a change

1. Edit the file under `config/`.
2. Copy it to `~/printer_data/config/` on the printer (or edit via Fluidd).
3. Restart Klipper (`RESTART` / `FIRMWARE_RESTART` from the Fluidd console, or `systemctl restart klipper`).
4. Check `~/printer_data/logs/klippy.log` for config errors and confirm behavior on the printer.

Input shaper calibration procedure: `config/README_input_shaper.md`.

## Git workflow

- Remotes: `origin` = `git@github.com:glukki/QIDI_Q2.git` (user's fork), `upstream` = `git@github.com:QIDITECH/QIDI_Q2.git`.
- `main` tracks upstream. `my_config` is the deployed config.
- One feature branch per tweak (e.g. `turn_off_motherboard_fan_when_idle`, `fix_input_shaper`), branched from `main`, one focused commit with a short imperative subject, then merged into `my_config` with a merge commit (`Merge branch '<name>' into my_config`). Keeping tweaks on separate branches lets each be rebased onto new upstream releases independently.
- `enable_adaptive_meshing` is an unmerged branch that vendors KAMP via `git subtree` at `Klipper-Adaptive-Meshing-Purging/`.
