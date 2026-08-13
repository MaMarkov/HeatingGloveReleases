# Flashing GloveHeater firmware

Each `.bin` here is a **complete** image — bootloader, partition table and
application in one file. There is nothing else to download and no toolchain to
install.

## Which file goes on which device

| File | Device |
|---|---|
| `pole_right-*.bin` | The **right-hand pole** |
| `pole_left-*.bin` | The **left-hand pole** |
| `glove_right-*.bin` | The **right-hand glove** |
| `glove_left-*.bin` | The **left-hand glove** |

**Left and right are not interchangeable.** A device only talks to its own
side's partner, so a left glove flashed onto the right one will sit searching
for a pole that never answers, with no error to explain why. The right pole's
button also turns the temperature *up* while the left pole's turns it *down*.

If you are unsure which you have, flash one and watch the serial output at
115200 baud — the first line says `Starting Pole R`, `Starting Glove L` and so
on.

## Easiest: flash from a browser

No installation. Needs Chrome or Edge (Firefox and Safari cannot talk to serial
ports).

1. Plug the device in by USB.
2. Go to **https://espressif.github.io/esptool-js/**
3. Leave the baud rate at 115200 and press **Connect**, then choose the device's
   COM port from the list.
4. Press **Add File**, choose the `.bin` for this device, and set its
   **Flash Address to `0x0`**.
5. Press **Program**, and wait for `Leaving... Hard resetting`.

`0x0` is important. These images already contain the bootloader at whatever
offset the chip expects, so the whole file goes at zero regardless of device.

## Alternative: esptool on the command line

If you already have Python:

```bash
pip install esptool

# A pole
esptool.py --chip esp32s3 --port COM4 --baud 460800 write_flash 0x0 pole_right-0.7.0.bin

# A glove
esptool.py --chip esp32 --port COM4 --baud 460800 write_flash 0x0 glove_right-0.7.0.bin
```

Note the different `--chip`: the poles are ESP32-**S3**, the gloves are plain
ESP32. Get it wrong and esptool refuses rather than doing damage.

## If it will not connect

- **Nothing in the port list** — the USB cable may be charge-only. Try another.
- **"Failed to connect"** — hold the BOOT button, press and release RESET, then
  release BOOT, and try again.
- **Flashes but does nothing** — check you used address `0x0` and not `0x10000`.
  `0x10000` is where the application alone would go, and these files are not
  that.

## Checking it worked

Open a serial monitor at **115200 baud**. Within a couple of seconds you should
see something like:

```
Starting Pole R, firmware 0.7.0
Pole R: restored setpoints R=31.00C L=31.00C, ring 1.00 rot/s
Pole R: battery 16.88V pack, 4.221V/cell, 100%
...
Setup complete.
```

The version on that first line is the one in the filename. If they disagree, the
flash did not take.
