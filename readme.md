Creality CFS Filament system Reverse Engineering

This is part of an effort to use the Creality CFS feeder system as a general AMS system for any Klipper-based printer.

This *should* be trivial, as the printer this normally interoperates with is also a Klipper-based machine. However, as usual, Creality are violating the GPL by not releasing their changes and additions to klipper (Creality implemented a custom bus type, and a bunch of utility bits to talk to the filament box).

Given that, the other option is reverse engineering how the feeder works. Fortunately, that should be pretty straightforward for most of the bits.

Anyways, high resolution board images are [here](PCBs.md).

Some attempts at analyzing the various binary firmware and python objects are [here](firmware.md)