# TD-GridController

A pair of modular TouchDesigner components (`.tox`) providing bi-directional I/O for 8x8 MIDI grid live-performance controllers, such as the Novation Launchpad lineup, DJTechTools's MIDI Fighter series, and 203Systems' Mystrix controllers.

It abstracts raw hardware note maps into standardized Cartesian coordinates (`tx`, `ty`, `active`) on input, and downscales any arbitrary TOP texture into low-latency SysEx FastLED/Apollo packets to drive the physical LEDs.

---

## Supported Hardware

| Controller | Input Status | LED Output (SysEx FastLED) | Notes |
| :--- | :--- | :--- | :--- |
| **Midi Fighter 64** | Fully Working | Fully Working | Tested; output requires compatible CFW (read below) |
| **Launchpad (any)** | Fully Working | Device dependent | Tested with LPX; output requires compatible CFW (read below) |
| **Mystrix** | Supported | Supported | Follows standard 8x8 note grid |

*Multiple controllers are supported simultaneously with independent routing. You can use one device to drive effects into any other controller's rendering area, to easily build lights that bleed onto nearby devices!*

## 🛠️ Components

### 1. `gridcontroller_in.tox`
Normalizes incoming MIDI streams across arbitrary channels (`ch1`–`ch16`) into clean 2D spatial coordinates.

* **Outputs:** 3 synchronous CHOP channels:
  * `tx`: Horizontal coordinate ($0..7$, left-to-right)
  * `ty`: Vertical coordinate ($0..7$, bottom-to-top)
  * `active`: Gate signal ($1$ when pressed, $0$ when released)
* **Agnostic Routing:** Strips manufacturer channel quirks to work with standard note layouts (`n36`–`n99`).

### 2. `gridcontroller_out.tox`
Accepts any TOP texture, downsamples it to an antialiased 8x8 canvas, and streams 60 FPS RGB updates to your hardware.

* **Native SysEx Pipeline:** Uses TouchDesigner's internal `midiOutCHOP.sendExclusive()`—no external Python libraries (`rtmidi`) or OS driver hooks required.
* **Bus Throttling & Change Detection:** Throttles output to a clean ~30 FPS frame cadence and skips duplicate packets to avoid flooding the MIDI buffer.
* **Dynamic Hardware Binding:** Binds directly to the target MIDI port defined in TouchDesigner's MIDI Device settings.

## ⚡ Quick Start

### Installation
1. Download the components from the `/tox` folder.
2. Drag and drop `gridcontroller_in.tox` and `gridcontroller_out.tox` into your network.

### Wiring Input
1. Open the parameters of `gridcontroller_in`.
2. Select your controller in the component settings.
3. Wire the output CHOP (`tx`, `ty`, `active`) into your logic, triggers, or interactive networks.

### Wiring Output
1. Connect any video stream, generative network, or texture into the input of `gridcontroller_out`.
2. Select your target device in the component settings.
3. The component downscales the canvas to 8x8 and maps pixel colors directly to the hardware LEDs.

## ☀️ Apollo FastLED support

The output driver implements the Apollo / FastLED SysEx protocol, for maximum performance on compatible firmware.

### Recommended firmware

- **Launchpad (any)**: Anth's [CoreFirmware](http://github.com/anthonyhfm/launchpad-core-firmware).
- **MIDI Fighter 64**: My own [RustyFighter64](https://github.com/zephyrcodesstuff/rf64)
- **Mystrix**: [Official firmware](https://github.com/203-Systems/MatrixOS) natively supports Apollo's FastLED. No need to use a custom firmware.


---

## License
AGPLv3.0. See [LICENSE](LICENSE) for more details.
