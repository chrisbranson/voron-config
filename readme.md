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

# Build firmware for Hydra board

```
cd ~/klipper
make menuconfig
```

Configure: -

* Micro-controller Architecture (STMicroelectronics STM32)
* Processor model (STM32F446)
* Bootloader offset (32KiB bootloader)
* Clock Reference (8 MHz crystal)
* Communication interface (USB (on PA11/PA12))
* [*] Optimize stepper code for 'step on both edges'
* (PB12) GPI0 pins to set at micro-controller startup

```
make
```

Put Hydra board into bootloader mode by pressing the reset button twice (fast double click), verify device is in DFU mode: -

```
ls /dev/serial/by-id/
```

And make a note of the file name shown, e.g. "usb-katapult_stm32f446xx_580050000F51363139343332-if00"

Flash the new klipper firmware: -

```
python3 ~/katapult/scripts/flashtool.py -f ~/klipper/out/klipper.bin -d /dev/serial/by-id/usb-katapult_stm32f446xx_580050000F51363139343332-if00
```

# Update Cartographer firmware

```
cd ~/cartographer_firmware/
git pull
./fw_update.sh
```

# Flashing from DFU mode (firmware corrupted)

## Octopus

Turn off power and place the jumper over the "boot" pins. Hold down the reset button (next to USB C connector) and power back on.

Run `lsusb` and you should see "ID 0483:df11 STMicroelectronics STM Device in DFU Mode"

```
cd ~/katapult
make clean
make menuconfig
```

Configure: -

* Micro-controller Architecture (STMicroelectronics STM32)
* Processor model (STM32F446)
* Build Katapult deployment application (Do not build)
* Clock Reference (12 MHz crystal)
* Communication interface (USB (on PA11/PA12))
* Application start offset (32KiB offset)
* USB ids (leave empty)
* Build Optimization Override (Size (-Os))
* () GPIO pins to set on bootloader entry
* [*] Support bootloader entry on rapid double click of reset button
* [ ] Enable bootloader entry on button (or gpio) state
* [*] Enable Status LED
* (PA13)  Status LED GPIO Pin

```
make
```

Flash to Octopus using dfu-util: -

```
sudo dfu-util -R -a 0 -s 0x08000000:mass-erase:force:leave -D ~/katapult/out/katapult.bin -d 0483:df11
```

Power off, remove the jumper from the "boot" pins and power back on. Follow the instructions above to flash the Klipper firmware.

## U2C

Turn off power, press and hold the "boot" button on the U2C, power back on. Wait a few seconds and release the "boot" button.

Run `lsusb` and you should see "ID 0483:df11 STMicroelectronics STM Device in DFU Mode"

Flash the Bigtreetech supplied firmware to the U2C using dfu-util: -

```
sudo dfu-util -d 0483:df11 -c 1 -i 0 -a 0 -s 0x08000000 -D ~/u2c/U2C_V2_STM32G0B1.bin
```

Power cycle the printer to complete.
