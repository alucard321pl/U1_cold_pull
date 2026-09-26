# Snapmaker U1 Cold Pull

A guided Klipper cold pull macro for Snapmaker U1, adapted from the `MMU_COLD_PULL` implementation distributed with Happy Hare. Happy Hare is not required.

**Status: tested on a physical Snapmaker U1.** I tested this macro on my Snapmaker U1 on September 26, 2026, and it worked very well. Templates, command parsing and validation checks have also been tested locally. This is an unofficial community adaptation.

Heating, purging, topping up the nozzle during cooling, and reheating are automated. **The final pull is manual, while pressing and holding the extruder release lever.** This version does not perform Happy Hare's motor-assisted 150 mm retraction and has no `PULL_SPEED` parameter.

## Installation

First, enable **root access** in the settings menu on the printer's touchscreen. Then, on a device connected to the same local network, enter the printer's IP address in a web browser to open **Fluidd**. Use Fluidd to edit the Klipper configuration and access the console.

**This macro is available through Fluidd, not through the printer's touchscreen menu.** Run the commands below in the Fluidd console and follow the messages there.

1. Back up your configuration.
2. Place [`u1_cold_pull.cfg`](u1_cold_pull.cfg) alongside your active `printer.cfg`.
3. Add the following to `printer.cfg`:

   ```ini
   [include u1_cold_pull.cfg]
   ```

4. Run `RESTART` while the printer is not printing.

No `[respond]` or `[force_move]` section is required. Keep Klipper's configured `min_extrude_temp` protection enabled. Do not replace your configuration with the example configuration from the U1 source repository.

## Preparation and use

1. Finish or cancel any print. Home the printer and select the desired tool using the normal U1 procedure. It must be mounted on the carriage, not parked. `T0` corresponds to `extruder`; `T1`–`T3` correspond to `extruder1`–`extruder3`.
2. Position the tool somewhere accessible, at least 20 mm above the bed, with clearance for purged filament. The macro does not move XYZ or change tools.
3. Disconnect the filament guide tube at the toolhead. Prepare approximately 250–300 mm of accessible filament and load it using the normal heated loading procedure. Leave the extruder release lever unpressed during automated extrusion. Do not force filament into a cold nozzle.
4. Run the command matching your cleaning filament, for example:

   ```gcode
   SM_COLD_PULL MATERIAL=PLA
   ```

5. Watch the console. At `No more motor moves...`, press and hold the extruder release lever, leaving the filament in place. There are no further extruder motor moves after this message.
6. At `PULL NOW`, keep the lever pressed and pull the filament steadily upward by hand. The heater is switched off at this point. Inspect the tip for an impression of the nozzle interior and removed debris.
7. Let go of the lever and reconnect the guide tube afterward. Repeat with fresh filament if needed.

Do not start a print, change tools, or run other printer operations during the procedure. Run the macro and read its progress messages in Fluidd. The macro is not available on the printer's touchscreen.

## Material profiles

Select the material of the filament used for cleaning, rather than the residue being removed. PLA is the default. These are alternative commands; run only one:

```gcode
SM_COLD_PULL MATERIAL=PLA
SM_COLD_PULL MATERIAL=PETG
SM_COLD_PULL MATERIAL=ABS
SM_COLD_PULL MATERIAL=NYLON
```

| Material | HOT_TEMP | COLD_TEMP | PULL_TEMP | Profile MIN_EXTRUDE_TEMP |
|---|---:|---:|---:|---:|
| PLA | 250 | 45 | 100 | 160 |
| PETG | 250 | 45 | 100 | 180 |
| ABS | 255 | 50 | 120 | 190 |
| NYLON | 260 | 50 | 120 | 190 |

All temperatures are in °C. These starting values come from Happy Hare. The successful printer test does not establish that every material profile has been tested. Adjust the temperatures for your filament and hotend.

## Parameters

| Parameter | Meaning | Default |
|---|---|---|
| `MATERIAL` | `PLA`, `PETG`, `ABS` or `NYLON` | `PLA` |
| `HOT_TEMP` | Initial purge temperature | Material profile |
| `COLD_TEMP` | Cooling endpoint before reheating | Material profile |
| `PULL_TEMP` | Temperature at which to pull manually | Material profile |
| `MIN_EXTRUDE_TEMP` | Requested cutoff for topping up the nozzle | Material profile |
| `CLEAN_LENGTH` | Initial purge length in whole mm, from 1 to 100 | `25` |
| `EXTRUDE_SPEED` | Purge and top-up speed in mm/s, greater than 0 and at most 5 | `1.5` |

Example override:

```gcode
SM_COLD_PULL MATERIAL=PLA HOT_TEMP=230 PULL_TEMP=100
```

`MIN_EXTRUDE_TEMP` only controls when this macro stops topping up the nozzle. **It does not change Klipper's cold-extrusion protection.** The effective cutoff is the higher of the requested value and the extruder's configured minimum extrusion temperature plus 10°C. With the stock 170°C minimum, PLA top-up moves therefore stop above 180°C. A helper checks the current temperature and `can_extrude` status again after each temperature wait.

The following relationship is required:

```text
30 <= COLD_TEMP < PULL_TEMP < effective top-up cutoff < HOT_TEMP <= configured max_temp - 5
```

Unknown parameters are rejected.

The heater is off during cooling until `COLD_TEMP` is reached, then the nozzle reheats to `PULL_TEMP`. On normal completion, the previous part-cooling fan setting is restored and the selected tool's heater target remains at zero.

## Stopping the procedure

The macro uses blocking temperature waits. Ordinary console commands may remain queued. There is no dedicated graceful-cancel command or macro-level temperature-wait timeout.

If immediate shutdown is necessary, use the printer interface's emergency stop (`M112`). After an error, check heater and fan settings before trying again: Klipper macros do not guarantee cleanup after every failure.

## Compatibility and verification

The adaptation was reviewed against [Snapmaker/u1-klipper commit 10f2f697](https://github.com/Snapmaker/u1-klipper/tree/10f2f69714c059f1dee0423db4f515b8a01f5388), including the four extruders, fan selection, cold-extrusion checks, macro templates and command parser.

Local checks covered template rendering, material/tool combinations, invalid parameters, printer-state guards and temperature-dependent top-up moves.

I tested this macro on my Snapmaker U1 on September 26, 2026, and it worked very well. This confirms operation on my printer; it does not mean that every firmware version, tool or material profile has been tested.

## Troubleshooting

| Message or symptom | Action |
|---|---|
| `Unknown command:"U1"` | The old name `U1_COLD_PULL` is incompatible with the parser. Replace the entire macro file, run `RESTART`, and use `SM_COLD_PULL`. |
| `Unknown command:"SM_COLD_PULL"` | Check the include line, file location and whether `RESTART` was completed. |
| `The U1 must be idle...` | Finish printing, filament loading, calibration or another active operation. |
| `Home first...` | Home the printer and position the tool at least 20 mm above the bed. |
| `Selected tool is not confirmed mounted...` | Select the tool normally and check that it is mounted on the carriage. |
| `Invalid temperatures...` | Check the temperature ordering and heater limit described above. |
| Cooling never reaches `COLD_TEMP` | Check ambient temperature and airflow. There is no macro-level wait timeout. |

Do not insert backslashes before underscores in commands. Keep helper macro names unchanged.

## Reporting issues

Include your U1 firmware version, selected tool, nozzle type, filament, exact command, and console messages. Describe which stage failed. A complete hardware test should cover purging, cooling, reheating, manual extraction and heater shutdown.

## Provenance and license

This adaptation uses the workflow and material profiles from [`MMU_COLD_PULL` as distributed in Happy Hare](https://github.com/moggieuk/Happy-Hare/blob/main/config/macros/mmu_misc.cfg). This identifies the source used for the adaptation; it does not establish who first wrote the macro or invented the cold pull technique.

The upstream source file carries the notice `Copyright (C) 2022-2026 moggieuk` and specifies GNU GPLv3. That file-level notice is retained in the macro header as an upstream copyright notice, rather than a claim of sole authorship of the cold pull macro.

Distributed under GNU GPL v3 (`GPL-3.0-only`). See [LICENSE](LICENSE).

U1 implementation references: [extruder handling](https://github.com/Snapmaker/u1-klipper/blob/10f2f69714c059f1dee0423db4f515b8a01f5388/klippy/kinematics/extruder.py) and [printer configuration](https://github.com/Snapmaker/u1-klipper/blob/10f2f69714c059f1dee0423db4f515b8a01f5388/lava/printer.cfg).
