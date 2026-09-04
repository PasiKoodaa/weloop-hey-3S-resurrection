
---

# WeLoop Watch Time Sync & Protocol Swiss Knife

> [!WARNING]
> **EXPERIMENTAL & USE AT YOUR OWN RISK**: Features interacting directly with watch hardware registers (GPS tracking, heart rate sensor triggers, device reboots, and historical data sync) rely on reverse-engineered protocols. While verified against native SDK implementations, sending raw BLE commands carries inherent risks of unexpected watch behavior or battery drain. Proceed with caution!

> Sync your watch time and manage your device long after official server shutdowns. A single HTML file opened in Chrome lets you sync your WeLoop Hey 3S watch time, monitor health sensors, trigger workouts, and manage watch configuration—no app installation needed, with zero dependency on cloud servers.

Official WeLoop app servers have shut down, rendering official apps non-functional. By reverse-engineering the official Android APK (`com.yf.lib.bluetooth`), this project identifies the exact **Protocol V2** wire formats used natively by the Hey 3S and exposes them through a pure, single-file HTML interface using the browser-native **Web Bluetooth API**.


---

## ⚡ The Breakthrough: Protocol V1 vs. Protocol V2

Previous third-party tools failed on the Hey 3S because they attempted to send legacy **Protocol V1** frames (`H2DR` headers, `0xA5` dialects, additive checksums) to a **Protocol V2** device.

The reverse-engineered SDK reveals two distinct communication stacks:

* **Protocol V1 (`com.yf.lib.bluetooth.b.b.*`)**: Legacy stack for older generation devices (Tommy, Now bands). Uses 8-byte `H2DR` frames with ASCII headers and explicit checksum trailers.
* **Protocol V2 (`com.yf.lib.bluetooth.b.c.*`)**: Native stack for **WeLoop Hey 3S**, Neo, and Coros hardware. Uses lightweight, direct GATT framing: `[CMD][SEQ=0x00][PAYLOAD...]` sent over write characteristic `6e400002`. It omits `H2DR` headers and software checksums, relying on BLE's native ATT/L2CAP link layer for packet integrity.

---

## ✨ Features

* **Zero Installation**: Single HTML file—opens directly in Chrome, Edge, or Opera.
* **Zero Cloud Dependency**: Operates 100% locally in your browser.
* **Cross-Platform**: Compatible with Windows, macOS, Linux, and Android.
* **Native Protocol V2 Sync**:
* Time synchronization via Opcode `0x8C` (`140`) using custom 2014/2015 epoch offsets.
* Supports 12H/24H formats and fractional UTC timezone offsets.


* **🏃 Sport & Activity Tracking**:
* Live query for daily steps, distance, active calories, and exercise minutes.
* Download historical activity logs (`0x7C` Index `0x05`).


* **🛰️ GPS & Running Mode**:
* Toggle native watch GPS workout tracking on/off (`0x98`).
* Query EPO satellite ephemeris status (`0x9B`).
* Download recorded GPS track point coordinates (`0x7C` Index `0x0E`).


* **💓 Sensor Control**:
* Toggle continuous optical HR monitoring (`0x97`).
* Extract dynamic heart rate log history (`0x7C` Index `0x06`).
* Standard GATT Battery Service querying (`0x180F`).


* **⚙️ Device Configuration**:
* Toggle Raise-to-Wake gesture (`0x80`).
* Set Left/Right wrist Wear Mode (`0x81`).
* Backlight level adjustment (`0x7E`).
* Toggle Anti-Lost (`0x88`) and Sedentary inactivity alerts (`0x82`).
* Remote Camera Shutter control (`0x8D`).
* Set Daily Activity Goals (`0x9A`).
* Push Weather Forecasts (`0x74`).
* Remote watch reboot (`0x89`).



---

## 🚀 How to Use

### System Requirements

| Platform | Browser | Supported |
| --- | --- | --- |
| Windows / macOS / Linux | Chrome, Edge, Opera | ✅ |
| Android | Chrome (Android 6.0+ with Location turned on) | ✅ |
| iOS / iPadOS | Any Browser | ❌ (Apple blocks Web Bluetooth) |
| Any Platform | Safari, Firefox | ❌ |


### Method 1: Local Execution

1. **Download** `index.html` locally.
2. **Unpair from OS Bluetooth Settings**: If your Hey 3S is already paired in your operating system settings, **unpair / forget the device** first (Web Bluetooth cannot claim system-paired devices).
3. **Open in Browser**:
* Double-click to open (`file://`) or host via a local HTTP server:
```bash
cd /path/to/project
python -m http.server 8000

```


Then open `http://localhost:8000/` in Chrome.



---

### Execution Steps

1. Go to OS Bluetooth settings and **unpair / forget** the Hey 3S.
2. Open the application, configure your timezone offset (e.g., `8` for UTC+8, `-5` for EST), and click **"Connect Device"**.
3. Select **WeLoop Hey 3S** from the browser's Bluetooth pair dialog.
4. Issue commands directly from the dashboard:
* Click **"Sync Watch Clock"** to send the Protocol V2 time packet.
* Configure watch settings (Raise-to-Wake, Backlight, Wear Mode).
* Toggle HR sensor or GPS workout mode.


5. Review outgoing packets and hardware replies in the **Activity Log**.

---

## 📖 Authoritative Protocol V2 Command Table (Hey 3S)

The following table details the native **Protocol V2** wire frames for the WeLoop Hey 3S (`6e400002` write characteristic):

| Feature | Opcode | Wire Packet Format | Notes / Migration from V1 |
| --- | --- | --- | --- |
| **Sync Time** | `0x8C` (`140`) | `[0x8C, 0x00, (Unix - 1388534400) 4B LE, TZ/30m, TZ/15m, 24hFlag]` | Epoch offset = 1388534400 (Jan 1, 2014/2015 UTC) |
| **Continuous Heart Rate** | `0x97` (`151`) | `[0x97, 0x00, enable ? 0x01 : 0x00]` | Replaces V1 `0xA5 0x62` |
| **Workout / GPS Track** | `0x98` (`152`) | `[0x98, 0x00, enable ? 0x01 : 0x00]` | Replaces V1 `0xA5 0x54` |
| **Raise-to-Wake** | `0x80` (`128`) | `[0x80, 0x00, enable ? 0x01 : 0x00]` | Replaces V1 `H2DR 0xB7` |
| **Wear Mode** | `0x81` (`129`) | `[0x81, 0x00, isRightHand ? 0x01 : 0x00]` | Left/Right wrist detection |
| **Anti-Lost Alert** | `0x88` (`136`) | `[0x88, 0x00, enable ? 0x01 : 0x00]` | Replaces V1 `H2DR 0xB2` |
| **Sedentary Alert** | `0x82` (`130`) | `[0x82, 0x00, enable ? 0x01 : 0x00]` | Inactivity reminder toggle |
| **Camera Remote Shutter** | `0x8D` (`141`) | `[0x8D, 0x00, enter ? 0x01 : 0x00]` | Replaces V1 `H2DR 0x9C` |
| **Reboot / Reset** | `0x89` (`137`) | `[0x89, 0x00, 0x00]` | Same opcode as V1, without `H2DR` framing |
| **Backlight Brightness** | `0x7E` (`126`) | `[0x7E, 0x00, mode]` | Modes: `0`=Off, `1`=Auto, `2`=High, `3`=Low |
| **Push Weather** | `0x74` (`116`) | `[0x74, 0x00, type, curTemp, maxMinTemp, forecast 2B LE, UTF-8 City]` | Weather payload frame |
| **Set Activity Goals** | `0x9A` (`154`) | `[0x9A, 0x00, steps 4B LE, distCm 4B LE, cal 4B LE, mins 2B LE]` | Target daily activity metrics |
| **Sync Historical Data** | `0x7C` (`124`) | `[0x7C, 0x00, dataIndex, address 4B LE]` | Indexes: `0x05` (Sport), `0x06` (HR), `0x0E` (GPS) |
| **Query GPS EPO Info** | `0x9B` (`155`) | `[0x9B, 0x00]` | Requests satellite ephemeris status |

### Service UUIDs

| Service / Characteristic | UUID |
| --- | --- |
| **WeLoop Service** | `6e400001-b5a3-f393-e0a9-77656c6f6f70` |
| **Write Characteristic (RX)** | `6e400002-b5a3-f393-e0a9-77656c6f6f70` |
| **Notify Characteristic (TX)** | `6e400003-b5a3-f393-e0a9-77656c6f6f70` |
| **GATT Battery Service** | `0000180f-0000-1000-8000-00805f9b34fb` |

---

## 🔧 Troubleshooting

| Issue | Cause & Solution |
| --- | --- |
| Device not listed in Web Bluetooth prompt | Ensure the Hey 3S screen is awake and **unpair it from OS Bluetooth settings** first. |
| Time command executes, but time doesn't update | Certain Hey 3S firmwares cache watch face displays; toggle watch screen or reconnect BLE to refresh UI. |
| Connection drops immediately | Ensure no background apps or mobile Bluetooth services are attempting to pair simultaneously. |

---

## ⚠️ Disclaimer

* This project is strictly for **personal hardware maintenance** and interoperability purposes.
* Protocol specifications were obtained via reverse engineering defunct official binaries (`com.yf.lib.bluetooth`).
* **Use at your own risk**: The maintainers are not responsible for unintended device states, firmware locks, or hardware issues resulting from direct BLE commands.

---

## 📄 License

MIT License
