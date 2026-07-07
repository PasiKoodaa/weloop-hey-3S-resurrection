# WeLoop Hey 3S 时间同步工具

> 停服也能对时间。一个 HTML 文件，用 Chrome 打开即可把 WeLoop Hey 3S 手表的时间同步为当前时间，无需安装任何 app，不依赖云端服务器。

WeLoop（唯乐）官方 app 早已停服，云端登录走不通，手表时间无法同步。但时间同步这件事**本就不需要云端**——它是手机和手表之间直接的 BLE 蓝牙握手。本项目通过反编译官方 app，提取了 Hey 3S 的时间同步协议，用一个纯 HTML 页面（基于浏览器原生 Web Bluetooth API）完成同步。

## ✨ 特性

- **零安装**：一个 HTML 文件，Chrome / Edge 打开即用
- **零依赖**：不依赖 WeLoop 云端、不依赖 Python、不依赖任何 app
- **跨平台**：Windows / macOS / Linux / 安卓 Chrome 都能用
- **协议透明**：协议全部逆向自官方 app，源码开放

## 🚀 使用方法

### 环境要求

| 平台 | 浏览器 | 支持 |
|------|--------|------|
| Windows / macOS / Linux | Chrome、Edge、Opera | ✅ |
| Android | Chrome（需 Android 6.0+，开启位置权限） | ✅ |
| iOS / iPadOS | 任何浏览器 | ❌（Apple 不开放 Web Bluetooth） |
| 任何平台 | Safari、Firefox | ❌ |

### 步骤

1. **下载** [index.html](./index.html) 到本地
2. **取消系统配对**：到系统蓝牙设置里，如果 Hey 3S 已配对，先**取消配对 / 忽略该设备**（Web Bluetooth 不能使用已配对设备，这是浏览器规范限制）
3. **用 Chrome 打开** `index.html`
   - 直接双击（`file://`）在某些新版 Chrome 上可能被限制，推荐用本地 http 服务器托管：
     ```
     cd 到 index.html 所在目录
     python -m http.server 8000
     ```
     然后浏览器访问 `http://localhost:8000/`
4. 填时区（中国填 `8`），点 **「连接并同步时间」**
5. Chrome 弹出蓝牙设备选择框，选 **WeLoop Hey 3S**
6. 等待日志显示 `✅ 时间同步成功` 即可

### 故障排除

| 现象 | 解决 |
|------|------|
| 选择框里看不到 Hey 3S | 靠近手表 / 手表亮屏 / 系统蓝牙里取消已配对的 Hey 3S |
| 提示"此浏览器不支持 Web Bluetooth" | 换 Chrome 或 Edge；iOS 不支持 |
| 写入成功但时间显示 `00:00` | 时间戳字节序问题，本项目已修复（小端序） |
| 写入成功但时间没变 | 部分固件需断开手表蓝牙重连后才刷新表盘显示 |
| 连不上 | 先取消系统配对，再重试 |

## 🔧 工作原理

时间同步通过 BLE 发送一个 8 字节命令包：

```
[cmd=140][seq=0][4字节时间戳][时区/30min][时区/15min]
```

- **时间戳**：`(Unix秒 - 1388534400)`，其中 `1388534400` = 2014-01-01 00:00:00 UTC（WeLoop 自定义纪元）
- **字节序**：小端（little-endian）
- **时区**：以 30 分钟和 15 分钟为单位的两个字节（中国 UTC+8 = 480 分钟 → 16 和 32）

BLE 通道使用 Nordic UART Service 的 weeloop 变体：

| 用途 | UUID |
|------|------|
| Service | `6e400001-b5a3-f393-e0a9-77656c6f6f70` |
| 写特征（手机→手表） | `6e400002-b5a3-f393-e0a9-77656c6f6f70` |
| 通知特征（手表→手机） | `6e400003-b5a3-f393-e0a9-77656c6f6f70` |

UUID 后缀 `77656c6f6f70` 是 ASCII 字符串 **"weloop"**。

详细协议逆向过程见 [PROTOCOL.md](./PROTOCOL.md)。

## 📋 兼容性

已实测：
- ✅ **WeLoop Hey 3S**（`WeLoop Hey 3S` 广播名）

理论上同协议族的其他 WeLoop 设备（XH3、Now 3 / Neo 等）也可能适用，但未实测。欢迎反馈。

## ⚠️ 免责声明

- 本项目**仅供个人同步自己手表的时间使用**，不用于商业用途。
- 协议通过**逆向工程** WeLoop 官方 app（已停服）所得，仅用于实现互操作。
- 本项目**不包含** WeLoop 官方 app 的 APK 或反编译源码，仅包含独立编写的新代码。
- 使用本工具的风险自负，作者不对任何设备损坏负责。

## 🙏 致谢

- 协议来源：WeLoop 官方 app（反编译分析）
- 工具基于 [Web Bluetooth API](https://developer.mozilla.org/docs/Web/API/Web_Bluetooth_API)

## 📄 License

MIT
