Here is the full translation of the README into English:

# WeLoop Watch Time Sync Tool

> Sync your watch time even after server shutdown. A single HTML file opened in Chrome lets you sync your WeLoop Hey 3S watch time to the current time—no app installation needed, with zero dependency on cloud servers.

The official WeLoop app has long been shut down, leaving users unable to sync their watch time or log into cloud services.
By reverse-engineering the official app, this project extracted the Hey 3S time sync protocol and uses a pure HTML page (powered by the browser-native Web Bluetooth API) to accomplish time synchronization.

> 🔗 **Use Online Directly (No Download Required)**: [https://micookie2.github.io/weloop-hey3s-time-sync/](https://micookie2.github.io/weloop-hey3s-time-sync/)

## ✨ Features

* **Zero Installation**: A single HTML file that works directly in Chrome / Edge.
* **Zero Dependencies**: Independent of WeLoop cloud servers, Python, or any mobile app.
* **Cross-Platform**: Works on Windows, macOS, Linux, and Android Chrome.
* **Transparent Protocol**: Fully reverse-engineered from the official app with open source code.

## 🚀 How to Use

### System Requirements

| Platform | Browser | Supported |
| --- | --- | --- |
| Windows / macOS / Linux | Chrome, Edge, Opera | ✅ |
| Android | Chrome (Requires Android 6.0+ with Location turned on) | ✅ |
| iOS / iPadOS | Any Browser | ❌ (Apple does not support Web Bluetooth) |
| Any Platform | Safari, Firefox | ❌ |

### Method 1: Online Version (Recommended & Easiest)

Simply open the online version hosted on GitHub Pages in your browser—**no downloading or installation required**:

> 🔗 **[https://micookie2.github.io/weloop-hey3s-time-sync/](https://micookie2.github.io/weloop-hey3s-time-sync/)**

Once opened, follow the "Synchronization Steps" below. We recommend bookmarking this page for future time syncing.

### Method 2: Local Execution (Offline / Backup if Online Version is Unavailable)

1. **Download** [index.html](https://www.google.com/search?q=./index.html) locally.
2. **Unpair from System**: Go to your OS Bluetooth settings. If the Hey 3S is already paired, **unpair / forget the device** first (Web Bluetooth cannot connect to paired devices due to browser specification restrictions).
3. **Open with Chrome**:
* Double-clicking to open (`file://`) might be restricted in newer Chrome versions (Web Bluetooth requires secure origins like HTTPS or `localhost`). It is recommended to serve it via a local HTTP server:
```bash
cd /path/to/index.html
python -m http.server 8000

```


Then open `http://localhost:8000/` in your browser.



### Synchronization Steps

1. **Unpair from System**: Go to your OS Bluetooth settings. If the Hey 3S is already paired, **unpair / forget the device** first (Web Bluetooth cannot connect to already paired system devices).
2. Open the page using either method above, enter your timezone (e.g., enter `8` for China UTC+8 or `-5` for EST), and click **"Connect & Sync Time"**.
3. When the Chrome Bluetooth device selector pops up, choose **WeLoop Hey 3S**.
4. Wait until the log displays `✅ Time sync successful`.

### Troubleshooting

| Issue | Solution |
| --- | --- |
| Hey 3S not visible in selector | Move closer to watch / turn on watch screen / unpair Hey 3S in OS Bluetooth settings |
| Message: "Web Bluetooth is not supported" | Switch to Chrome or Edge; iOS is unsupported |
| Write succeeds but watch shows `00:00` | Timestamp endianness issue; fixed in this project (uses Little-Endian) |
| Write succeeds but watch time unchanged | Certain firmwares require disconnecting watch Bluetooth and reconnecting to refresh the watch face |
| Failed to connect | Unpair from system Bluetooth settings first, then retry |

## 🔧 How It Works

Time synchronization sends an 8-byte command packet via BLE:

```
[cmd=140][seq=0][4-byte timestamp][TZ/30min][TZ/15min]

```

* **Timestamp**: `(Unix seconds - 1388534400)`, where `1388534400` = 2014-01-01 00:00:00 UTC (WeLoop custom epoch).
* **Byte Order**: Little-Endian.
* **Timezone**: Two bytes representing UTC offset measured in 30-minute and 15-minute intervals (e.g., China UTC+8 = 480 minutes → 16 and 32).

The BLE channel uses a WeLoop variant of the Nordic UART Service:

| Purpose | UUID |
| --- | --- |
| Service | `6e400001-b5a3-f393-e0a9-77656c6f6f70` |
| Write Char (Phone → Watch) | `6e400002-b5a3-f393-e0a9-77656c6f6f70` |
| Notify Char (Watch → Phone) | `6e400003-b5a3-f393-e0a9-77656c6f6f70` |

The UUID suffix `77656c6f6f70` is the ASCII string **"weloop"**.

For a detailed walkthrough of the reverse-engineered protocol, see [PROTOCOL.md](https://www.google.com/search?q=./PROTOCOL.md).

## 📋 Compatibility

Verified on:

* ✅ **WeLoop Hey 3S** (`WeLoop Hey 3S` broadcast name)

Theoretically, other WeLoop devices using the same protocol family (e.g., XH3, Now 3 / Neo) might also work, but remain unverified. Feedback is welcome.

## ⚠️ Disclaimer

* This project is **intended solely for personal time synchronization** on your own devices and not for commercial use.
* The protocol was obtained via **reverse engineering** the official (defunct) WeLoop app strictly to enable interoperability.
* This repository **does not contain** official WeLoop APK files or reverse-compiled source code; it contains only independently written code.
* Use this tool at your own risk. The author accepts no liability for hardware damage.

## 🙏 Acknowledgments

* Protocol Source: WeLoop official app (reverse-engineering analysis)
* Built using the [Web Bluetooth API](https://developer.mozilla.org/docs/Web/API/Web_Bluetooth_API)

## 📄 License

MIT
