# Beal-Rx — input board

Beal-Rx clips onto an existing irrigation controller's zone-output wiring,
senses which zone it's energizing, and reports that over LoRa to one or more
[Beal-TX](../Beal-TX/README.md) units. It is the LoRa coordinator of a Beal
system — it's where commands originate, and it's already the natural central
device since it's the one wired to the existing controller.

The schematic is split into three sheets: **Inputs**, **Radio**, **Power**.

## Inputs (`io_ext.kicad_sch`)

- 12 sense channels, each isolated from the field wiring by an opto-isolator
  (3× LTV-844S, 4 channels each). This gives galvanic isolation from whatever
  the existing controller puts on its zone outputs, with no current drawn
  from or injected back into the controller.
- Sized against 24VAC zone outputs but tolerant of smaller signals
  (9VAC/9VDC), so the same board can read other controller types without a
  hardware change.
- A 14-position terminal block (`TBL004-508-14BE-2GY`) brings in the 12
  channels plus the shared `INPUT_COM` return.
- A dedicated **STM32C031K6Ux MCU** on this sheet scans all 12 opto outputs
  and reports state changes up to the LoRa-E5 (I2C/UART, plus an interrupt
  line) — the LoRa-E5 doesn't have enough spare GPIOs to read 12 channels
  directly itself, so this small companion MCU owns the input-scanning job.
  It has its own SWD (TC2050) connector for programming/debug.

## Radio (`radio_mcu.kicad_sch`)

- **LoRa-E5 module** (U5, Seeed, STM32WL-based) — the application MCU. It
  owns the LoRa radio, drives the status LED (D3), reads the pairing button
  (SW2, on its own dedicated GPIO), and talks to the Inputs sheet's
  STM32C031 over I2C/UART to learn which zone just turned on.
- Pairing button (SW2) behavior:
  - Short press: pairs this coordinator with a Beal-TX peer (implemented).
  - Double press: maps a specific input channel to a specific output channel
    on a specific Beal-TX (planned, procedure still TBD).
  - Long press: probable factory reset (not decided).
- U.FL/antenna connector (J1) for the 868 MHz antenna.
- USB-C connector (J5) — power input only.
- SWD (Tag-Connect J7) for programming/debugging the LoRa-E5.
- One extension connector (J2, `Conn_02x04_Odd_Even`) reserved for a future
  add-on board (e.g. more sense inputs) — not used by anything today.

## Power (`power.kicad_sch`)

- `TPSM863253` buck power module producing the board's logic-level rail from
  USB 5V. No boost converter or valve driver on this board — Beal-Rx only
  senses, it never drives anything, so it has no need for the 9V drive rail
  that Beal-TX needs.
- Powered from a 5V USB supply (mains-adapter or similar) — it lives next to
  the existing controller, which typically has power available nearby.

## Open items

- [ ] Channel-to-channel mapping procedure (input on Beal-Rx → output on a
      specific Beal-TX) — design not finalized.
- [ ] Long-press button behavior (factory reset) — not decided.
- [ ] Extension connector (J2) use case — reserved, nothing designed against
      it yet.
