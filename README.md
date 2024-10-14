# nRF52 Bootloader

## Features

- DFU over Serial and OTA ( application, Bootloader+SD )
- Self-upgradable via Serial and OTA
- DFU using UF2 (https://github.com/Microsoft/uf2) (application only)
- Auto-enter DFU briefly on startup for DTR auto-reset trick (832 only)

## How to use

There are two pins, `DFU` and `FRST` that bootloader will check upon reset/power:

- `Double Reset` Reset twice within 500 ms will enter DFU with UF2 and CDC support (only works with nRF52840)
- `DFU = LOW` and `FRST = HIGH`: Enter bootloader with UF2 and CDC support
- `DFU = LOW` and `FRST = LOW`: Enter bootloader with OTA, to upgrade with a mobile application such as Nordic nrfConnect/Toolbox
- `DFU = HIGH` and `FRST = HIGH`: Go to application code if it is present, otherwise enter DFU with UF2
- The `GPREGRET` register can also be set to force the bootloader can enter any of above modes (plus a CDC-only mode for Arduino).
`GPREGRET` is set by the application before performing a soft reset.

```c
#include "nrf_nvic.h"
void reset_to_uf2(void) {
  NRF_POWER->GPREGRET = 0x57; // 0xA8 OTA, 0x4e Serial
  NVIC_SystemReset();         // or sd_nvic_SystemReset();
}
```

Please check the board definition for details.

### Making your own UF2

To create your own UF2 DFU update image, simply use the [Python conversion script](https://github.com/Microsoft/uf2/blob/master/utils/uf2conv.py) on a .bin file or .hex file, specifying the family as **0xADA52840** (nRF52840)

```
nRF52840
uf2conv.py firmware.hex -c -f 0xADA52840
```

If using a .bin file with the conversion script you must specify application address with the -b switch, this address depend on the SoftDevice size/version e.g S140 v6 is 0x26000, v7 is 0x27000

```
nRF52840
uf2conv.py firmware.bin -c -b 0x26000 -f 0xADA52840
```

To create a UF2 image for bootloader from a .hex file using separated family of **0xd663823c**

```
uf2conv.py bootloader.hex -c -f 0xd663823c
```

### What's the difference between bin and hex?

## Burn & Upgrade with pre-built binaries

You can burn and/or upgrade the bootloader with either a J-link or DFU (serial) to a specific pre-built binary version

Note: The bootloader can be downgraded. Since the binary release is a merged version of
both bootloader and the Nordic SoftDevice, you can freely upgrade/downgrade to any version you like.

## How to compile and build

Use J-Link to "unbrick" the device.

### Build

```
git clone this repo
cd bootloader
git submodule update --init
make all
```

### Flash

To flash the bootloader (without softdevice/mbr) using JLink:

```
make flash
```

If you are using pyocd as debugger, add `FLASHER=pyocd` to make command:

```
make FLASHER=pyocd flash
```

To upgrade the bootloader using DFU Serial via port /dev/ttyACM0

```
make SERIAL=/dev/ttyACM0 flash-dfu
```

```
make flash-sd  # flash SoftDevice (will erase chip)
make flash-mbr # flash mbr only
```

# 说明

make all 之后，写入 opensk.uf2


`DCONFIG_GPIO_AS_PINRESET` 被定义的时候， `P0.18` 会被当作 `RESET`

这个 `P0.18` 不能修改成别的 PIN

