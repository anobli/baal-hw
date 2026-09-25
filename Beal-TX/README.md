# Beal-TX — output board

Beal-TX lives out in the field at the valves, powered by its own battery +
solar supply. It receives commands over LoRa from a [Beal-Rx](../Beal-Rx/README.md)
unit and drives valve solenoids accordingly. It is a LoRa peer node — it has
no wired connection to the existing controller, and only acts on commands
sent by the coordinator.

The schematic is split into three sheets: **I/O Ext Board Output**
(`input.kicad_sch` — the sheet name is more accurate than the filename,
which is a naming leftover), **Radio_MCU**, **Power**.

## I/O Ext Board Output (`input.kicad_sch`)

- **DRV8912QPWPRQ1** (U6) — a 12-channel automotive smart driver, controlled
  directly by the LoRa-E5 over SPI (`SDI`/`SDO`/`SCLK`/`SCS_N`), with
  `SLEEP_N`/`FAULT_N` handshake lines. There is no companion MCU on this
  board — unlike Beal-Rx's Inputs sheet, the LoRa-E5 talks to the driver
  itself.
- 3 valve output channels (`OUTPUT1`–`OUTPUT3`), each driven as a full
  H-bridge — not a shared-common topology. Each DRV8912 half-bridge is
  limited to 1A RMS, which isn't enough for the target valves, so each side
  of a valve's H-bridge is actually two DRV8912 half-bridges wired in
  parallel for extra current headroom. That's 4 half-bridges per valve
  (2 for the P side, 2 for the N side), which is why the driver's 12
  channels only yield 3 valve outputs rather than 6. Target valves are 9VDC
  latching solenoids (a latching valve needs only a brief pulse of the
  correct polarity to switch, then holds its position with no power
  applied — a good fit for a battery/solar-powered field unit).
- 3 status LEDs (`LED1`–`LED3`), one per valve channel.
- Field wiring lands on a 6-position screw terminal (J6, `691401710006B`) —
  6 pins for the 3 valve channels' P/N pairs.
- Fault detection matters more here than in a typical hobby project: a
  stuck-open or undetected-failed valve can waste water or damage a crop.
  Careful use of the DRV8912's `FAULT_N` reporting is a design requirement,
  not a nice-to-have.

## Radio_MCU (`radio_mcu.kicad_sch`)

- **LoRa-E5 module** (U5, Seeed, STM32WL-based) — the application MCU. It
  owns the LoRa radio, drives the status LED, reads the pairing button, and
  drives the DRV8912 directly over SPI.
- Pairing button behavior mirrors Beal-Rx:
  - Short press: pairs this peer with a Beal-Rx coordinator (implemented).
  - Double press: maps a specific output channel to a specific input channel
    on the paired Beal-Rx (planned, procedure still TBD).
  - Long press: probable factory reset (not decided).
- U.FL/antenna connector for the 868 MHz antenna.
- SWD (Tag-Connect) for programming/debugging the LoRa-E5.
- Two extension connectors (`Conn_02x04_Odd_Even`) reserved for future
  add-on boards — not used by anything today. Planned (not yet designed)
  candidates:
  - a sensor input module (flow meter, humidity sensor);
  - a second MCU board with an ESP32, adding smart features and WiFi/BT
    connectivity;
  - possibly an output-expansion board, to drive more than 3 valves from one
    Beal-TX.

## Power (`power.kicad_sch`)

- `TPSM863253` buck power module for the board's logic-level rail.
- `LM5157-Q1` (gated by `9V_EN`) boosts up to 9V to feed the DRV8912 — the
  drive voltage the target latching solenoid valves need for a latch pulse.
- Powered from a battery + solar pack, delivered to the board as 5V over the
  same USB-C connector type used on Beal-Rx (the board only sees a `VBUS`
  net — battery/solar sourcing happens in the external power pack).
- Automotive-grade parts are used deliberately, not for automotive
  compliance but because the board sits in an outdoor enclosure that can see
  high internal temperatures from direct sun exposure — automotive parts
  give margin on temperature rating (and generally ESD/robustness) that
  consumer-grade parts don't.

## Open items

- [ ] Channel-to-channel mapping procedure — design not finalized (shared
      with Beal-Rx, see its README).
- [ ] Long-press button behavior (factory reset) — not decided.
- [ ] Both extension connectors — reserved for planned add-on boards
      (sensor input module, ESP32 MCU board, output-expansion board), none
      designed yet.
- [ ] Battery chemistry/capacity for the power pack — not yet sized.
