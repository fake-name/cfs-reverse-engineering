## CFS Internal PCBs

### Main Motherboard
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

#### Pinouts:

J4 (Power/Comm Connector)

| Pin #      | Function | MCU Pin No | Logical Pin |
| ------------- | ------------- | ------------- | -------- |
| 1 | +24V | n/a |  |
| 2 | +24V | n/a |  |
| 3 | GND | n/a |  |
| 4 | GND | n/a |  |
| 5 | RS485-B | see note |  |
| 6 | RS485-A | see note |  |
| 7 | Filament Buffer SW 1 | p.55 |  |
| 8 | Filament Buffer SW 2 | p.56 |  |

The 485 bus uses 3 pins:

 - P.25 - TX Enable/RX Disable (maybe inverted?)
 - P.26 - RX
 - P.27 - TX


J15 (FP LCD Connector)

| Pin #  |  Function | MCU Pin No | Logical Pin |
| -------| --------- | ---------- | ----------- |
| 1      | power     | 3.3V       |             |
| 2      | power     | GND        |             |
| 3      |           | p.16       |             |
| 4      |           | p.17       |             |
| 5      |           | p.18       |             |
| 6      |           | p.33       |             |
| 7      | backlight |            |             |

Note: The backlight pin is a open collector transistor.

J7 Front-Panel LED PCB Connector

| Pin #  |  Function | MCU Pin No | Logical Pin |
| -------| --------- | ---------- | ----------- |
| 1      | power     | 5.0V       |             |
| 3      |           | p.63       |             |
| 4      |           | p.93       |             |
| 5      |           | p.64       |             |
| 6      |           | p.90       |             |
| 7      |           | p.91       |             |
| 8      |           | p.66       |             |
| 9      |           | p.92       |             |
| 10     | power     | GND        |             |

Note: All LED connectors are Open-Collector Transistors.
Presumably, the LEDs are between 5V and the various open-collector
outputs.


Motor driver pins

|  Driver pin    |   M1   |   M2   |   M3   |   M4   |
| -----| ------ | ------ | ------ | ------ |
| IN 1 | p.84   | p.82   | p.62   | p.60   |
| IN 2 | p.83   | p.81   | p.61   | p.59   |

Current Feedback Pins

| Motor |  Pin   |
| ---   | ------ |
| M1    | p.34   |
| M2    | p.15   |
| M3    | p.24   |
| M4    | p.23   |

Each feedback pin is driven by an op-amp with some gain and a offset shift.

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