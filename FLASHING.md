# Flashing GloveHeater firmware

There are **two files per device** and they are not interchangeable.

| Suffix | What it is | How it goes on |
|---|---|---|
| `-full.bin` | Bootloader, partition table and application in one image | Over a USB cable, at address `0x0` |
| `-ota.bin` | The application alone | Wirelessly, from the phone app |

Use `-full.bin` for a blank board, a board of unknown state, or any device that
will not boot. Use `-ota.bin` for a device that is already working. The phone
app refuses a `-full.bin`, because writing one into an over-the-air update slot
would put a bootloader in the middle of the application partition.

## Which file goes on which device

| File | Device |
|---|---|
| `pole_right-*` | The **right-hand pole** |
| `pole_left-*` | The **left-hand pole** |
| `glove_right-*` | The **right-hand glove** |
| `glove_left-*` | The **left-hand glove** |

**Left and right are not interchangeable.** A device only talks to its own
side's partner, so a left glove flashed onto the right one will sit searching
for a pole that never answers, with no error to explain why. The right pole's
button also turns the temperature *up* while the left pole's turns it *down*.

Every image says what it is, in plain text inside the file itself — the phone
app reads that rather than trusting the filename. If you are unsure which device
you have in front of you, flash one and watch the serial output at 115200 baud:
the first two lines say `Starting Pole R, firmware 0.8.0` and
`Image GHFW|0.8.0|pole_right|`.

---

# Over the air, from the phone app

The quickest route for a device that already works, and the only one that needs
no cable. **Nothing has to be downloaded or copied anywhere** — the app fetches
the firmware itself.

1. Open the app and tap the **update icon** in the top bar.
2. Each device has a row showing two things: the firmware it is running, and the
   newest firmware published. A device that has never been seen shows *not
   seen* — a glove has to be attached to its pole, and that pole switched on,
   before the app can know anything about it.
3. Press **Update to x.y.z** on the device you want.

That is the whole thing. The phone will ask for permission to join a WiFi
network — that is the device itself, which drops Bluetooth and puts up its own
access point for the transfer. Allow it.

The phone needs internet when you press the button, but only for the first few
seconds while the image downloads; after that it has none for a minute or two.
Keep the phone close and the screen on.

Only one device updates at a time, and only the one you chose.

## Publishing a firmware

The app reads
<https://github.com/MaMarkov/HeatingGloveReleases/tree/main/OTA>. Drop the
`-ota.bin` files there and they appear in the app — there is no manifest or
index to update, because the filenames carry the version and the device.

Keep the `<target>-<version>-ota.bin` naming. Anything else in that folder is
ignored, `-full.bin` included, which is deliberate: a full-flash image sent over
the air would write a bootloader into the middle of the application partition.

To check what the app will see:

```bash
cd Mobile/gloveheater_app
dart run tool/check_releases.dart
```

---

# Over a cable

## Easiest: flash from a browser

No installation. Needs Chrome or Edge (Firefox and Safari cannot talk to serial
ports).

1. Plug the device in by USB.
2. Go to **https://espressif.github.io/esptool-js/**
3. Leave the baud rate at 115200 and press **Connect**, then choose the device's
   COM port from the list.
4. Press **Add File**, choose the `-full.bin` for this device, and set its
   **Flash Address to `0x0`**.
5. Press **Program**, and wait for `Leaving... Hard resetting`.

`0x0` is important. These images already contain the bootloader at whatever
offset the chip expects, so the whole file goes at zero regardless of device.

## Alternative: esptool on the command line

If you already have Python:

```bash
pip install esptool

# A pole
esptool.py --chip esp32s3 --port COM4 --baud 460800 write_flash 0x0 pole_right-0.8.0-full.bin

# A glove
esptool.py --chip esp32 --port COM4 --baud 460800 write_flash 0x0 glove_right-0.8.0-full.bin
```

Note the different `--chip`: the poles are ESP32-**S3**, the gloves are plain
ESP32. Get it wrong and esptool refuses rather than doing damage.

## If it will not connect

- **Nothing in the port list** — the USB cable may be charge-only. Try another.
- **"Failed to connect"** — hold the BOOT button, press and release RESET, then
  release BOOT, and try again.
- **Flashes but does nothing** — check you used the `-full.bin` at address `0x0`.
  The `-ota.bin` at `0x0` produces exactly this: it flashes without complaint and
  then does nothing, because there is no bootloader in it.

---

# Checking it worked

Open a serial monitor at **115200 baud**. Within a couple of seconds you should
see something like:

```
Starting Pole R, firmware 0.8.0
Image GHFW|0.8.0|pole_right|
Pole R: restored setpoints R=31.00C L=31.00C, ring 1.00 rot/s
Pole R: battery 16.88V pack, 4.221V/cell, 100%
...
Setup complete.
```

The version on those first lines is the one in the filename. If they disagree,
the flash did not take.

The app shows the same thing without a cable: connect to a pole, open the update
screen, and every device it can see is listed with the firmware it is running.
