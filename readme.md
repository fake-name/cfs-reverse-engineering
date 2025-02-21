Creality CFS Filament system Reverse Engineering

This is part of an effort to use the Creality CFS feeder system as a general AMS system for any Klipper-based printer.

This *should* be trivial, as the printer this normally interoperates with is also a Klipper-based machine. However, as usual, Creality are violating the GPL by not releasing their changes and additions to klipper (Creality implemented a custom bus type, and a bunch of utility bits to talk to the filament box).

Given that, the other option is reverse engineering how the feeder works. Fortunately, that should be pretty straightforward for most of the bits.

Anyways, high resolution board images are [here](PCBs.md).

Some attempts at analyzing the various binary firmware and python objects are [here](firmware.md)

Overall system structure:

![screenshot](imj/CFS-Diagram.drawio.svg)

The board can be flashed with klipper by just treating it as having a STM32F106VET6 (which is binary compatible with the GD32F303VET6). You loose a bit of potential clock-rate by not doing giga-device specific stuff, but this is fairly slow anyways so it should be fine. 

Specifying the klipper host connection to be on PA2/PA3 even works (albeit with RS485).

