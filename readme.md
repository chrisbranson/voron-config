# Reference

https://canbus.esoterical.online/

# Update Klipper, Moonraker etc. using Kiauh

```
./kiauh/kiauh.sh
```

# Build firmware for Octopus controller board

```
cd ~/klipper
make menuconfig
```

Configure: -

* Micro-controller Architecture (STMicroelectronics STM32)
* Processor model (STM32F446)
* Bootloader offset (32KiB bootloader)
* Clock Reference (12 MHz crystal)
* Communication interface (USB (on PA11/PA12))
* [*] Optimize stepper code for 'step on both edges'
* () GPI0 pins to set at micro-controller startup

```
make
```

Put Octopus board into bootloader mode by pressing the reset button twice (fast double click), verify device is in DFU mode: -

```
ls /dev/serial/by-id/
```

And make a note of the file name shown, e.g. "usb-katapult_stm32f446xx_3D0032000A5053424E363420-if00"

Flash the new klipper firmware: -

```
python3 ~/katapult/scripts/flashtool.py -f ~/klipper/out/klipper.bin -d /dev/serial/by-id/usb-katapult_stm32f446xx_3D0032000A5053424E363420-if00
```

# Build firmware for SB2209 (RP2040 based) toolhead board

```
cd ~/klipper
make menuconfig
```

Configure: -

* Micro-controller Architecture (Raspberry Pi RP2040/RP235x)-->
* Processor model (rp2040)
* Bootloader offset (16KiB bootloader)|
* Communication Interface (CAN bus) --->
* (4) CAN RX gpio number
* (5) CAN TX gpio number
* (1000000) CAN bus speed
* [*] Optimize stepper code for 'step on both edges'
* () GPIO pins to set at micro-controller startup

```
make
```

Flash the new klipper firmware: -

```
~/katapult/scripts/flashtool.py -i can0 -u 0e6a23032289 -f ~/klipper/out/klipper.bin -v
```

# Update Cartographer firmware

```
cd cartographer_firmware/
git pull
./fw_update.sh
```
