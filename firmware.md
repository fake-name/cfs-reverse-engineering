Firmwares:

MCU Firmware files are taken from [here](https://github.com/Guilouz/Creality-K2Plus-Extracted-Firmwares/tree/main/Firmware/usr/share/klipper/fw/cfs). In particular, I'm examining `cfs0_050_G32-cfs0_000_113.bin`, admittedly for no particular reason other then it seems the most current version.

Just throwing the firmare into `strings already produces a LOT of really interesting snippets:

```
 \ | /
- RT -     Thread Operating System
15:19:29
Jan  6 2025
 / | \     %d.%d.%d build %s %s
 2006 - 2022 Copyright by RT-Thread team
```
Interestingly, the MCU does not run a klipper based system. Instead, it seems to be based on RT-Thread, which is a primarily Chinese-origin RTOS. 

RT-Thread is apache licensed (not copyleft), but it does require attribution, and interestingly searching for `"creality" "rt-thread"` returns no meaningful results, so creality seems to have managed to even violate a bsd-style license!

Ghidra gives us the full version info:

```
    debug_print(s_\_|_/_08018378);
    debug_print(s_-_RT_-_Thread_Operating_System_08018384);
    debug_print(s_/_|_\_%d.%d.%d_build_%s_%s_080183c0,5,0,2,s_Jan_6_2025_080183b4,s_15:19:29_080183a8);
    debug_print(s_2006_-_2022_Copyright_by_RT-Thre_080183e4);
```

So this is based on RT-thread 5.0.2, built on `Jan_6_2025` at `15:19:29`. 5.0.2 was released
on Oct 7, 2023, so they're a bit behind in updating the copyrights in the code if nothing else.

There's also a LOT of debug/informational statements:

```

feeder reverse detooth slot_id:%d
hub reverse update [dir:%s, speed:%d]
feeder reverse detooth force
detooth update [dir:%s, speed:%d]
MS3791 is overload
motor_blocked_detect_timer
pull back hub PE retry:%d
pull back hub PE timeout, retry times exhausted
times:%d, %d error,T%d AT8236 and MS3791 push filament timeout
#<times:%d, %d error,T%d pull_back_hub motor block, state=%02x
perhaps pull back odometer
already pull back odometer
etc...
```

In particular, there appear to be at least the remaining stubs of a interactive interface:

```
slot%d read rfid and margin estimation
preloading
read
enable
disable
aging
Usage: 
preloading enable            - enable preloading
preloading disable           - disable preloading
preloading aging             - aging preloading
preloading read [slot_id]    - read filament info
```

```
Usage: 
bl24cxx init                            - init bl24cxx
bl24cxx capacity                        - bl24cxx get capacity
bl24cxx ready                           - bl24cxx is ready?
bl24cxx deadlock <address>              - bl24cxx deadlock
bl24cxx undeadlock                      - bl24cxx undeadlock
```

```
Usage: 
odo_mag init            - init magnetic odometer
odo_mag test            - test magnetic odometer communication
odo_mag monitor [en]    - print magnetic odometer positon
odo_mag reset           - reset magnetic odometer
odo_mag read step       - read raw value of magnetic odometer
odo_mag read mm         - read position of magnetic odometer
```

Having what are basically help strings in the generated firmware image is interesting. 

There are also a number of situations where there are strings with VT100 control characters in them:`[31m[E/odometer_magnetic]`, `[0m[D/odometer_magnetic]`, `[31m[E/preloading]`, etc... I suspect there may be a debug UART somewhere.

It seems unlikely to me that all the debug strings are fed back over the RS485 port, in particular considering that there can be up to 4 of these things sharing a single RS485 bus. 

There's also some super weird stuff:

`database checksum error, load default value ...`

I have no idea what a "database" would be in the context of a embedded MCU with no real storage space. My best guess is it's something related to the I2C EEPROM. There are also supporting strings: `read_motor_info eeprom                  - read motor block info from eeprom`.

I suspect there are a number of locations where weird terminology is more a function of poor translation. There are a lot of references to `<something>_aging` in various places, which I suspect may be ore accurately stated as a timeout or latency.

------

So I've been going through and naming function stubs, working through the decompilation of the MCU source, and things are starting to make some sense. They are definitely measuring the motor current for the brushed DC motors:

```
        rt_kprintf(s_current_channel_=_%d_0800445b + 1,uVar3);
        FUN_080073e8();
        FUN_0800c0fc(0xff);
        iVar1 = _DAT_08004474;
        while (fVar5 = (float)FUN_0800725a(), iVar2 = _DAT_08004490, (int)ABS(fVar5) < iVar1)
        {
            iVar2 = FUN_08003aa8(uVar3,0,5);
            if (iVar2 != 0)
            {
                rt_kprintf(s_AT%d_pull_out,_restart_08004476 + 2,uVar3);
                FUN_0800c130();
                goto LAB_0800425c;
            }
            FUN_08019cc0(10);
        }
```




------



Complete strings listing (trimmed a bit for for things that are obviously not real strings):

```
 sn_information=%s
heartbeat
preload
rs485
print_check
 current mode = %d
 485 uid = %d
15:19:26
Jan  6 2025
MF003 V%d.%d.%d build %s %s
[33m[W/system] 
feeder reverse detooth slot_id:%d
hub reverse update [dir:%s, speed:%d]
feeder reverse detooth force
detooth update [dir:%s, speed:%d]
===================================
motor info:
block
noblock
feeder0 %s
feeder1 %s
feeder2 %s
feeder3 %s
hub %s
slot_id=%d
Bfeeder_curr_voltage=%d.%02dV
feeder_thrust_voltage=%d.%02dV
feeder_resistance_voltage=%d.%02dV
feeder_target_speed=%d
feeder_pid_output=%d
feeder_dir=%s
hub_target_speed=%d.%02drps
hub_curr_speed=%d.%02drps
hub_pid_output=%d
hub_dir=%s
reverse detooth timeout, %dms
is_reverse_detooth
MS3791_is_blocked
MS3791 is overload
MS3791 is overload
motor_blocked_detect_timer
feeder_reverse_detooth_reenable_timer
#<feeding stop, odometer not change
@Judge attempt: %d
sec remaining: %d
Retry attempt: %d
%s: last_odom = %d.%02dmm
%s: odom - last_odom = %d.%02dmm
%s: %d!
illegal data
 pull back hub PE timeout
already pull back hub PE
pull back hub PE retry:%d
pull back hub PE timeout, retry times exhausted
#<pull_back_hub motor block, state=%02x
pull back odometer retry:%d
pull back odometer timeout
already pull back odometer
cmd_parse_out %d!
 --------------------out ret = %d
buffer_fill_timeout
forced_preloading %d
 forced_preloading!
 reverse_detooth_set [en] 
origin reverse_detooth_en = %d
new reverse_detooth_en = %d
realtime
eeprom
Usage: 
 eeprom_init_flag: %d
AHTxx_init_flag: %d
odometer_magnetic_init_flag: %d
Buffer_init_flag: %d
rfid_init_flag[0]: %d; rfid_init_flag[1]: %d
[31m[E/preloading] 
pretension hub insert fil detect timeout
pretension hub pullout fil detect timeout
[0m[D/preloading] 
%d realtime insert
%d realtime pullout
%d finish action
== first detect a card
other_slot_id FEEDER_DIR_IN
other_slot_id FEEDER_DIR_OUT
(rfid move)hub pullout fil detect timeout
rfid has a card
move count:%d
first detect success
==detect multi cards
margin_estimation_and_read_rfid feed hub timeout
hub move count:%d
[33m[W/preloading] 
margin estimation timeout, plese check the odometer of hub
[32m[I/preloading] 
detect rfid:%d
different rfid uid detected, retry ...
-------------------------------------------
the number of RFID retries read message is exhausted
 rfid continuous detect timeout, plese move next the slot
Crfid detected next to it, move it
the number of RFID retries next to the move is exhausted
 slot%d insert filament
[32m[I/preloading] 
slot%d pullout filament
presd
 slot%d start preloading
slot%d pretension
STATE_CHECK_HUB
hub has filament
slot%d read rfid and margin estimation
preloading
read
enable
disable
aging
Usage: 
preloading enable            - enable preloading
preloading disable           - disable preloading
preloading aging             - aging preloading
preloading read [slot_id]    - read filament info
is busy=%d
slot%d =====================================
    has filament=%d
    has rfid=%d
    filament percent=%d%%
    rfid_base
        month=%c
        day=%.*s
        year=%.*s
        supplier=%.*s
        batch=%.*s
        mat_id=%.*s
        col=%.*s
        len=%.*s
        number=%.*s
        reserve=%.*s
        flag=%d
    rfid_high
B        len=%d.%02d
 =====================
RFID reader id: %d 
test count: %d, 
card found count: %d, 
card found fail: %d
read pass count: %d, 
read fail count: %d
 sensor_init_fail!
 host_temp = %d, slave_temp = %d
host_hum = %d, slave_hum = %d
sensor_test_fail!
 current channel = %d
 AT%d pull out, restart
8T%d feed finish, start pull back
T%d pull back finish, wait for pull out
finish test
all count: %d
card found count: %d
read pass count: %d
========ERROR========
RED%d_CTL
BLUE%d_CTL
LIMIT_%d
BUF_LIMIT_1: PD8
BUF_LIMIT_2: PD9
 %d vol:%d.%03dV  
<IDETA_ADC_MCU:PC5
YA_PWM_1:PD2
YA_PWM_2:PD3
IDETB_ADC_MCU:PC1
YB_PWM_1:PD0
YB_PWM_2:PD1
IDETC_ADC_MCU:PA1
YC_PWM_1:PD14
YC_PWM_2:PD15
IDETD_ADC_MCU:PA0
YD_PWM_1:PD12
YD_PWM_2:PD13
==================MOTHER BOARD test==================
EEPROM_SCL:PE5
EEPROM_SDA:PE6
SCL2:PB8
SDA2:PB9
SCL1:PB10
SDA1:PB11
Look at the BLDC
FG1:PE11
PWM:PE13
BRAK:PE4
F&F%FO
AHT30_SCL_3V:PC11
AHT30_SDA_3V:PC12
SPI_CS:PB12
SPI_CK:PB13
SPI_MISO:PB14
SPI_MOSI:PB15
=====================FINISH TEST=====================
slot%d trigger success:%d
buffer error,please check connect and board
 error_full:%d, error_empty:%d
free_count:%d, full_count:%d, to full: %dms, to free: %dms
buffer component error !
dir = out, up; time = %d ms, count = %d
dir = back, up; time = %d ms, count = %d
dir = out, down; time = %d ms, count = %d
dir = out, down; time = %d ms
============================
BLDC gravity aging mode count = %d
BLDC_aging_test finished, count = %d
filament in channel%d
out ok
 back ok
 RFID_authentication
switch mode:%d
switch standard mode!
Cillegal mode!
switch_mode [mode]
print_buffer_aging
Current mode is not support this command
free_count:%d, full_count:%d
aging_photoelectric_CMD [first_id] [end_id]
[%d] %s: ch: %d ready !
==============================
try to move into reader timeout
read RFID message failed
try to move out reader timeout
ch = %d, in reader area time = %d ms, circle time = %d ms
 switch to STANDARD_MODE
 init
temp
slave_num
light
Usage: 
lcd init                   					-lcd init to 0
lcd hum     <hum>  									-lcd set hum, such as 'lcd hum 31'
lcd slave_num <slave_num>          -lcd set slave number
lcd light   <0/1>  									-lcd set light open/off
[0m[D/RS485_AUTO_ADDR] 
error package->head is %02X
package_crc is %02X, cal_crc is %02X
sda=%d, scl=%d
 init
 ready
undeadlock
deadlock
capacity
write
read
bl24cxx_is_ready:%d
Please using 'bl24cxx init' first.
bl24cxx_get_capacity:%d
bl24cxx_write_bytes:address=%d, data=%d
bl24cxx_read_bytes:address=%d, data=%d
Usage: 
bl24cxx init                            - init bl24cxx
bl24cxx capacity                        - bl24cxx get capacity
bl24cxx ready                           - bl24cxx is ready?
bl24cxx deadlock <address>              - bl24cxx deadlock
bl24cxx undeadlock                      - bl24cxx undeadlock
Bodometer_magnetic current position:%d.%02dmm
[0m[D/odometer_magnetic] 
success
fail
read userid %s, id=0x%02x
[31m[E/odometer_magnetic] 
odometer_magnetic_init communication fail
odometer_magnetic_init mt6826_timer create fail
init
reset
test
monitor
read
Please using 'odo_mag init' first.
odo_mag communication %s
step
Usage: 
odo_mag init            - init magnetic odometer
odo_mag test            - test magnetic odometer communication
odo_mag monitor [en]    - print magnetic odometer positon
odo_mag reset           - reset magnetic odometer
odo_mag read step       - read raw value of magnetic odometer
odo_mag read mm         - read position of magnetic odometer
odometer_magnetic current step:%d
 SetReg: send RFID_ADDR failed!
SetReg: send reg_addr failed!
SetReg: send data failed!
GetReg: send RFID_ADDR failed!
GetReg: send reg_addr failed!
GetReg:  RFID_ADDR read failed!
rfid[%d] softReset failed, reg_data = %x!
 ReaderA_Select: rfid[%d] reg_data = %x
 card %d auth block %d failed, reg_data = %x
card %d card_write:JREG_FIFOLEVEL reg_data = %d
card %d card_write 2:JREG_FIFOLEVEL reg_data = %d
card_write: block num %d is error!
card %d card_write 2:JREG_FIFODATA reg_data = %d
card_read: block num %d is error!
card %d card_read: reg_data = %d
can't find %s device!
read aht10 sensor humidity   : %d.%d %%
read aht10 sensor temperature: %d.%d
initialize sensor failed!
dbg_temp = %d, dbg_humi = %d
 detooth_test [id] [dir]
id is between 0 to 3
dir illegal
card read failed!
 228241200A1
1010010FF0000
0330000001000000
rfid_init_info_read %d, %d
 read vector %d is failed!
------------base %d info-------------
%s is 
 read vector 8 is failed!
------------high %d info-------------
now length = %d
 busy
none
unknown
init
 write
read
show
Usage: 
rfid init     <card id>               -rfid connect card
Please using 'rfid init <card id>' first.
base
high
RCrfid read     <card id>  <base/high>  -rfid card read data
read failed!
 status = %d
[32m[I/RS485] 
find %s success!
[31m[E/RS485] 
init %s failed!
init %s success!
open %s failed!
find %s failed!
 uid = %d
 preload_check_flag = 1, heartbeat update
 enable buffer!
disable buffer!
 heartbeat
 heartbeat_wait ret = %d
 rs485_lb
rs485 timeout, %d
 box status:
STATE_IDLE
STATE_PRELOAD
STATE_PRINT
STATE_RELOAD
STATE_ERROR
STATE_TEST
unknown
task_tset [0(out)/1(back)/2(out+back)] [motor_id] [times]
get device uid=0x%02x
protocol_dev.status=%d
Bmotor%d, average_vol=%d.%02d, starting_vol=%d.%02d
[32m[I/AT8236] 
find %s device!
[31m[E/AT8236] 
pwm run failed! can't find %s device!
adc run failed! can't find %s device!
 at8236_test id=%d speed=%d dir=%d
at8236(%d) voltage=%d.%02d
voltage(%d)=%d.%02d, voltage(%d)=%d.%02d
adct
 at8236 adc timer ok
at8236_pid_set outkp=%d outki=%d outkd=%d bkp=%d bki=%d bkd=%d
 motor_id = %d, led set err %x
 led_test col=%d id=%d, open=%d
tim_rotational_speed_detect
pwm1
[32m[I/MS3791] 
find MS3791 enable device!
[31m[E/MS3791] 
pwm run failed! can't find MS3791 enable dev!
MS3791_test speed=%d dir=%d
MS3791_time_test [dir] [speed] [time_ms]
dir is 0 or 1
speed is between 0 to 255
MS3791_time_test dir = %d speed = %d time_ms = %d
Bmove position = %d.%02dmm
 MS3791_read_fg pluse=%d
 ms3791_pid_set kp=%d.%02d ki=%d.%02d kd=%d.%02d
 buffer free!
buffer full!
Bodom first:%d.%02dmm,now=%d.%02dmm,diff:%d.%02dmm
 err 0x52!--1
 Berr 0x51
init buffer irq
 buffer status = %d
fmc_word_program fail. ret = %d, data = 0x%x
 %s: can't find the key: %d 
[33m[W/db_key_handler] 
database checksum error, load default value ...
?times:%d,T%d test_fixture %d ok
times:%d, %d error, pull back other filaments timeout!
T%d out start
times:%d, %d error,T%d out timeout!
feed: ch = %d, motor max voltage = %d.%02d v
last_odom = %d.%02dmm
odom - last_odom = %d.%02dmm
times:%d, %d error,T%d AT8236 and MS3791 push filament timeout
#<times:%d, %d error,T%d pull_back_hub motor block, state=%02x
perhaps pull back odometer
already pull back odometer
times:%d, %d error,T%d pull_back_hub timeout
times:%d, %d error,T%d back timeout!
back: ch = %d, motor max voltage = %d.%02d v
box aging finish all channel
 box_aging_mode_test [first_channel] [end_channel] [times]
channel ranges from 0 to 3
[33m[W/armlibc.syscalls] 
Please enable RT_USING_POSIX_FS
[33m[W/stdlib] 
thread:%s exit:%d!
 bus fault:
SCB_CFSR_BFSR:0x%02X 
IBUSERR 
PRECISERR 
IMPRECISERR 
UNSTKERR 
STKERR 
SCB->BFAR:%08X
psr: 0x%08x
r00: 0x%08x
r01: 0x%08x
r02: 0x%08x
r03: 0x%08x
r04: 0x%08x
r05: 0x%08x
r06: 0x%08x
r07: 0x%08x
r08: 0x%08x
r09: 0x%08x
r10: 0x%08x
r11: 0x%08x
r12: 0x%08x
 lr: 0x%08x
 pc: 0x%08x
hard fault on thread: %s
hard fault on handler
FPU active!
failed vector fetch
debug event
usage fault:
SCB_CFSR_UFSR:0x%02X 
UNDEFINSTR 
INVSTATE 
INVPC 
NOCP 
UNALIGNED 
DIVBYZERO 
mem manage fault:
SCB_CFSR_MFSR:0x%02X 
IACCVIOL 
DACCVIOL 
MUNSTKERR 
MSTKERR 
SCB->MMAR:%08X
 %bp
dev != RT_NULL
rt_object_get_type(&dev->parent) == RT_Object_Class_Device
rt_object_is_systemobject(&dev->parent)
rt_object_is_systemobject(&dev->parent) == RT_FALSE
[31m[E/kernel.device] 
To initialize device:%s failed. The error code is %d
dev->ref_count != 0
[33m[W/hwtimer] 
frequency setting out of range! It will maintain at %d Hz
timer != RT_NULL
timer->ops != RT_NULL
timer->info != RT_NULL
[33m[W/I2C] 
wait ack timeout
[31m[E/I2C] 
ACK or NACK timeout.
NACK: sending first addr
NACK: sending second addr
NACK: sending repeated addr
send bytes: error %d
i2c_bus_lock
[32m[I/I2C] 
I2C bus [%s] registered
[31m[E/I2C] 
I2C bus %s not exist
I2C bus operation not supported
bus != RT_NULL
bus != RT_NULL
buffer != RT_NULL
completion != RT_NULL
Function[%s]: scheduler is not available
Function[%s]: interrupt is disabled
Function[%s] shall not be used before scheduler start
Function[%s] shall not be used in ISR
rt_list_isempty(&(completion->suspended_list))
queue != RT_NULL
size > 0
queue->magic == DATAQUEUE_MAGIC
Function[%s]: scheduler is not available
Function[%s]: interrupt is disabled
Function[%s] shall not be used before scheduler start
Function[%s] shall not be used in ISR
data_ptr != RT_NULL
size != RT_NULL
ops != RT_NULL && ops->convert != RT_NULL
probe
 enable
read
disable
voltage
Unknown command. Please enter 'adc' for help
adc probe <device name>   - probe adc by name
success
probe %s %s 
failure
Please using 'adc probe <device name>' first
adc enable <channel>   - enable adc channel
%s channel %d enables %s 
adc read <channel>     - read adc value on the channel
%s channel %d  read value is 0x%08X 
adc disable <channel>  - disable adc channel
%s channel %d disable %s 
adc convert voltage <channel> 
%s channel %d voltage is %d.%03dV 
Usage: 
adc probe <device name> - probe adc by name
adc read <channel>      - read adc value on the channel
adc disable <channel>   - disable adc channel
adc enable <channel>    - enable adc channel
pin != RT_NULL
_hw_pin.ops != RT_NULL
pin [option] GPIO
     num:      get pin number from hardware pin
               e.g. MSH >pin mode GPIO output
     read:     read pin level of hardware pin
               e.g. MSH >pin read GPIO
               e.g. MSH >pin write GPIO high
     help:     this help list
GPIO e.g.:
output
input
input_pullup
input_pulldown
output_od
Parameter invalid : %s!
mode
read
write
Parameter invalid : %s!
probe
 enable
disable
phase
dead_time
Usage: 
pwm phase      <channel> <phase>            - set pwm phase
pwm probe <device name>                  - probe pwm by name
success
probe %s %s
failure
Please using 'pwm probe <device name>' first.
pwm enable <channel>                     - enable pwm channel
    e.g. MSH >pwm enable  1              - PWM_CH1  nomal
%s channel %d is enabled %s 
pwm disable <channel>                    - disable pwm channel
Get info of device: [%s] error.
Info of device [%s] channel [%d]:
period      : %d
pulse       : %d
Y@Duty cycle  : %d%%
Set info of device: [%s] error
Usage: pwm set <channel> <period> <pulse>
pwm info set on %s at channel %d
%s phase is set %d 
%s dead_time is set %d 
pwm probe      <device name>               - probe pwm by name
pwm phase      <channel> <phase>           - set pwm phase
pwm dead_time  <channel> <dead_time>       - set pwm dead time
[33m[W/UART] 
rx_fifo != RT_NULL
(serial != RT_NULL) && (data != RT_NULL)
rx_dma != RT_NULL
serial->ops->dma_transmit != RT_NULL
dev != RT_NULL
tx_fifo != RT_NULL
tx_dma != RT_NULL
serial != RT_NULL
serial->parent.rx_indicate != RT_NULL
tx != RT_NULL
len <= rt_dma_calc_recved_len(serial)
[31m[E/drv.adc] 
invalid channel
@invalid param
 failed register %s, err=%d
@invalid cmd:%x
failed register %s, err=%d
[31m[E/DBG] 
Can not find -1 of gd32_periph_list's member of Port!
Can not find -1 of gd32_periph_list's member of TimerIndex!
@Unsport timer periph!
%s register failed
[31m[E/DBG] 
Unsport gpio port!
Unsport timer periph!
result == RT_EOK
uart != RT_NULL
 serial != RT_NULL
cfg != RT_NULL
uart1
result == RT_EOK
shell != RT_NULL
finsh: can not find device: %s
tshell
shrx
no memory for shell
RT-Thread shell commands:
 %-16s - %s
total    : %d
used     : %d
maximum  : %d
available: %d
%s: command not found.
%-16s - %s
    %-16s - %s
retp
Too many args ! We only Use:
thread
rt_thread_t
%-*.*s 
 ---  ------- ---------- ----------  ------  ---------- ---
%-*.*s %3d 
 suspend
 ready  
 init   
 close  
 running
 0x%08x 0x%08x    %02d%%   0x%08x %s
semaphore
%-*.*s v   suspend thread
 --- --------------
%-*.*s %03d %d:
%-*.*s %03d %d
event
%-*.*s      set    suspend thread
  ---------- --------------
%-*.*s  0x%08x %03d:
%-*.*s  0x%08x 0
mutex
%-*.*s   owner  hold priority suspend thread 
 -------- ---- -------- --------------
%-*.*s %-8.*s %04d %8d  %04d 
%-*.*s %-8.*s %04d %8d  %04d
':F#F
!}#F
 `%r
mailbox
%-*.*s entry size suspend thread
 ----  ---- --------------
%-*.*s %04d  %04d %d:
%-*.*s %04d  %04d %d
msgqueue
%-*.*s entry suspend thread
 ----  --------------
%-*.*s %04d  %d:
%-*.*s %04d  %d
timer
%-*.*s  periodic   timeout    activated     mode
 ---------- ---------- ----------- ---------
%-*.*s 0x%08x 0x%08x 
activated   
deactivated 
periodic
one shot
current tick:0x%08x
device
%-*.*s         type         ref count
 -------------------- ----------
Unknown
%-*.*s %-20s %-8d
 Usage: list [options]
[options]:
%.*s
main
tid != RT_NULL
sem != RT_NULL
value < 0x10000U
(flag == RT_IPC_FLAG_FIFO) || (flag == RT_IPC_FLAG_PRIO)
rt_object_is_systemobject(&sem->parent.parent)
Function[%s] shall not be used in ISR
rt_object_is_systemobject(&sem->parent.parent) == RT_FALSE
 Function[%s]: scheduler is not available
Function[%s]: interrupt is disabled
Function[%s] shall not be used before scheduler start
mutex != RT_NULL
rt_object_is_systemobject(&mutex->parent.parent)
rt_object_is_systemobject(&mutex->parent.parent) == RT_FALSE
 event != RT_NULL
rt_object_is_systemobject(&event->parent.parent)
rt_object_is_systemobject(&event->parent.parent) == RT_FALSE
mb != RT_NULL
rt_object_is_systemobject(&mb->parent.parent)
rt_object_is_systemobject(&mb->parent.parent) == RT_FALSE
rt_object_is_systemobject(&mq->parent.parent)
rt_object_is_systemobject(&mq->parent.parent) == RT_FALSE
buffer != RT_NULL
size != 0
[33m[W/kernel.device] 
(%s) assertion failed at function:%s, line number:%d 
RT_NULL
unknown
EUNKNOW
 \ | /
- RT -     Thread Operating System
15:19:29
Jan  6 2025
 / | \     %d.%d.%d build %s %s
 2006 - 2022 Copyright by RT-Thread team
end_align > begin_align
heap
heap
level == RT_EOK
(rt_uint8_t *)mem >= m->heap_ptr
(rt_uint8_t *)mem < (rt_uint8_t *)m->heap_end
small
mem init, error begin address 0x%x, and end address 0x%x
m != RT_NULL
rt_object_get_type(&m->parent) == RT_Object_Class_Memory
rt_object_is_systemobject(&m->parent)
(((rt_ubase_t)mem) & (RT_ALIGN_SIZE - 1)) == 0
(((rt_ubase_t)rmem) & (RT_ALIGN_SIZE - 1)) == 0
small_mem != RT_NULL
MEM_ISUSED(mem)
rt_object_is_systemobject(&small_mem->parent.parent)
MEM_POOL(&small_mem->heap_ptr[mem->next]) == small_mem
(rt_uint8_t *)rmem >= (rt_uint8_t *)small_mem->heap_ptr
(rt_uint8_t *)rmem < (rt_uint8_t *)small_mem->heap_end
information != RT_NULL
obj != object
object != RT_NULL
Function[%s] shall not be used in ISR
!(object->type & RT_Object_Class_Static)
thread != RT_NULL
 thread != RT_NULL
thread:%s stack overflow
warning: %s stack is close to end of stack address.
thread != RT_NULL
priority < RT_THREAD_PRIORITY_MAX
stack_start != RT_NULL
(thread->stat & RT_THREAD_STAT_MASK) == RT_THREAD_INIT
rt_object_is_systemobject((rt_object_t)thread)
rt_object_is_systemobject((rt_object_t)thread) == RT_FALSE
thread == rt_thread_self()
thread != RT_NULL
 Function[%s]: scheduler is not available
Function[%s]: interrupt is disabled
Function[%s] shall not be used before scheduler start
Function[%s] shall not be used in ISR
tick != RT_NULL
timer != RT_NULL
timeout != RT_NULL
time < RT_TICK_MAX / 2
rt_object_get_type(&timer->parent) == RT_Object_Class_Timer
rt_object_is_systemobject(&timer->parent)
rt_object_is_systemobject(&timer->parent) == RT_FALSE
 (*(rt_tick_t *)arg) < RT_TICK_MAX / 2
timer
RTTUP
RTTDOWN
 jlinkRtt
 SEGGER_RTT ADDRESS:%p 
[32m[I/feed] 
feeding 500ms timeout, odometer not change
feeding 25s timeout
stage 7: enter buffer_check
[33m[W/feed] 
feeding timeout, buffer not full
feeding stop, buffer full
Bstage8: %d.%02dmm
?feeding stop, feed extruder timeout!
 sys_uid[0] = 0x%x
sys_uid[1] = 0x%x
sys_uid[2] = 0x%x
SIGRTRED: Redirect: can't open: 
pGfeed_hub_outside
pull_back_hub
feed_top_to_hub
cmd_parse_back_stage2
filament_reset_pos
database_get_value
database_set_value
STDIN
STDOUT
STDERR
remove
_sys_open
_sys_close
_sys_read
_sys_write
_sys_ensure
_sys_seek
_sys_flen
rt_device_unregister
rt_device_destroy
rt_device_init
rt_device_open
rt_device_close
rt_device_read
rt_device_write
rt_device_control
rt_device_set_rx_indicate
rt_device_set_tx_complete
rt_device_hwtimer_isr
rt_device_hwtimer_register
rt_i2c_master_recv
i2c_bus_device_read
i2c_bus_device_write
i2c_bus_device_control
rt_i2c_bus_device_device_init
rt_completion_init
rt_completion_wait
rt_completion_done
rt_data_queue_init
rt_data_queue_push
rt_data_queue_pop
rt_data_queue_peek
rt_data_queue_reset
rt_data_queue_deinit
rt_data_queue_len
rt_ringbuffer_init
rt_ringbuffer_put
rt_ringbuffer_put_force
rt_ringbuffer_get
rt_ringbuffer_peek
rt_ringbuffer_putchar
rt_ringbuffer_putchar_force
rt_ringbuffer_getchar
rt_ringbuffer_reset
rt_ringbuffer_create
rt_ringbuffer_destroy
rt_hw_adc_register
rt_adc_read
rt_adc_enable
rt_adc_disable
rt_adc_voltage
_pin_read
_pin_write
_pin_control
rt_pin_attach_irq
rt_pin_detach_irq
rt_pin_irq_enable
rt_pin_mode
rt_pin_write
rt_pin_read
rt_pin_get
_serial_buffer_clear
_serial_poll_rx
_serial_poll_tx
_serial_int_rx
_serial_int_tx
_serial_fifo_calc_recved_len
rt_dma_recv_update_get_index
rt_dma_recv_update_put_index
_serial_dma_rx
rt_serial_init
rt_serial_open
rt_serial_close
rt_serial_read
rt_serial_write
rt_serial_control
rt_hw_serial_register
rt_hw_serial_isr
rt_hw_i2c_init
gd32_uart_configure
gd32_uart_control
gd32_uart_putc
gd32_uart_getc
GD32_UART_IRQHandler
rt_hw_usart_init
finsh_get_prompt_mode
finsh_set_prompt_mode
finsh_getchar
finsh_rx_ind
finsh_set_device
finsh_get_device
finsh_set_echo
finsh_get_echo
_msh_exec_cmd
rt_application_init
_ipc_list_suspend
rt_sem_init
rt_sem_detach
rt_sem_create
rt_sem_delete
_rt_sem_take
rt_sem_release
rt_sem_control
rt_mutex_init
rt_mutex_detach
rt_mutex_create
rt_mutex_delete
_rt_mutex_take
rt_mutex_release
rt_mutex_control
rt_event_init
rt_event_detach
rt_event_create
rt_event_delete
rt_event_send
_rt_event_recv
rt_event_control
rt_mb_init
rt_mb_detach
rt_mb_create
rt_mb_delete
_rt_mb_send_wait
rt_mb_urgent
_rt_mb_recv
rt_mb_control
rt_mq_init
rt_mq_detach
rt_mq_create
rt_mq_delete
_rt_mq_send_wait
rt_mq_urgent
_rt_mq_recv
rt_mq_control
rt_hw_cpu_shutdown
0123456789abcdef
0123456789ABCDEF
_heap_unlock
rt_system_heap_init
plug_holes
rt_smem_detach
rt_smem_alloc
rt_smem_realloc
rt_smem_free
rt_object_init
rt_object_detach
rt_object_allocate
rt_object_delete
rt_object_is_systemobject
rt_object_get_type
rt_object_find
_scheduler_stack_check
rt_schedule_insert_thread
rt_schedule_remove_thread
_thread_timeout
_thread_init
rt_thread_init
rt_thread_startup
rt_thread_detach
rt_thread_delete
rt_thread_sleep
rt_thread_delay_until
rt_thread_control
rt_thread_set_suspend_state
rt_thread_suspend_with_flag
rt_thread_resume
rt_timer_init
rt_timer_detach
rt_timer_create
rt_timer_delete
rt_timer_start
rt_timer_stop
rt_timer_control
0123456789ABCDEF
TTR REGGES
0123456789ABCDEFq3buH@CFQ!uF^t1nkRnzSSmHqfZ(@KAtB3rUpf$1BJp2eZMDc|w{
motor blocked, MS3791=%d, at8236_0=%d, at8236_1=%d, at8236_2=%d, at8236_3=%d
read_motor_info realtime                - read motor info at realtime
read_motor_info eeprom                  - read motor block info from eeprom
rfid_fil_len_per_rotation=%d, circumference_percent=%d%%, area_percent=%d%%
feed filament exceed max length, start_pos=%d, curr_pos=%d, result=%d
rfid continuous detect timeout, spool not move(move_length=%d.%02d), plese check motor
ch	count	pass	fail	
 1	%d			%d			  %d			
 2	%d			%d			  %d			
 3	%d			%d			  %d			
 4	%d			%d			  %d			
lcd temp    <temp>									-lcd set temp, such as 'lcd temp 27'
lcd SlaveNumber <SlaveNum>          -lcd set slave number, such as lcd SlaveNum 1
get_uniid
get_addr_table
online_check
set_addr
bl24cxx read <address>                  - bl24cxx read one byte
bl24cxx write <address> <write_byte>    - bl24cxx write one byte
rfid write    <card id>  <base/high>  -rfid card write init data
mat id
reserve
batch
month
year
number
supplier
color
uart1
adc1
gd32_flash
times:%d, %d error,T%d MS3791 push filament timeout or buffer timeout
     mode:     set pin mode to output/input/input_pullup/input_pulldown/output_od
     write:    write pin level(high/low or on/off) to hardware pin
pwm probe      <device name>                - probe pwm by name
pwm dead_time  <channel> <dead_time>        - set pwm dead time
pwm enable     <channel>                    - enable pwm channel
pwm enable     <channel>                   - enable pwm channel
pwm disable    <channel>                    - disable pwm channel
pwm disable    <channel>                   - disable pwm channel
pwm get        <channel>                    - get pwm channel info
pwm get        <channel>                   - get pwm channel info
pwm set        <channel> <period> <pulse>   - set pwm channel info
pwm set        <channel> <period> <pulse>  - set pwm channel info
    e.g. MSH >pwm enable -1              - PWM_CH1N complememtary
Warning: There is no enough buffer for saving data, please increase the RT_SERIAL_RB_BUFSZ option.
adc0
adc2
pwm1
pwm3
pwm4
pwm5
pwm8
i2c0
uart0
uart2
uart3
%-*.*s pri  status      sp     stack size max used left tick  error
thread
Network Interface
DAC Device
ADC Device
MTD Device
SPI Device
PWM Device
CAN Device
WLAN Device
WDT Device
Graphic Device
Sound Device
USB Slave Device
Touch Device
Block Device
Portal Device
Pin Device
PM Pseudo Device
Timer Device
Character Device
Sensor Device
Bus Device
Miscellaneous Device
Phy Device
Security Device
device
Pipe
msgqueue
timer
list threads
list devices
list semaphores
list message queues
list timers
list events
I2C Bus
USB OTG Bus
SPI Bus
SDIO Bus
USB Host Bus
list mutexs
list mailboxs
event
mutex
mailbox
rt_object_get_type(&sem->parent.parent) == RT_Object_Class_Semaphore
rt_object_get_type(&mq->parent.parent) == RT_Object_Class_MessageQueue
rt_object_get_type(&event->parent.parent) == RT_Object_Class_Event
rt_object_get_type(&mutex->parent.parent) == RT_Object_Class_Mutex
rt_object_get_type(&mb->parent.parent) == RT_Object_Class_MailBox
OK     
EIO    
EPERM  
ETRAP  
ERROR  
EBUSY  
ENOSPC 
EINVAL 
ENOMEM 
ENOSYS 
ENOENT 
Using default rt_hw_cpu_shutdown().Please consider implementing rt_hw_cpu_reset() in another file.
rt_hw_cpu_reset() doesn't support for this board.Please consider implementing rt_hw_cpu_reset() in another file.
rt_hw_us_delay() doesn't support for this board.Please consider implementing rt_hw_us_delay() in another file.
ERSFULL
EINTRPT
ETIMOUT
ERSEPTY
((small_mem->lfree == small_mem->heap_end) || (!MEM_ISUSED(small_mem->lfree)))
(rt_ubase_t)((rt_uint8_t *)mem + SIZEOF_STRUCT_MEM) % RT_ALIGN_SIZE == 0
(rt_uint8_t *)rmem >= (rt_uint8_t *)small_mem->heap_ptr && (rt_uint8_t *)rmem < (rt_uint8_t *)small_mem->heap_end
(rt_ubase_t)mem + SIZEOF_STRUCT_MEM + size <= (rt_ubase_t)small_mem->heap_end
rt_object_get_type(&small_mem->parent.parent) == RT_Object_Class_Memory
(thread->stat & RT_THREAD_SUSPEND_MASK) == RT_THREAD_SUSPEND_MASK
rt_object_get_type((rt_object_t)thread) == RT_Object_Class_Thread
reverse_detooth_set
reverse_detooth_set[en]
read_motor_info
read_motor_info
hardware_init_info
hardware_init_info
preloading
preloading
switch_mode_test
switch_mode_test[mode]
print_buffer_aging
print_buffer_aging
aging_photoelectric_CMD
aging_photoelectric_CMD[first_id][end_id]
lcd[option]
reset
reset immediate
bl24cxx
bl24cxx
odo_mag
odo_mag[option]
ahtxx_read
ahtxx_read
temp_humi
temp_humi [temp] [humi]
detooth_test
detooth_test[id][dir]
rfid
rfid[option]
mbd_test
mbd_test[motor_id]
task_test
task_test[out / back / out + back][motor_id][times]
device_uid
get device uid
check_state
check_state
at8236_test
at8236_test[id][speed][dir]
at8236_adc_test
at8236_adc_test[id]
at8236_adc_timer
at8236_adc_timer
at8236_pid_set
at8236_pid_set outkp outki outkd bkp bki bkd
led_test
led_test[white / red][id][close / open / breath / double_shan]
MS3791_test
MS3791_test[dir][speed]
MS3791_time_test
MS3791_time_test[dir][speed][time_ms]
MS3791_read_fg
MS3791_read_fg
ms3791_pid_set
ms3791_pid_set kp ki kd
buffer_test
buffer_test
box_aging_mode_test
box_aging_mode_test[first_channel][end_channel][times]
adc [option]
pin [option]
pwm [option]
help
RT-Thread shell help.
List threads in the system.
free
Show the memory usage in the system.
clear
clear the terminal screen
version
show RT-Thread version information
list
list objects
```