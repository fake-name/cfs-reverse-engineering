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

Current plan to convert the system to klipper in a neat way involves a simple interposer board to re-purpose the RS-485 lines:

![screenshot](imj/can_adapter.svg)

The adapter board sits between the existing rear-panel connector and the existing controller board. It basically has 3 connectors, a CAN transciever, and a bypass capacitors. It simply carries all other signals + power through the PCB, but replaces the two RS-485 lines with CAN signalling. 

Since the IO needed to drive a CAN interface is exposed on the unused "J10" connector, that is connected to an appropriate CAN bus transciever IC (a [TCAN332G](https://www.ti.com/lit/ds/symlink/tcan332g.pdf)). 

The other end is another custom PCB. In this case, it's another custom PCB heavily inspired by the [candleLight FD](https://linux-automation.com/en/products/candlelight-fd.html) (read: I used some of their schematics for a starting point, the entire layout is my own work). It breaks out the CFS bus to a usb-CAN adapter (via [candleLight_fw](https://github.com/candle-usb/candleLight_fw) and the 24V power in via either a DC Barrel jack or a 2-pin micro-fit connector (there are pads for both on the PCB).

It also has a pair of LEDs to indicate buffer status, and some activity LEDs. 

Both boards are up at [Adapter](~/Adapter/). License for them is GPL3, since the candleLight FD is GPL3.