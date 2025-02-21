# CFS Internal PCBs

## Main Motherboard
![screenshot](imj/P2060161.jpg)  
![screenshot](imj/P1010157.jpg)  

RS485 interface [3Peak TPT75176H](https://www.3peak.com/rs485/tpt75176h)
![screenshot](imj/P2060169.jpg)  

DC Motor Drivers: [HTD8236](https://offer-product.oss-cn-beijing.aliyuncs.com/product/offer/attachment/null/file/subPdf_749347_333113_20240123-141150640.pdf)
The use of the [LM358](https://www.ti.com/product/LM358) op amps are a mystery. Possibly current sensing? That would also align with the presence of the `R500` 500 mΩ resistors.   
Also present: a AMS1117 3.3v linear regulator. 
![screenshot](imj/P2060172.jpg)  

[24C16F EEPROM](https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-8719-SEEPROM-AT24C16C-Datasheet.pdf) in a SOT23-5 package.  
Q10 is probably a switch for the LCD's backlight LED?
![screenshot](imj/P2060173.jpg)  

Main CPU: [GD32F303](https://www.gigadevice.com/product/mcu/main-stream-mcus/gd32f30x-series/gd32f303) - STM32 ripoff ARM.
![screenshot](imj/P2060174.jpg)  

Brushless motor driver - [MS8828](https://www.relmon.com/en/index.php/list/detail/300.html). Made by [Ruimeng TECHNOLOGY](https://www.relmon.com)(sic). This part is interesting, because the firmware has references to a different part in some strings in it (`MS3791`)
![screenshot](imj/P2060175.jpg)  

## Pinouts:

### Connector types:
The majority of the connectors on the PCB are variants of the JST ["GH" connector style](https://www.jst.com/products/crimp-style-connectors-wire-to-board-type/gh-connector/), though not likely actually made by JST. 

All connectors other then the RS-485/Power (J4), the LCD (J15) and the filament monitor (J6) are GH-style. This includes the feeder motor connector (J5) which is indeed a GH connector, though with a right-angle-mount PCB connector.

##### J10 "Address" connector (not used?)

This connector is particularly interesting, since it is unused in the CFS configuration, and exposes the USB pins of the MCU. That means it should be possible to make up a USB cable. Alternatively, it also exposes the CAN pins.

The USB interface is annoying in that the MCU documentation specifies approximately 50Ω series termination resistors on both the DM and DP pins, which are not present. It may be possible to add these resistors in-line with the cable. I need to do some experiments.

PA4,6,7 don't have any features that seem to make them of particular interest. 

| Pin #  |  Function | MCU Pin No | Logical Pin | Note |
| -------| --------- | ---------- | ----------- | ----------- |
| 1      | power     | 3.3V       |             |             |
| 2      |           | P.70       | PA11        | USBDM / CAN0_RX       |
| 3      |           | p.71       | PA12        | USBDP / CAN0_TX       |
| 4      |           | p.31       | PA6         |             |
| 5      |           | p.32       | PA7         |             |
| 6      |           | p.29       | PA4         |             |
| 7      | power     | GND        |             |             |

##### J4 (Power/Comm Connector)
 
| Pin #  | Function             | MCU Pin No | Logical Pin |
| ------ | -------------------- | ---------- | ----------- |
| 1      | +24V                 | n/a        |             |
| 2      | +24V                 | n/a        |             |
| 3      | GND                  | n/a        |             |
| 4      | GND                  | n/a        |             |
| 5      | RS485-B              | see note   |             |
| 6      | RS485-A              | see note   |             |
| 7      | Filament Buffer SW 1 | p.55       | PD8         |
| 8      | Filament Buffer SW 2 | p.56       | PD9         |
The 485 bus uses 3 pins:
 
 - P.25 (PA2) - TX Enable/RX Disable - 2N7002 translator
 - P.26 (PA3) - RX
 - TX - I think strapped to ground. 

This is I believe technically violating RS-485 specs.

This device, like many crappy chinese "RS-485" systems, does not *really* support RS485. Instead, the bus is run in a pseudo-differential-open-collector mode (much like CAN networks), in that the transmitter is permanently wired up as "transmitting", e.g. the "space" mode, and rely on the bus termination resistors to drive the bus back to the idle "mark" state. Aside from not being real RS-485, this STRONGLY limits the maximum baud-rate the bus due to the fact that the only termination resistors present on the PCB are two 300Ω resistors, with `A` pulled to VCC (5V) and `B` pulled to ground.

While there is a provision for a 120Ω termination resistor on the board, it is not loaded on the board I have.



##### J15 (FP LCD Connector)


| Pin #  |  Function | MCU Pin No | Logical Pin |
| -------| --------- | ---------- | ----------- |
| 1      | power     | 3.3V       |             |
| 2      | power     | GND        |             |
| 3      |           | p.16       | PC1         |
| 4      |           | p.17       | PC2         |
| 5      |           | p.18       | PC3         |
| 6      |           | p.33       | PC4         |
| 7      | backlight |            |             |


Note: The backlight pin is a open collector transistor.

##### J7 Front-Panel LED PCB Connector

| Pin #  |  Function | MCU Pin No | Logical Pin |
| -------| --------- | ---------- | ----------- |
| 1      | power     | 5.0V       |             |
| 3      |           | p.63       | PC6         |
| 4      |           | p.93       | PB7         |
| 5      |           | p.64       | PC7         |
| 6      |           | p.90       | PB4         |
| 7      |           | p.91       | PB5         |
| 8      |           | p.66       | PC9         |
| 9      |           | p.92       | PB6         |
| 10     | power     | GND        |             |

Note: All LED connectors are Open-Collector Transistors.
Presumably, the LEDs are between 5V and the various open-collector
outputs.


##### Motor driver pins

|  Driver pin  |   M1       |   M2       |   M3        |   M4        |
| ------------ | ---------- | ---------- | ----------- | ----------- |
| IN 1         | p.84 (PD3) | p.82 (PD1) | p.62 (PD15) | p.60 (PD13) |
| IN 2         | p.83 (PD2) | p.81 (PD0) | p.61 (PD14) | p.59 (PD12) |

##### Current Feedback Pins

| Motor |  Pin   | Logical Pin |
| ---   | ------ | ----------- |
| M1    | p.34   | PC5         |
| M2    | p.15   | PC0         |
| M3    | p.24   | PA1         |
| M4    | p.23   | PA0         |

Each feedback pin is driven by an op-amp with some gain and a offset shift.

##### Brushed Motor connector / J5


| Pin #  |  Function | MCU Pin No | Logical Pin |
| -------| --------- | ---------- | ----------- |
| 1      | I2C?      | P.79       |  PC11       |
| 2      | Power     | GND        |             |
| 3      | I2C?      | p.80       |  PC12       |
| 4      |           | p.88       |  PD7        |
| 5      |           | p.87       |  PD6        |
| 6      |           | p.86       |  PD5        |
| 7      |           | p.85       |  PD4        |
| 8      | Power     | 5V         |             |
| 9      |           | M4-Out1    |             |
| 10     |           | M4-Out2    |             |
| 11     |           | M3-Out1    |             |
| 12     |           | M3-Out2    |             |
| 13     |           | M2-Out1    |             |
| 14     |           | M2-Out2    |             |
| 15     |           | M1-Out1    |             |
| 16     |           | M1-Out2    |             |


Pins 1 & 3 have 2N7002 discrete level translators to go from 5V I2C to 3.3V MCU I2C, I believe. They connect thru to the Temp/Humidity measurement I2C sensor in the chamber. 

##### RFID Readers / J8 / J1

| Pin #  |  Function | J8          | J19         |
| -------| --------- | ----------- | ----------- |
| 1      | Power     | 3.3V        |             |
| 2      | I2C?      | p.48 (PB11) | p.96 (PB9)  |
| 3      | Power     | GND         |             |
| 4      | I2C?      | p.47 (PB10) | p.95 (PB8)  |
| 5      | Power     | GND         |             |
| 6      |           | N/C?        |             |

Pin 6 on both seems to be not-connected, surprisingly.

##### J6 - Odometer and second-stage filament presence detector


| Pin #  |  Function | MCU Pin No | Logical Pin |
| -------| --------- | ---------- | ----------- |
| 1      | Power     | 3.3V       |             |
| 2      |           | p.46       |  PE15       |
| 3      |           | p.53       |  PB14       |
| 4      |           | p.54       |  PB15       |
| 5      |           | p.52       |  PB13       |
| 6      |           | p.51       |  PB12       |
| 7      |           | p.89       |  PB3        |
| 8      | Power     | Gnd        |             |
| 9      |           | p.38       |  PE7       |
| 10     |           | p.39       |  PE8       |
| 11     |           | p.40       |  PE9       |
| 12     |           | p.41       |  PE10       | 

Pins 9-12 have R/C filters on them. My current guess is they are 
the filament presennce paths, while the other 6 are for the odometer.

Hopefully I can extend https://github.com/Klipper3d/klipper/pull/5369 to support the odometer.

##### BLDC Motor Driver:

| Driver Func | MCU Pin No | Logical Pin |
| ----------- | ---------- | ----------- |
| SS          | p.03       | PE4         |
| FG          | p.42       | PE11        |
| F/R         | p.02       | PE3         |
| PWM         | p.44       | PE13        |

FG is a tachometer output from the driver. SS is motor brake, F/R is forward/reverse, PWM is speed control (obviously).


### RFID Readers:

[FM17622](https://eng.fmsh.com/AjaxFile/DownLoadFile.aspx?FilePath=/UpLoadFile/20190410/FM17510_ps_eng.pdf&fileExt=file) - Supports most protocols and interfaces. 13.56 MHz (I think?)
![screenshot](imj/P2060177.jpg)  

![screenshot](imj/P2060180.jpg)  

Both RFID readers are identical.
![screenshot](imj/P2060181.jpg)  

### LCD 

Front-Panel LCD - [TM1621B](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2209231730_TM-Shenzhen-Titan-Micro-Elec-TM1621B_C5174490.pdf) - Stand-Alone LCD driver, no MCU. 
![screenshot](imj/P2060184.jpg)  

### Connector PCB
![screenshot](imj/P2060186.jpg)  

Cable colors

 - Yellow - 24V
 - Green - GND
 - Blue - 485
 - Red - 485
 - White - sw?
 - Black - sw?

### Filament sensor PCB

Forgot to take pictures of the other side of the board.
![screenshot](imj/P2060191.jpg)  

### Filament feeder assembly

This is partly a pass-thru to the filament sensor PCB (above). 

It also has a MagnTek [MT6826S](https://www.magntek.com.cn/upload/pdf/202407/MT6826S_Datasheet_Rev.1.1.pdf). 
![screenshot](imj/P2060197.jpg)  

![screenshot](imj/P2060198.jpg)  



### Filament 4-into-1 buffer PCB.
![screenshot](imj/P1010147.jpg)  
![screenshot](imj/P1010150.jpg)  

The 4-into-1 PCB is very dumb, it just has 2 open-collector outputs that correspond to each optointerruptor. One for each end of the buffer-state.

All the parts on the board are just a DC-DC converter to produce a local power supply from the 24V bus. 

Since all CFS boxes will have access to the buffer state pins, whichever one is currently active in feeding to the print head can autonomously keep the buffer loaded without needing the printer controller's involvement.