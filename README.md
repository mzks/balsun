# balsun - Balloon adaptive link board with Wi-SUN

This is an arduino-compatible board for balloon experiments.

## Features

 - Arduino UNO R4 Minima-compatible hardware
 - Low EMC noise for general balloon RF system
 - Wi-SUN FAN communication (MIC/TELEC, FCC)
 - Open hardware and software

## Specifications

 - MCU Renesas RA4M1
 - 5V IOs (GPIO, UART, SCI, I2C, CAN, ADC, DAC, OpAmp) with castellated holes
 - Write bootloader/sketch/program via USB type-C
 - Multi power sources (1.5--5V) and more (users can mount their regurator via through holes)
 - PCB size is 64.5 * 53.4 mm, FR-4 1.6 mm
 - Wi-SUN with BP35C5 (RoHM) and two types of the anntena

## How can I get?

All hardware design is published as KiCAD files.
You can ask some manifactures to build them.
If you are customer of JLCPCB, Elecrow, FusionPCB, or PCBWay, you can find the zip files including everything to build PCB in [here](/hardware/info/gerber_to_order).
For PCBA, BOM and Pos files are [here](/hardware/info).
All parts are available in Mouser (2024 Oct. 24).

## Contact
Keita Mizukoshi (mizukoshi.keita@jaxa.jp)

## Local information
Japanese specific information and discussion among local balloon community in [jp-docs](/docs/jp-docs.md).
