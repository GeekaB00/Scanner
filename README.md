# Geekaboo Multi Scanner

A pocket radio scanner for the **ESP32 "Cheap Yellow Display"**, in 2.8" and 3.5" versions. It shows nearby Bluetooth devices, Wi-Fi networks, crowded Wi-Fi channels, Bluetooth trackers, and MeshCore radios on a touch screen, and it comes with **Boo**, a little radio pet that lives on the airwaves.

<p align="center">
  <img src="boot.gif" width="240" alt="Geek Scanner boot screen with animated radar">
</p>

**Version:** 3.0.1 · **Author:** Geekaboo

---

## Contents

- [Features](#features)
- [Hardware](#hardware)
- [Install the firmware](#install-the-firmware)
- [Using the scanner](#using-the-scanner)
  - [Boot screen](#boot-screen)
  - [Boo, the radio pet](#boo-the-radio-pet)
  - [Screen layout](#screen-layout)
  - [BLE tab](#ble-tab)
  - [WIFI tab](#wifi-tab)
  - [CHAN tab](#chan-tab)
  - [TAGS tab](#tags-tab)
  - [MESH tab](#mesh-tab)
  - [Watchlist](#watchlist)
  - [Detail page and signal meter](#detail-page-and-signal-meter)
  - [Full-screen radar](#full-screen-radar)
  - [Other Geekaboo scanners](#other-geekaboo-scanners)
  - [Settings](#settings)
  - [Touch calibration](#touch-calibration)
  - [LED colours](#led-colours)
- [Troubleshooting](#troubleshooting)
- [Build from source](#build-from-source)
- [Change the boot image](#change-the-boot-image)
- [Host your own web installer](#host-your-own-web-installer)
- [Repository layout](#repository-layout)
- [Limitations](#limitations)
- [Changelog](#changelog)

---

## Features

| Tab | What it does |
|---|---|
| **BLE** | Lists nearby Bluetooth Low Energy devices, sorted by signal strength. Unnamed devices show their vendor (Apple, Samsung, Google, …). |
| **WIFI** | Lists 2.4 GHz Wi-Fi networks with name, channel, security type and BSSID. Open networks are marked in orange. |
| **CHAN** | A Wi-Fi channel analyzer. It draws each network as a curve over channels 1 to 13 and suggests the least crowded of channels 1, 6 and 11. |
| **TAGS** | Detects Bluetooth trackers: Apple Find My (AirTag), Samsung SmartTag, Tile and Google Find Hub. It warns when a tracker has stayed near you for a long time. |
| **MESH** | Finds MeshCore devices: companion radios advertising over Bluetooth, other Nordic UART devices, and MeshCore devices in Wi-Fi update mode. |
| **★** | Your watchlist: devices and networks you pinned, shown as *here* or *gone*, with an alert when that changes. |

The firmware also has:

- **Boo, the radio pet:** a Tamagotchi-style ghost that eats new signals, gets excited in busy places, sleepy when it's quiet, suspicious of trackers following you, and happy when another Geekaboo scanner is near.
- **Full-screen radar:** every device and network as a blip on an animated radar, with distance from the centre showing signal strength.
- **Signal meter and graph:** tap any entry for a large live signal reading plus a graph of the last 60 seconds. Walk around to find where a device is.
- **Portrait or landscape:** one button rotates the screen in 90° steps, and the rotation is remembered after a restart.
- **Settings page:** filters, scan speed, tracker alert time, screen brightness and colour inversion, all saved to flash.
- **Touch calibration:** a short tap-the-crosses routine, so touch lines up on any panel.
- **Scanners see each other:** every Geekaboo scanner sends a small Bluetooth beacon, so other scanners show it (in light blue) and announce when it arrives or leaves.
- **Crash report:** if the firmware ever crashes, the next boot shows where it happened, so a photo of the screen is enough to find the bug.
- **Animated boot screen:** a small radar sweeps on the boot image while it waits for your tap.

---

## Hardware

Three boards are supported. Each has its own firmware file.

| | **ESP32-2432S028** (2.8") | **ESP32-3248S035R** (3.5") | **ESP32-3248S035C** (3.5") |
|---|---|---|---|
| Firmware file | `…-cyd28.bin` | `…-cyd35r.bin` | `…-cyd35c.bin` |
| Screen | 240×320, ILI9341 | 320×480, ST7796 | 320×480, ST7796 |
| Touch | Resistive, XPT2046 on its own SPI bus (CLK 25, MOSI 32, MISO 39, CS 33, IRQ 36) | Resistive, XPT2046 sharing the display bus (CS 33) | **Capacitive**, GT911 on I2C (SDA 33, SCL 32, INT 21, RST 25) |
| Display pins | MISO 12, MOSI 13, SCK 14, CS 15, DC 2 | same | same |
| Backlight | pin 21 | pin 27 | pin 27 |
| Tested on hardware | **Yes** (2-USB version) | Not confirmed yet | **Yes** |

All three have the RGB LED on pins 4 / 16 / 17 (lit when the pin is low).

**R or C?** Look at the back of the 3.5" board, or at the screen: the **C** (capacitive) version has a smooth glass front that reacts to a light touch, like a phone. The **R** (resistive) version has a slightly soft plastic surface that needs a firm press.

You also need a good USB cable and port, because Wi-Fi and Bluetooth together draw a lot of current. No soldering or extra parts are needed.

---

## Install the firmware

### Option 1: Web installer (easiest)

You need **Chrome or Edge on a desktop computer**. Safari and Firefox don't support Web Serial.

1. Connect the CYD with a USB **data** cable.
2. Open the project's installer page (see [Host your own web installer](#host-your-own-web-installer)), or go to **[web.esphome.io](https://web.esphome.io)**.
3. Click **Connect** (or **FLASH FIRMWARE**) and pick the serial port. It's usually called *CH340* or *CP210x*.
4. On the installer page, pick your board first. On web.esphome.io, choose **Install** and select the file for your board from `firmware/` (see [Hardware](#hardware)).
5. When it's done, press **RST** on the board.

If the browser can't connect, hold **BOOT**, tap **RST**, release **BOOT**, then try again.

### Option 2: esptool (command line)

```bash
pip install esptool
esptool --chip esp32 --port /dev/ttyUSB0 --baud 921600 write-flash 0x0 firmware/geekaboo-scanner-v3.0.1-cyd28.bin
```

Use the file for your board (`-cyd28`, `-cyd35r` or `-cyd35c`). On Windows the port looks like `COM5`. On macOS it's `/dev/cu.usbserial-…`. Older esptool versions use `esptool.py` and `write_flash`.

The `.bin` is a **merged image** (bootloader, partition table and app in one file), so it always goes to address **0x0**.

### Option 3: Arduino IDE

See [Build from source](#build-from-source).

---

## Using the scanner

### Boot screen

After power-on the Geek Scanner image appears (the same way up as the scanner: upside down only if you rotated the scanner upside down), with the version number at the bottom and a small radar sweeping in the bottom-left corner. Wi-Fi and Bluetooth start in the background while it's showing. The RGB LED pulses magenta while it waits.

**Tap the screen to start.** That first tap only dismisses the image, so it never presses a button underneath.

- **3.5" boards:** on the very first start, [touch calibration](#touch-calibration) runs straight after this tap.
- **Hold your finger on the boot screen for 3 seconds** to run touch calibration on any board. A magenta bar says *release to calibrate touch*.

After the tap you meet [Boo](#boo-the-radio-pet). To go straight to the scanner lists instead, switch off **Settings → Start with Boo**.

If a red line appears at the top of the boot screen, or a red **CRASH REPORT** box, the previous restart was unexpected. See [Troubleshooting](#troubleshooting).

### Boo, the radio pet

<p align="center">
  <img src="boo-moods.png" width="720" alt="Boo's ten moods">
</p>

Boo is a little ghost with an antenna. It lives on the radio signals the scanner picks up: its mood, its hunger and its level all come from what is on the air around you. While Boo is on screen the scanner keeps running (mostly Bluetooth, with a Wi-Fi scan every 4th round), and the antenna blinks and sends out radio waves.

```
┌──────────────────────────────┐
│ BOO  Lv3 Listener      happy │  header: level and mood
│ ┌──────────────────────────┐ │
│ │ New signal: Phone        │ │  speech bubble
│ └──────────────────────────┘ │
│            ( o o )           │  Boo (tap to pat)
│ FOOD ▇▇▇▇▇▁▁   JOY ▇▇▇▇▇▇▁▁  │
│ BLE 14 WIFI 6 TAGS 1 PALS 0  │  what's around right now
│ Lv3 Listener  112 signals    │  level, signals sniffed, progress line
│ [ SCANNER ]     [  RADAR  ]  │
└──────────────────────────────┘
```

**Moods.** Boo picks the first mood that applies, from top to bottom:

| Mood | When |
|---|---|
| Loved | You just patted Boo (tap on Boo). Heart eyes. |
| Surprised | A new device appeared very close (stronger than −45 dBm) |
| Friend | Another [Geekaboo scanner](#other-geekaboo-scanners) just arrived. Boo waves and says hi by name. |
| Suspicious | A tracker has been near you longer than the *Tracker alert* time |
| Hungry | FOOD is below 20 |
| Curious | A new device or network just appeared. Boo says its name. |
| Excited | 15 or more Bluetooth devices around |
| Sleepy | 2 or fewer Bluetooth devices for 2 minutes |
| Bored | JOY is below 25 |
| Happy | Everything else |

**FOOD** slowly goes down (about 1 point every 90 seconds). Boo eats *new* signals: every device or network it hasn't seen before adds 2 points. Hanging around in the same room won't feed it, so take it for a walk: a train station, a shop or a busy street is a feast.

**JOY** slowly goes down too (about 1 point every 2 minutes). Tap Boo to pat it (+8 for the first five pats in a minute, then +1). Another Geekaboo scanner nearby cheers Boo up; a tracker lurking near you spoils the mood.

**Levels** come from the total number of signals Boo has ever sniffed: Hatchling, Sniffer (25), Listener (75), Scout (150), Tracker (300), Hunter (600), Phantom (1200), Specter (2500), Wraith (5000) and Legend (10000). The thin line under the level shows the progress to the next one.

- **SCANNER** opens the scanner lists. **Tap the header** of the scanner to come back to Boo.
- **RADAR** opens the [full-screen radar](#full-screen-radar); its BACK button returns to Boo.
- Boo is always shown in portrait. If you rotated the scanner to landscape, that rotation comes back when you leave Boo; upside-down portrait is kept.
- If memory runs short, Boo skips Wi-Fi scans for a while and the panel shows `WIFI -`. Bluetooth keeps working.
- Boo keeps living while you use the scanner: it still eats new signals and gets hungry.
- FOOD, JOY and the signal count are saved to flash every 5 minutes and when you leave Boo, so they survive a restart. **Factory reset** starts Boo over; *Reset settings* doesn't touch Boo.

### Screen layout

```
┌──────────────────────────────┐
│ Geekaboo v3.0.1     12 dev ● │  header: name, version, count, scan dot
├────┬────┬────┬────┬────┬─────┤
│BLE │WIFI│CHAN│TAGS│MESH│  ★  │  tabs (★ = watchlist)
├────┴────┴────┴────┴────┴─────┤
│▌Device name        ▇▇▇▇▁ -58 │  ▌ yellow edge = on your watchlist
│ aa:bb:cc:dd:ee:ff            │  list rows: tap = details, hold = watch
│ …                            │
├────┬────┬────┬────┬────┬─────┤
│ ▲  │ ❚❚ │ ⟳  │ ◎  │ ⚙  │  ▼  │  footer (◎ = radar)
└────┴────┴────┴────┴────┴─────┘
```

**Header**
- **Tap the header** to go to [Boo](#boo-the-radio-pet).
- Left: **Geekaboo** and the firmware version.
- Right: the number of items in the current tab, or **PAUSED**.
- The dot on the far right is lit while a scan is running: cyan for Bluetooth, green for Wi-Fi.

**Footer buttons**

| Button | Action |
|---|---|
| ▲ | Scroll up one page |
| ❚❚ / ▶ | Pause or resume scanning. While paused the list order is frozen, which makes it easier to tap an entry. |
| ⟳ | Rotate the screen 90°. The setting is remembered. |
| ◎ (green) | Open the [full-screen radar](#full-screen-radar) |
| ⚙ | Open [Settings](#settings) |
| ▼ | Scroll down one page |

**Signal colours**

| Colour | Signal |
|---|---|
| Green | Stronger than −60 dBm (close) |
| Yellow | −60 to −80 dBm |
| Red | Weaker than −80 dBm (far away) |
| Grey | Not heard for 10 seconds (stale) |

Entries that haven't been heard for the time set in **Forget after** (default 1 minute) disappear from the list.

**Tap or hold:** a tap registers when you lift your finger. Holding a list row for about 0.7 seconds adds it to (or removes it from) the [watchlist](#watchlist).

### BLE tab

All Bluetooth Low Energy devices nearby, strongest first.

- **Top line:** the device name. If the device sends no name, it shows the vendor from its manufacturer data (for example `<Apple>`), or `<unnamed>`.
- **Bottom line:** the MAC address. Trackers are marked **TAG** in orange. Other Geekaboo scanners are shown in light blue, with `scanner v3.0 3.5C` (version and board).
- **Right side:** a signal bar and the signal strength in dBm.

The list holds up to 48 devices. When it's full, the device heard longest ago is replaced.

### WIFI tab

All 2.4 GHz Wi-Fi networks nearby, strongest first.

- **Top line:** the network name (SSID), or `<hidden>`.
- **Bottom line:** channel, security type and BSSID. Open networks (no password) are shown in orange.

Security types: `OPEN`, `WEP`, `WPA`, `WPA2`, `WPA/2`, `WPA2-E` (enterprise), `WPA3`, `WPA2/3`.

The list holds up to 48 networks. The ESP32 only has a 2.4 GHz radio, so 5 GHz and 6 GHz networks don't appear.

When the list (or the CHAN graph) is empty, it says why:

| Message | Meaning |
|---|---|
| *scanning Wi-Fi...* | The first scan hasn't finished yet |
| *no Wi-Fi networks found* | The scan worked, but nothing is in range |
| *Wi-Fi skipped: low memory (11k)* | Memory was too short for a scan; it tries again on the next round |
| *Wi-Fi scan failed (code), retrying* | The radio refused the scan; the Wi-Fi part restarts automatically |

### CHAN tab

A channel graph of the Wi-Fi networks from the WIFI scan.

- Each network is drawn as a curve centred on its channel. The height of the curve is its signal strength.
- The five strongest networks are labelled with their name.
- Channels 1, 6 and 11 are highlighted, because these are the channels that don't overlap.
- **best channel:** of 1, 6 and 11, the one with the least nearby traffic. It's a good pick for your own router or access point.

### TAGS tab

Bluetooth trackers nearby:

| Tracker | How it's recognised |
|---|---|
| Apple Find My (AirTag and other Find My devices) | Apple manufacturer data, type 0x12 |
| Samsung SmartTag | Service 0xFD5A |
| Tile | Service 0xFEED / 0xFEEC |
| Google Find Hub | Eddystone service 0xFEAA, frame type 0x40/0x41 |

Each row shows the tracker type, the MAC address, and **how long it has been near you**.

- When a new tracker is detected, the LED flashes red.
- When a tracker has been nearby longer than the **Tracker alert** time (default 10 minutes), its row turns **magenta**.

> **About the timer:** most trackers change their Bluetooth address regularly for privacy. When that happens the scanner sees a "new" device and the timer starts again. Treat the time as an indication, not proof. If you think you're being tracked, use the official tools as well: *Find My* on iPhone, *Unknown tracker alerts* on Android, or the SmartThings and Tile apps.

### MESH tab

[MeshCore](https://github.com/meshcore-dev/MeshCore) devices that can be seen over Bluetooth or Wi-Fi:

| Shown as | Meaning |
|---|---|
| **Node name** in magenta, `companion` | A MeshCore companion radio advertising as `MeshCore-<node name>`. Only the node name is shown. |
| `<name>` or `<NUS device>`, `UART?` | A device with the Nordic UART service. MeshCore uses this service, but so do many other gadgets, so treat these as *possible* matches. |
| `MeshCore-…` in orange, `Wi-Fi OTA` | A MeshCore device in Wi-Fi update mode (its `MeshCore-OTA` access point). |

Companions come first, then UART devices, then Wi-Fi entries. In this tab the scanner mostly scans Bluetooth, with a Wi-Fi scan every 4th round to catch OTA access points.

> A companion radio **stops advertising while your phone app is connected to it**. Disconnect the app to see it here. The CYD has no LoRa radio, so it can't hear the mesh itself; see [Limitations](#limitations).

### Watchlist

Pin devices or networks you care about, such as your keys tag, a MeshCore node or your home network, and the scanner tells you when they come and go.

**Adding and removing**
- **Hold** a row in BLE, WIFI, TAGS or MESH for about 0.7 seconds, or tap **WATCH** on a [detail page](#detail-page-and-signal-meter).
- To remove: hold the row again (in any tab, including ★), or tap **UNWATCH** on the detail page.
- Up to 8 items. The list is saved in flash and kept after restarts.

**The ★ tab** shows each item with its status:

| Status | Meaning |
|---|---|
| **here** (yellow, with signal bar) | Heard in the last 60 seconds |
| **gone 3m 10s** (magenta) | Not heard for 60 seconds, and for how long |
| **not seen** | Never heard since the board started |
| **waiting...** | Just added, no scan of its radio yet |

The header shows how many are here, for example `2/3 here`. In the other tabs, watched items move to the top and get a **yellow edge**.

**Alerts:** when a watched item comes back or goes away, a banner shows in the header for 4 seconds (*"Keys is here"* in green, *"Keys is gone"* in magenta) and the LED blinks yellow.

An item is only marked gone after its own radio has actually scanned: at least 3 passes with nothing heard. So a watched Wi-Fi network doesn't turn "gone" just because you're on the BLE tab, where only Bluetooth is scanned. The ★ tab scans both radios.

> The watchlist follows a device by its address. That works for devices with a fixed address (Tile, MeshCore nodes, ESP32 gadgets, Wi-Fi networks), but **not for phones and AirTags**, which change their address regularly for privacy.

### Detail page and signal meter

Tap any row in the BLE, WIFI, TAGS, MESH or ★ tab to open its detail page.

- **Top:** a large signal reading in dBm with a full-width bar. It updates twice a second. It shows **LOST** when the device hasn't been heard for 10 seconds.
- **Signal graph:** the last 60 seconds of signal, one point per second, with the highest value labelled *peak*. Gaps mean the device wasn't heard.
- **Bluetooth devices:** MAC, name, tracker type, MeshCore type, vendor (company ID), TX power, service UUID, how often and how long ago it was seen, how long it has been nearby, and raw manufacturer data.
- **Wi-Fi networks:** SSID, BSSID, channel, security, how often and how long ago it was seen, and how long it has been known.

**Finding a device:** open its detail page and walk around slowly. The number goes up (towards −30) as you get closer. Bluetooth signal strength jumps around a lot, so pause for a few seconds at each spot. Your body also blocks the signal, so turn around slowly too.

- **WATCH / UNWATCH** button at the bottom: adds this device to the [watchlist](#watchlist) or removes it.

Tap anywhere else to go back.

### Full-screen radar

Tap the green ◎ button in the footer. Every device and network nearby appears as a blip on an animated radar.

| Blip colour | Meaning |
|---|---|
| White | Bluetooth device |
| Green | Wi-Fi network |
| Orange | Tracker |
| Magenta | MeshCore device |
| Light blue | Another Geekaboo scanner (with expanding rings for 10 s when it just arrived) |
| Yellow (bigger) | On your watchlist |
| Grey | Not heard for 10 seconds |

- **Distance from the centre is real:** stronger signal means closer to the centre. The rings mark −50, −70 and −90 dBm.
- **The angle is not a direction.** A single antenna can measure how strong a signal is, but not where it comes from. Each device gets a fixed spot on the circle (worked out from its address), so it stays in the same place and you can follow it.
- The beam sweeps once every 2.5 seconds; blips flare up as it passes and then fade.
- **Tap a blip** to see its name, signal and address in the panel. **Tap it again** to open its detail page (graph, WATCH); BACK on that page returns to the radar.
- The panel shows how many devices of each type are around, and `mem …k` (free memory).
- **BACK** (or tapping the header) returns to the list, or to Boo if you opened the radar from Boo.

While the radar is open it scans mostly Bluetooth, with a Wi-Fi scan every 4th round. Watch alerts still show in the header.

### Other Geekaboo scanners

Every scanner sends a small Bluetooth **beacon** called **Geekaboo-XXXX**. XXXX is a code unique to each board (taken from its chip); your own code is shown in **Settings → Visible**. Other scanners pick it up and show it:

- **BLE list:** in light blue, with `scanner v3.0 3.5C` (firmware version and board).
- **Radar:** a light-blue blip.
- **Detail page:** a *Scanner* line, for example *Geekaboo v3.0, 3.5C board*. The signal meter and graph help you find the other scanner.
- **Watchlist:** hold its row to watch it, like any other device.

**Scanner alerts:** when another scanner comes into range, the header shows *"Scanner Geekaboo-B7C1 (3.5C) nearby"* in light blue, the LED blinks light blue three times, and its radar blip gets expanding rings for 10 seconds. When it hasn't been heard for 60 seconds you get *"Geekaboo-B7C1 left"*. Scanners that are already nearby when you switch on are announced too. Both boards detect each other, so both show a message.

| Setting | What it does |
|---|---|
| **Visible** | Turns your own beacon on or off. Off = other scanners (and phones) can't see this one. |
| **Scanner alerts** | Turns the messages, LED blinks and rings off; other scanners still appear in the lists and on the radar. |

The beacon is a non-connectable advertisement: nobody can connect to the scanner through it, and it uses less than 1 KB of memory. Phones and other Bluetooth scanners nearby can see the name *Geekaboo-…* too while Visible is on.

### Settings

Tap ⚙ in the footer. Tap a setting to step to its next value. Changes are saved right away and survive a restart. When the list doesn't fit on one screen (2.8" board, and landscape), use ▲ and ▼ next to BACK to scroll.

The header shows the memory: `mem 52k  low 23k` is the free memory now and the lowest it has been since the board started. That's useful when reporting a problem.

| Setting | Values | Default | What it does |
|---|---|---|---|
| Min signal | off, −90, −80, −70 dBm | off | Hides devices and networks weaker than this |
| Hide stale | off, on | off | Hides entries not heard for 10 seconds instead of greying them out |
| Scan time | 2, 3, 5 s | 3 s | Length of one Bluetooth scan pass. Longer passes find more devices but update more slowly. |
| Forget after | 30 s, 1 min, 5 min | 1 min | Removes entries not heard for this long |
| Tracker alert | 5, 10, 20, 30 min | 10 min | A tracker nearby longer than this turns magenta |
| Brightness | 25, 50, 75, 100 % | 100 % | Screen backlight |
| Invert colours | off, on | off | Flips the panel's colour mode. Turn on if black looks white and colours look like a negative. |
| Visible | on (with your code), off | on | Sends the [Geekaboo beacon](#other-geekaboo-scanners) so other scanners can see this one |
| Scanner alerts | on, off | on | Messages, LED and radar rings when another scanner arrives or leaves |
| Start with Boo | on, off | on | Show [Boo](#boo-the-radio-pet) after the boot screen. Off = start in the scanner lists. |
| Calibrate touch | start | | Runs [touch calibration](#touch-calibration) |
| Reset settings | tap twice | | Puts all settings above back to their defaults. The watchlist, touch calibration and Boo are kept. |
| Restart | tap twice | | Reboots the board |
| Factory reset | tap twice | | Erases **everything** saved (settings, watchlist, touch calibration, rotation, Boo) and restarts |

The last three need a **second tap within 3 seconds**: the first tap turns the button red with *tap again*.

Tap **BACK** at the bottom to return to the scanner.

### Touch calibration

Calibration lines up the touch panel with the screen. It runs by itself on the first start of a **3.5"** board. On any board you can run it from **Settings → Calibrate touch**, or by holding a finger on the boot screen for 3 seconds.

1. A cyan cross appears in the **top-left** corner. Tap its centre, ideally with a fingernail or stylus.
2. Do the same for the crosses in the **top-right** and **bottom-left** corners.
3. A green **check** cross appears bottom-right. Tap it.
   - If the tap lands within about 30 pixels, you see **saved** and the scanner continues.
   - If not, it says *not accurate enough* and starts again.

The result is saved to flash and kept after restarts and setting resets. It works out by itself whether the panel's axes are swapped or mirrored, so it fixes "touch is upside down or sideways" as well as small offsets. If nobody touches the screen for 60 seconds, calibration stops and keeps the previous values.

The 2.8" board works out of the box with built-in values, so it doesn't ask for calibration.

### LED colours

| LED | Meaning |
|---|---|
| Pulsing magenta | Boot screen, waiting for a tap |
| Magenta | Touch calibration running |
| Blue | Bluetooth scan running |
| Green | Wi-Fi scan running |
| Short red flash | New tracker detected |
| Yellow blinks | Watchlist alert (something came or went) |
| Light-blue blinks | Another Geekaboo scanner arrived or left |

---

## Troubleshooting

### The boot screen keeps blinking / the board restarts by itself

The board is restarting in a loop. Look at the **red line at the top of the boot screen**:

| Red line says | Cause | What to do |
|---|---|---|
| `POWER DIP (brownout)` | The USB supply can't deliver enough current when the radios start | Use a short, good-quality data cable and a USB port directly on the computer or a powered hub, or a 5 V / 1 A+ phone charger |
| `CRASH at bluetooth` / `CRASH at wi-fi` | A radio failed to start | Reflash with **Erase device** ticked (web installer) or run `esptool erase-flash` first, then flash again |
| `WATCHDOG at …` | The firmware froze | Same as above; if it keeps happening, open an issue with the full red line |

Serial output at **115200 baud** shows every boot step with the free memory, for example `[boot] bluetooth  free heap …`.

### A red CRASH REPORT box appears on the boot screen

If the firmware crashed, the board saved where it happened, and the next boot shows it: the task name, the address where it stopped (`PC`), the error cause and a backtrace. The same report is printed on serial at 115200 baud.

- **Take a photo of the box** (or copy the serial lines) before you tap; the report is cleared after the tap.
- Open an issue with the photo and the firmware version. The addresses can be turned into exact source lines with the matching build's `.elf` file.
- If you reflash to try a fix, **don't erase the board**, otherwise a stored report is lost.

### Nothing happens when I tap the boot screen

- On the 2.8" and 3.5" **R** boards, tap firmly; the panel is resistive and needs pressure, not a light swipe. A fingernail or stylus works best.
- On the 3.5" **C** board, check the serial output at 115200 baud for `[touch] GT911 found`. If it says `NOT found`, the touch chip didn't answer; power the board off and on again.
- If the screen redraws by itself every few seconds, it's restarting. See the previous section.

### Touches land in the wrong place

Run [touch calibration](#touch-calibration): hold a finger on the boot screen for 3 seconds, or use **Settings → Calibrate touch** if you can still reach it. The serial output shows the raw values and the result (`[cal] …`), which helps when reporting a problem.

### Display looks wrong or garbled

**Colours look like a negative** (black shows as white): turn on **Settings → Invert colours**. No rebuild needed.

**Picture is garbled or the screen stays white:** make sure you flashed the file for your board (`-cyd28`, `-cyd35r` or `-cyd35c`).

The 2.8" board ships with different display panels. The release build uses the ILI9341 driver with inverted colours, which matches most **2-USB** boards. If yours is still garbled, build from source with one of these settings in `User_Setups/CYD28_User_Setup.h`:

| Variant | Setting in `User_Setup.h` | Try when |
|---|---|---|
| **A** (release) | `ILI9341_2_DRIVER`, `TFT_INVERSION_ON`, 40 MHz | Default, most 2-USB boards |
| B | `ST7789_DRIVER`, `TFT_INVERSION_OFF`, `TFT_RGB_ORDER TFT_BGR`, 27 MHz | 2-USB board with ST7789 panel |
| C | `ILI9341_DRIVER`, `TFT_INVERSION_ON`, 27 MHz | A is garbled |
| D | `ILI9341_2_DRIVER`, `TFT_INVERSION_OFF`, 40 MHz | Colours look like a negative (single-USB boards) |

### The browser can't find the board

- Use Chrome or Edge on a desktop. Phones and Safari can't do Web Serial.
- Some USB cables only carry power. Try another cable.
- Install the USB driver for the chip on your board: **CH340** or **CP210x**.
- Use the BOOT + RST trick from [Install the firmware](#install-the-firmware).

---

## Build from source

### Requirements

| Item | Version used |
|---|---|
| Arduino IDE 2.x or `arduino-cli` | arduino-cli 1.3.1 |
| ESP32 board package (Espressif) | **3.3.2** |
| [TFT_eSPI](https://github.com/Bodmer/TFT_eSPI) by Bodmer | 2.5.44 |
| [XPT2046_Touchscreen](https://github.com/PaulStoffregen/XPT2046_Touchscreen) by Paul Stoffregen (2.8" board only) | 1.4 |

### Choose the board

The sketch picks the board from the display config file **`User_Setup.h` in the sketch folder** (`src/Geekaboo_Scanner/`). The repository ships with the 2.8" config. For another board, copy its file over it:

| Board | Copy this file to `src/Geekaboo_Scanner/User_Setup.h` |
|---|---|
| ESP32-2432S028 (2.8") | `User_Setups/CYD28_User_Setup.h` (already in place) |
| ESP32-3248S035R (3.5" resistive) | `User_Setups/CYD35R_User_Setup.h` |
| ESP32-3248S035C (3.5" capacitive) | `User_Setups/CYD35C_User_Setup.h` |

The `User_Setup.h` in the sketch folder takes priority over the one inside the TFT_eSPI library folder, so you don't need to touch the library. If the file doesn't match a known board, the build stops with an error that says what to copy.

### Steps (Arduino IDE)

1. Install the **esp32** board package by Espressif Systems: *Tools → Board → Boards Manager*.
2. Install **TFT_eSPI** and **XPT2046_Touchscreen**: *Tools → Manage Libraries*.
3. [Choose the board](#choose-the-board) by copying its `User_Setup.h` into the sketch folder.
4. Open `src/Geekaboo_Scanner/Geekaboo_Scanner.ino`.
5. Select:
   - **Board:** `ESP32 Dev Module`
   - **Partition Scheme:** `Huge APP (3MB No OTA/1MB SPIFFS)`. This is **required**: the firmware is about 1.8 to 2 MB and doesn't fit the default partition.
6. Click **Upload**.

### Steps (arduino-cli)

```bash
arduino-cli core install esp32:esp32@3.3.2
arduino-cli lib install "TFT_eSPI" "XPT2046_Touchscreen"
# pick the board (here: 3.5" resistive)
cp src/Geekaboo_Scanner/User_Setups/CYD35R_User_Setup.h src/Geekaboo_Scanner/User_Setup.h

arduino-cli compile -b esp32:esp32:esp32:PartitionScheme=huge_app \
  --output-dir build --export-binaries src/Geekaboo_Scanner
arduino-cli upload -b esp32:esp32:esp32:PartitionScheme=huge_app \
  -p /dev/ttyUSB0 --input-dir build src/Geekaboo_Scanner
```

### Make a merged image for the web installer

```bash
esptool --chip esp32 merge-bin -o firmware/geekaboo-scanner-vX.Y.Z.bin \
  --flash-mode dio --flash-freq 80m --flash-size 4MB \
  0x1000  build/Geekaboo_Scanner.ino.bootloader.bin \
  0x8000  build/Geekaboo_Scanner.ino.partitions.bin \
  0xe000  ~/.arduino15/packages/esp32/hardware/esp32/3.3.2/tools/partitions/boot_app0.bin \
  0x10000 build/Geekaboo_Scanner.ino.bin
```

Then update the file name and `"version"` in `manifest.json`.

### Change the version number

The version is defined once, near the top of the sketch:

```cpp
#define FW_VERSION "3.0.1"
```

The boot screen, the header and the serial output all use it.

### Useful constants

| Constant | Default | Meaning |
|---|---|---|
| `STALE_MS` | 10000 | After this many ms an entry is greyed out or shown as LOST |
| `MAX_DEVS` | 48 | Maximum Bluetooth devices kept in memory |
| `MAX_APS` | 48 | Maximum Wi-Fi networks kept in memory |
| `OPT_*` arrays | | The values the Settings page cycles through |

---

## Change the boot image

1. Pick a picture. Portrait works best; it's scaled to fit the screen without cropping, and the empty space is filled with the colour of the picture's edge.
2. Convert it, once per screen size:
   ```bash
   pip install pillow
   # 2.8" board, 240x320
   python3 tools/make_splash.py my_picture.jpg src/Geekaboo_Scanner/splash_image.h
   # 3.5" boards, 320x480
   python3 tools/make_splash.py my_picture.jpg src/Geekaboo_Scanner/splash_image_35.h 320x480
   ```
3. Build and flash again.

To refresh the animated preview at the top of this README after changing the image or version:

```bash
python3 tools/make_gif.py src/Geekaboo_Scanner/splash_image.h docs/boot.gif 3.0.1 2
```

The boot image is drawn **upside down** compared with the default portrait orientation (`tft.setRotation(2)` in `setup()`). Change it to `0` to show it the other way round. The image takes about 150 KB of flash on the 2.8" board and 300 KB on the 3.5" boards.

---

## Host your own web installer

The installer is a static page using [ESP Web Tools](https://esphome.github.io/esp-web-tools/), so it runs on **GitHub Pages**:

1. Push this repository to GitHub.
2. Go to *Settings → Pages* and set the source to **Deploy from a branch**, branch `main`, folder `/ (root)`.
3. After a minute the installer is live at `https://<your-user>.github.io/<repo-name>/`.

Web Serial only works over **HTTPS**. GitHub Pages provides that; opening `index.html` from your own disk won't work.

The page has a board picker; each board has its own manifest (`manifest-cyd28.json`, `manifest-cyd35r.json`, `manifest-cyd35c.json`). To publish a new version, put the new `.bin` files in `firmware/`, then update the file names and versions in the three manifests and the version in `index.html`.

---

## Repository layout

```
.
├── README.md                     this manual
├── index.html                    web installer page with board picker (ESP Web Tools)
├── manifest-cyd28.json           which .bin to flash, per board
├── manifest-cyd35r.json
├── manifest-cyd35c.json
├── .nojekyll                     serve files as-is on GitHub Pages
├── firmware/                     merged images, flash at 0x0
│   ├── geekaboo-scanner-v3.0.1-cyd28.bin
│   ├── geekaboo-scanner-v3.0.1-cyd35r.bin
│   └── geekaboo-scanner-v3.0.1-cyd35c.bin
├── src/
│   └── Geekaboo_Scanner/
│       ├── Geekaboo_Scanner.ino  the firmware (all boards)
│       ├── User_Setup.h          display config that picks the board (2.8" by default)
│       ├── User_Setups/          display configs for each board
│       ├── splash_image.h        boot image, 240×320
│       └── splash_image_35.h     boot image, 320×480
├── tools/
│   ├── make_splash.py            convert a picture into a boot image
│   └── make_gif.py               animated boot screen GIF for the README
└── docs/
    ├── boot.gif                  animated boot screen for this README
    └── boo-moods.png             Boo's moods for this README
```

---

## Limitations

- **2.4 GHz only.** The ESP32 can't see 5 GHz or 6 GHz Wi-Fi.
- **One radio for both.** Bluetooth and Wi-Fi share one antenna, so the scanner switches between them. In the WIFI and CHAN tabs it only scans Wi-Fi; in BLE and TAGS it only scans Bluetooth.
- **Memory is tight.** Bluetooth and Wi-Fi together use most of the ESP32's memory. To leave room, Wi-Fi is only switched on during a Wi-Fi scan (except in the WIFI and CHAN tabs), and a Bluetooth scan briefly waits when memory gets very low.
- **No LoRa.** The MESH tab finds MeshCore devices through Bluetooth and Wi-Fi only. It can't hear mesh packets, repeaters or adverts on 869 MHz.
- **No Bluetooth Classic.** Older devices that only use Classic Bluetooth (some headsets, car kits) don't appear.
- **Rotating addresses.** Phones and trackers change their Bluetooth address for privacy, so one physical device can appear as several entries over time.
- **No direction.** One antenna gives signal strength, not direction. On the radar only the distance from the centre means something.
- **Signal ≠ distance.** dBm gives a rough idea of distance, but walls, bodies and antenna direction have a big effect.

---

## Changelog

| Version | Changes |
|---|---|
| **3.0.1** | Fixed: crash on the 3.5" C with Boo open (out of memory in the Wi-Fi event task). Boo's drawing buffer is half the size, and Wi-Fi scans in the background (Boo, radar, MESH tab) keep a bigger memory reserve and back off when memory is short. The boot image now shows the same way up as the scanner and Boo. |
| 3.0.0 | **Boo, the radio pet:** a Tamagotchi-style ghost whose mood comes from the signals around it, with FOOD, JOY and levels, saved to flash. Opens after the boot screen (Settings → *Start with Boo*); tap the header to go back to Boo. |
| 2.11.0 | Scanner alerts: banner, light-blue LED blinks and radar rings when another Geekaboo scanner arrives or leaves (Settings → *Scanner alerts*). |
| 2.10.0 | Geekaboo beacon: scanners can see each other (light blue in lists and on the radar, *Scanner* line on the detail page). Settings → *Visible*. |
| 2.9.1 | Fixed: Bluetooth crash from running out of memory (3.5" C). Wi-Fi is now only on during Wi-Fi scans, Bluetooth waits when memory is very low, smaller device table, memory figures in the Settings header. |
| 2.9.0 | ★ watchlist with here/gone alerts. 60-second signal graph on detail pages. Full-screen radar of all devices. Animated radar on the boot screen. *Restart* and *Factory reset* in Settings (tap twice). Crash report on the boot screen. Wi-Fi status messages when a list is empty. Taps register on release; hold = watch. Fixed: radar crash (uninitialised font in TFT_eSPI sprites), Wi-Fi scans skipped by a too-strict memory check. |
| 2.6.0 | Support for the 3.5" ESP32-3248S035R and ESP32-3248S035C. Touch calibration (first start on 3.5", Settings, or 3-second hold on the boot screen). *Invert colours* setting. Board picker on the installer page. |
| 2.5.0 | Version shown on the boot screen. Removed the MeshCore companion connection to save memory. TAGS tab back. |
| 2.4.0 | TAGS tab removed (test build). |
| 2.3.1 | Fixed a restart loop at boot (memory). Boot image flipped 180°. Crash banner on the boot screen. |
| 2.3.0 | Settings page. MeshCore companion connection. |
| 2.2.0 | MESH tab. |
| 2.1.0 | Boot image that waits for a tap; version in the header. |
| 2.0.0 | Portrait mode, rotate button, WIFI, CHAN and TAGS tabs, detail page with signal meter. Fixed oversized text. |
| 1.0.0 | First version: Bluetooth scanner. |

---

<p align="center">Made by <b>Geekaboo</b></p>
