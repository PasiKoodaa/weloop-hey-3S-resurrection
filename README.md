
---

# WeLoop Watch Time Sync & Swiss Knife Tool

> [!WARNING]
> **EXPERIMENTAL & USE AT YOUR OWN RISK**: The extended features added in this tool (GPS triggers, continuous heart rate sensor toggles, device reboots, and block data syncs) are experimental and directly interact with watch hardware parameters. While tested on reverse-engineered protocols, misuse or interrupted writes could cause unexpected watch behavior. Proceed at your own discretion!

> Sync your watch time and manage your device even after official server shutdowns. A single HTML file opened in Chrome lets you sync your WeLoop Hey 3S watch time, track sports metrics, trigger GPS workouts, and manage watch configuration—no app installation needed, with zero dependency on cloud servers.

The official WeLoop app has long been shut down, leaving users unable to sync their watch time or log into cloud services.
By reverse-engineering the official app, this project extracted the Hey 3S time sync protocol and extended BLE SDK commands into a pure HTML page (powered by the browser-native Web Bluetooth API).


---

## ✨ Features

* **Zero Installation**: A single HTML file that works directly in Chrome / Edge.
* **Zero Dependencies**: Independent of WeLoop cloud servers, Python, or any mobile app.
* **Cross-Platform**: Works on Windows, macOS, Linux, and Android Chrome.
* **Time Synchronization**:
* **Official 2-Step Protocol**: Handshake via opcode `0x90` with full 11-byte calendar payload.
* **Legacy Protocol**: Single-frame fallback using opcode `0x8C`.
* Customizable 12H/24H hour format and timezone offsets.


* **🏃 Sport Tracking & Live Counters**:
* Read daily totals: steps, distance (km), active calories, and exercise minutes (`0x9F`).
* Historical step log sync (`0x7C` / `0x05`).


* **🛰️ GPS & Running Mode**:
* Toggle watch GPS running workout mode (`0xA5 0x54`).
* Query satellite EPO info status (`0x9B`).
* Download recorded GPS track point coordinates (`0x7C` / `0x0E`).


* **💓 Health & Sensor Control**:
* Toggle continuous heart rate monitoring on/off (`0xA5 0x62`).
* Download dynamic heart rate log history (`0x7C` / `0x06`).
* Query battery level via standard Bluetooth Battery Service (`0x180F`).


* **⚙️ Device Configuration & Remote Triggers**:
* Toggle Raise-to-Wake gesture light (`0xB7`).
* Change backlight level: Off / Auto / High (`0xB0`).
* Toggle Anti-Lost vibration warnings (`0xB2`).
* Open/Exit watch camera shutter mode (`0x9C`).
* Remote watch reboot command (`0x89`).



---

## 🚀 How to Use

### System Requirements

| Platform | Browser | Supported |
| --- | --- | --- |
| Windows / macOS / Linux | Chrome, Edge, Opera | ✅ |
| Android | Chrome (Requires Android 6.0+ with Location turned on) | ✅ |
| iOS / iPadOS | Any Browser | ❌ (Apple does not support Web Bluetooth) |
| Any Platform | Safari, Firefox | ❌ |


### Method 1: Local Execution (Offline / Backup if Online Version is Unavailable)

1. **Download** [index.html](https://www.google.com/search?q=./index.html) locally.
2. **Unpair from System**: Go to your OS Bluetooth settings. If the Hey 3S is already paired, **unpair / forget the device** first (Web Bluetooth cannot connect to paired devices due to browser specification restrictions).
3. **Open with Chrome**:
* Double-clicking to open (`file://`) might be restricted in newer Chrome versions (Web Bluetooth requires secure origins like HTTPS or `localhost`). It is recommended to serve it via a local HTTP server:
```bash
cd /path/to/index.html
python -m http.server 8000

```


Then open `http://localhost:8000/` in your browser.



---

### Execution Steps

1. **Unpair from System**: Go to your OS Bluetooth settings. If the watch is paired, **unpair / forget the device** first.
2. Open the tool in Chrome / Edge, enter your target timezone (e.g., `8` for UTC+8, `-5` for EST), select your preferred clock format (12-Hour or 24-Hour), and click **"Connect Device"**.
3. When the Chrome Bluetooth selector pops up, choose your device (e.g., **WeLoop Hey 3S**, **Tommy**, or **Now**).
4. Perform desired operations:
* Click **"Sync Watch Clock"** to update time.
* Query daily metrics using **"Query Today's Sport Totals"**.
* Adjust screen backlight or raise-to-wake settings under **Device Configuration**.


5. Observe execution outputs and notifications in the real-time **Activity Log**.

---

### Troubleshooting

| Issue | Solution |
| --- | --- |
| Device not visible in selector | Move closer / wake screen / unpair watch in OS Bluetooth settings |
| Message: "Web Bluetooth is not supported" | Switch to Chrome or Edge; iOS is unsupported |
| Write succeeds but watch shows `00:00` | Select "Official 2-Step" protocol or check timezone offset |
| Write succeeds but watch display unchanged | Certain firmwares require disconnecting watch Bluetooth and reconnecting to refresh the watch face |
| Failed to connect | Unpair from system Bluetooth settings first, then retry |

---

## 🔧 Protocol & Command Reference

The tool uses standard **H2DR** (Host-to-Device Request) frames and the **0xA5** protocol dialect over Nordic UART Service (variant `weloop`).

### Frame Formats

1. **Standard H2DR Frame** (8 bytes):
```text
[0..3] : "H2DR" (ASCII: 0x48, 0x32, 0x44, 0x52)
[4]    : Command Opcode
[5]    : Subcommand
[6]    : Parameter / Sequence
[7]    : Checksum (Sum of bytes [0..6] & 0xFF)

```


2. **0xA5 Dialect**:
```text
[0]    : 0xA5 (Sync byte)
[1]    : Opcode
[2..]  : Command Payload

```



### Opcodes Table

| Opcode | Description | Structure / Example |
| --- | --- | --- |
| `0x8C` | Legacy Single-Frame Time Sync | `[0x8C, 0x00, 4B LE Timestamp, TZ/30m, TZ/15m]` |
| `0x90` | Time Sync Step 1 Handshake | H2DR Frame (`0x90`, SubCmd `0x0B`) + 11-byte Calendar Payload |
| `0x9F` | Daily Sport Totals Query | H2DR Frame (`0x9F`, `0x00`, `0x00`) → Returns steps, distance, calories, active time |
| `0x54` | Toggle GPS Workout Mode | `[0xA5, 0x54, 0x01]` (Start) / `[0xA5, 0x54, 0x00]` (Stop) |
| `0x62` | Continuous HR Sensor | `[0xA5, 0x62, 0x01, 0, 0, 0, 0, 0]` (On) / `[0xA5, 0x62, 0x00, ...]` (Off) |
| `0x7C` | Request Block Data Sync | `[0x7C, 0x00, DataIndex, 4B Address]` <br>

<br> • `0x05`: Sport Logs <br>

<br> • `0x06`: Heart Rate Logs <br>

<br> • `0x0E`: GPS Track Coordinates |
| `0x9B` | Query GPS Satellite EPO Info | H2DR Frame (`0x9B`, `0x00`, `0x00`) |
| `0xB7` | Raise-To-Wake Gesture | H2DR Frame (`0xB7`, `0x00`, `0x01`/`0x00`) |
| `0xB0` | Backlight Level Control | H2DR Frame (`0xB0`, `0x00`, Mode: `0`/`1`/`2`) |
| `0xB2` | Anti-Lost Vibration Alert | H2DR Frame (`0xB2`, `0x00`, `0x01`/`0x00`) |
| `0x9C` | Remote Camera Control | H2DR Frame (`0x9C`, `0x00`, `0x18` Open / `0x00` Exit) |
| `0x89` | Watch Reboot | H2DR Frame (`0x89`, `0x01`, `0x00`) |

### Service UUIDs

| Service / Characteristic | UUID |
| --- | --- |
| **WeLoop Service** | `6e400001-b5a3-f393-e0a9-77656c6f6f70` |
| **Write Characteristic (RX)** | `6e400002-b5a3-f393-e0a9-77656c6f6f70` |
| **Notify Characteristic (TX)** | `6e400003-b5a3-f393-e0a9-77656c6f6f70` |
| **Battery Service** | `0000180f-0000-1000-8000-00805f9b34fb` |

---

## 📋 Compatibility

Verified on:

* ✅ **WeLoop Hey 3S** (`WeLoop Hey 3S` broadcast name)

Theoretically compatible with other devices in the WeLoop BLE family (e.g., WeLoop Tommy, Now, Now 3 / Neo, XH3), though specific feature availability varies by firmware.

---

## ⚠️ Disclaimer

* This project is **intended solely for personal device maintenance** on your own hardware and not for commercial use.
* The protocol was obtained via **reverse engineering** the official (defunct) WeLoop app strictly to enable interoperability.
* This repository **does not contain** official WeLoop APK files or reverse-compiled source code; it contains only independently written code.
* **Use at your own risk**: The authors accept no liability for hardware malfunction, data loss, or bricked devices resulting from the use of experimental commands.

---

## 🙏 Acknowledgments

* Protocol Source: WeLoop official Android app (reverse-engineering analysis)
* Built using the [Web Bluetooth API](https://developer.mozilla.org/docs/Web/API/Web_Bluetooth_API)

---

## 📄 License

MIT
