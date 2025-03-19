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

I had JLCPCB do the PCBs, they work out to ~$7 ea (CAN Adapter) and ~$10 ea (CFS-Bus Adapter), at qty 5 of each. I'm also trying LCSC's [custom-cable service](https://www.lcsc.com/customcables?utm_source=customcables&utm_medium=navbar) to make the short interposing cables, which wound up being ~$50 for 10 of each.

Full production files for each of the PCBs are in the respective folders, they should drag & drop into JLCPCB's PCBA service. The cables are based on a lot of guesswork about what the right connectors are, once I confirm I have the right part-numbers, I'll document that here as well.

Note that immediately after ordering the CFS-Bus-Adapter, I realized it would be super neat to allow the CAN transciever to be either connected to the CAN peripheral in the MCU (which is how it is now), and a UART interface. That would let this interact with the existing firmware if/when it get's properly reverse engineered, or Creality finally stops violating the GPL and provides the source for their [distributed-in-compiled-form-only klipper extensions](https://github.com/Guilouz/Creality-K2Plus-Extracted-Firmwares/tree/main/Firmware/usr/share/klipper/klippy/extras) (see all `.so` files, which are compiled with cython to make them opaque).


------

First rev board issues:

 - Pinout on J3 for CFS Can adapter board is reversed.   
   This can be fixed by just mirroring the routing of the jumper cable.
 - The open-collector buffer signals have pullups to 5V in the buffer. That means the buffer status LED resistors as implemented (i.e. between 24V and the buffer status pins) are effectively always on, since the 5V pullup winds up sinking current from the 24V LED rail.   
   This needs to be fixed by rev-ing the board (or with track hackery). Right now, the CFS buffer just changes the LED brightness, rather then turning them on and off.