# Design history

This page records what was published for SLC_USBP, when, what the published
design files can do, and which published measurements cannot be reproduced
from them. Every statement cites a file, a commit or a page.

## What was published

**Paper.** S. Oberloier, N. G. Whisman, F. Hafting and J. M. Pearce, "Open
source framework for a Broadly Expandable and Reconfigurable data acquisition
and automation device (BREAD)", *HardwareX* 15 (2023) e00467,
doi:10.1016/j.ohx.2023.e00467 (Europe PMC PMC10498007). Received 9 August 2022,
revised 25 July 2023, accepted 24 August 2023. The SLC_USBP material is the
section "SLC_USBP 2 Channel USB Port" (Figs. 34-36), the characterization
section (Table 9, Fig. 53) and the SLC_USBP bill of materials.

**OSF project.** "BREAD Slice - USB Port (SLC_USBP)", osf.io/j9rh5, cited as
reference [42] in the paper (N. Whisman, S. Oberloier, J.M. Pearce, F.
Hafting, 2021; accessed 24 October 2021). Created 8 June 2021. OSF keeps every
uploaded version of a file. The project holds three design revisions
(dates from the OSF API, `api.osf.io/v2/nodes/j9rh5/files/osfstorage/` and
each file's `versions` listing):

| Upload date | Files | Tool | UART wiring |
| --- | --- | --- | --- |
| 2021-06-08 | Gerbers, version 1 (`%TF.CreationDate 2021-06-08`); `Documentation/USBP_BOM.csv` | KiCad 5.1.10 | FT232 TXD to Nano D0/RX, FT232 RXD to Nano D1/TX |
| 2021-07-20 | `BREAD_Slice.sch`, `.net`, `.pro`, `.kicad_pcb` version 1, `.kicad_pcb-bak` | KiCad 5.1.10 | same as above |
| 2022-08-09 | `BREAD_Slice.kicad_sch`, `.kicad_pcb` version 2, Gerbers version 2, `BOM_USBP.ods` | KiCad 6.0.4 | FT232 TXD to Nano D1/TX, FT232 RXD to Nano D0/RX |
| 2023-07-24 | `.kicad_pcb` version 3, Gerbers version 3, `Schematic_USBP.pdf`, `Firmware/USB_Test_Device.ino`, case STLs | KiCad 6.0.4 | same as 2022 |

The current (default) download of every Gerber is the 2023 version. The 2021
Gerbers remain downloadable as OSF version 1 of each file (for example
`https://osf.io/download/60bfad2bda0ff2001b38ffaf/?revision=1`, the bottom
copper layer). The 2021 job file lists front and back silkscreen layers that
are not on OSF.

All three board layouts carry the silkscreen text "USB Slice / SLC_USBP / R1
2021". The 2023 layout adds connector labels (12V, SCL, SDA, ESTOP, SYNC, INT,
USB1, USB2, ARDUINO NANO). The 2022 and 2023 layouts add C5 and R2, a DTR
auto-reset network from the FT232 to the Nano's reset pin. The 2023 copper
layers are identical to the 2022 ones apart from the creation date.

The paper's Fig. 35 (FT232 schematic, with C5 and R2) and Fig. 36 (routed
board, with the 2023 connector labels) show the 2023 revision.

**This repository.** Commit afd3c3b (2024-05-23) imported the 2023 OSF files;
its Gerbers match OSF version 3 apart from line endings. Commit 539db76
(2024-09-09) redrew the board (12 V-to-5 V buck, WS2812, Qwiic connector; the
FT232 became U3 and the mux U2). Commit 0e3d950 (2024-09-30) added rev A
Gerbers marked "rev.A [09-2024]". Tag `rev-a-2024` holds that design in KiCad
10 form.

## The UART wiring changed between 2021 and 2022

The FT232RL's TXD (pin 1) is an output and RXD (pin 5) an input. The Arduino
Nano's D1/TX (footprint pad 1) is an output and D0/RX (pad 2) an input. A
working link joins each output to the other side's input.

- 2021 (KiCad 5): net `/TR` joins FT232 pin 1 (TXD) to Nano pad 2 (D0/RX), and
  `/RT` joins FT232 pin 5 (RXD) to Nano pad 1 (D1/TX). This is the correct
  crossover. Sources: `BREAD_Slice.net` (nets `/TR` and `/RT`), the Gerber X2
  pad attributes in Gerbers version 1, and `.kicad_pcb` version 1.
- 2022 and 2023 (KiCad 6): `/TR` joins FT232 pin 1 (TXD) to Nano pad 1
  (D1/TX), and `/RT` joins FT232 pin 5 (RXD) to Nano pad 2 (D0/RX). Two
  outputs share one net and two inputs share the other. Sources: pad
  attributes in Gerbers versions 2 and 3 (`BREAD_Slice-B_Cu.gbr`), `.kicad_pcb`
  versions 2 and 3, and every board in this repository's history.
- The change is in the schematic. The Nano symbol sits at the same place in
  both schematics with the same pin geometry. In the 2021 schematic the `TR`
  label is wired to D0/RX and `RT` to D1/TX; in the 2022 schematic the two
  labels are exchanged at the Nano. The FT232 end is unchanged.

On a 2022 or 2023 board the FT232 cannot pass data to or from the Nano.

## What every published revision can and cannot do

- **USB role.** The FT232RL is a USB device. J2 and J3 are USB-A
  receptacles, the connector a host provides. The board has no USB host
  controller, and the ATmega328P on the Nano has no USB peripheral. A USB
  device plugged into J2 or J3 meets another device, and nothing enumerates
  either one. No revision can communicate with USB devices plugged into its
  ports.
- **The only working USB link** is a USB host, such as a PC, connected to J2
  or J3 with an A-to-A cable, with the FT232 as the device. That cable is
  outside the USB specification. VBUS of both ports is tied to the slice's
  +5 V rail through FB1, so the slice's rail and the PC's VBUS are joined.
- **Mux control.** The FSUSB42 SEL pin is wired to Nano D7 with no pull
  resistor. The published firmware never configures D7. The FSUSB42
  datasheet (onsemi, Note 5) says the control input must be held high or low
  and must not float. Its recommended supply is 2.4-4.4 V (absolute maximum
  5.6 V); every revision runs it from +5 V.
- **SPI.** The FT232 connects to the Nano's UART pins (D0/D1). The Nano's
  SPI pins (D11-D13) are not connected to it on any OSF revision.
- **Firmware.** `firmware/USB_Test_Device.ino` (identical to the OSF copy
  apart from line endings) opens the serial port at 9600 baud, reads one
  character after reset, sets the on-board LED if it is `r`, and stops
  reading. The line that would send a reply is commented out. It has no I2C
  code.

With the 2021 board, a PC host on an A-to-A cable and a different sketch that
echoes characters at 500 kbaud, the hardware path for an echo exists. With the
2022 or 2023 board it does not, because of the UART wiring.

## Published measurements and the published files

| Paper statement | Verdict | Evidence |
| --- | --- | --- |
| 2 USB-A ports into a USB switch (Fig. 34) | Consistent | J2/J3 to FSUSB42 in all revisions |
| FT232 converts USB to "Arduino-readable SPI format" | Contradicted | FT232 wired to UART pins D0/D1; SPI pins unconnected |
| "Can connect and communicate with up to two USB devices" | Contradicted | No USB host on the board (see above) |
| Max speed 0.5 Mbps (text, Table 9) | Not reproducible | Published firmware runs at 9600 baud and does not echo; 2022/2023 boards cannot pass FT232 data |
| Echo test: a character sent "through the USB connector", received and sent back | Not reproducible | As above; no published firmware echoes |
| Up to 64 characters in one transmission | Not reproducible | Published firmware reads one character per reset |
| Fig. 53, "four 'k' characters ... over UART" at 0.5 and 1 Mbps | Consistent with UART; not with USB | See below |
| Figs. 35 and 36 | Consistent with the 2023 files | Those files have the TX-to-TX wiring |
| BOM: 7 lines, 20.04 USD headline | Contradicted | Line totals sum to 27.06 (Canadian supplier links); FB1, C3, C4, C5, R1 and R2 are on the board but not in the BOM |
| "All slices have had basic communication functionality validated (both via the Backplane and via USB)" | Not reproducible | The published firmware has no I2C, so it cannot answer the backplane |

**Fig. 53.** The two oscilloscope captures (20 µs/div and 10 µs/div, 8 MSa/s)
show idle-high asynchronous serial frames with bit times of about 2 µs and
1 µs, consistent with 0.5 and 1 Mbit/s UART. USB full-speed signalling on D+
and D- runs at 12 Mbit/s (83 ns per bit), which an 8 MSa/s capture cannot
resolve. The trace labelled "USB Connector (call)" is therefore a UART line,
not the USB connector's data pins. The figure shows a serial echo at those
baud rates between whatever drove that line and the Nano. It does not show
USB enumeration, traffic through the mux, or any USB device on J2 or J3, and
the captures do not identify which board or sketch was used. 0.5 Mbit/s is
a UART rate, not a USB 2.0 link rate.

No board, sketch or bench record behind Table 9 or Fig. 53 has been published.
The OSF project, this repository and the paper record none.

## Response

Rev B replaces the FT232/mux design with switched, current-limited, metered
USB power ports (#2). Tag `rev-a-2024` keeps the earlier design.
