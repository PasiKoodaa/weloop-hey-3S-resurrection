# WeLoop Hey 3S BLE 时间同步协议

本文档记录 WeLoop Hey 3S 手表时间同步的 BLE 协议，通过对官方 app（`WeLoop_83624.apk`，包名 `com.yf.*`）反编译分析所得。

## 1. BLE 通道

Hey 3S 使用 Nordic UART Service (NUS) 的 weeloop 变体，所有 UUID 后缀 `77656c6f6f70` 是 ASCII **"weloop"**。

| 用途 | UUID |
|------|------|
| Service | `6e400001-b5a3-f393-e0a9-77656c6f6f70` |
| 写特征 RX（手机→手表） | `6e400002-b5a3-f393-e0a9-77656c6f6f70` |
| 通知特征 TX（手表→手机） | `6e400003-b5a3-f393-e0a9-77656c6f6f70` |

设备广播名前缀：`WeLoop Hey 3S`

> 另有数据通道 Service `6e400001-...-e50e24dcca9e`，用于固件传输、运动数据等，时间同步用不到。

## 2. 设备识别

app 中设备类型枚举（`com.yf.lib.bluetooth.b.g`）：

```java
WELOOPHEY3S("WELOOPHEY3S", new String[]{"WeLoop Hey 3S"}, 7)
```

广播名前缀匹配 `"WeLoop Hey 3S"` 即识别为该型号。

## 3. 时间同步命令

### 命令号

```
CMD_SYNC_TIME = 140  (0x8C)
```

来源：app 请求枚举 `com.yf.lib.bluetooth.c.b.syncTime`，经 `com.yf.lib.bluetooth.b.c.h` 的 `case syncTime` 路由，调用 `r.a(ag agVar)` 构造命令（`com.yf.lib.bluetooth.b.c.b.r.java`）。

### 时间戳算法

```java
// com.yf.lib.bluetooth.b.c.b.p.a()
public static long a() {
    return (Calendar.getInstance().getTimeInMillis() / 1000) - 1388534400;
}
```

`1388534400` = 2014-01-01 00:00:00 UTC（WeLoop 自定义纪元）。

### 时区字节

```java
// com.yf.lib.bluetooth.b.m
public static byte b() {  // 30 分钟为单位
    return (byte) ((timeZone.getOffset(now) / 60000) / 30);
}
public static byte c() {  // 15 分钟为单位
    return (byte) ((timeZone.getOffset(now) / 60000) / 15);
}
```

中国 UTC+8 = 480 分钟 → `b()` = 16，`c()` = 32。

### Payload 构造

```java
// r.a(ag)
ByteBuffer d2 = d(6);              // 6 字节缓冲
d2.putInt((int) p.a());            // 4 字节时间戳
d2.put(m.b());                     // 时区 /30min
d2.put(m.c());                     // 时区 /15min
return new r(140, d2);             // 命令号 140
```

Payload 共 **6 字节**：

| 偏移 | 长度 | 含义 |
|------|------|------|
| 0-3 | 4 字节 | 时间戳（Unix秒 − 1388534400） |
| 4 | 1 字节 | 时区偏移，单位 30 分钟 |
| 5 | 1 字节 | 时区偏移，单位 15 分钟 |

## 4. 字节序：小端

⚠️ **关键点**：反编译代码中 `ByteBuffer.putInt()` 默认大端序，但**实测 Hey 3S 固件按小端序解析时间戳**。

- 发大端：手表时间戳解析为极大值溢出，显示 `00:00`
- 发小端：时间正确

因此实现时必须用小端序：

```javascript
// Web Bluetooth / JavaScript
dv.setUint32(0, ts >>> 0, true);  // true = little-endian
```

```python
# Python
struct.pack("<I", ts & 0xFFFFFFFF)
```

## 5. 分包格式

命令通道每包 20 字节，格式：

```
[cmd][seq][...payload (最多 18 字节) ...]
```

时间同步 payload 仅 6 字节，单包即可：

```
完整包 (8 字节): [140][0][ts_byte0][ts_byte1][ts_byte2][ts_byte3][tz_30][tz_15]
```

示例（小端，unix=1783392778, tz=8）：

```
8c 00 8a 0f 89 17 10 20
```

## 6. 返回包

设备通过通知特征返回 3 字节：

```
[cmd=140][seq][result]
```

`result == 1` 表示成功。

部分固件对时间同步不发返回包，写入成功即可。

## 7. 代码定位（反编译源码）

| 作用 | 文件 |
|------|------|
| 时间同步入口 | `com/yf/smart/weloopx/device/setting/a/g.java`（Hey3sRepairTimezoneHelper）|
| 请求路由 | `com/yf/lib/bluetooth/b/c/h.java`（`case syncTime`）|
| 命令构造 | `com/yf/lib/bluetooth/b/c/b/r.java`（`r.a(ag)`）|
| 时间戳算法 | `com/yf/lib/bluetooth/b/c/b/p.java`（`p.a()`）|
| 时区字节 | `com/yf/lib/bluetooth/b/m.java`（`m.b()` / `m.c()`）|
| GATT 回调 + UUID | `com/yf/lib/bluetooth/b/c/b.java` |
| 设备类型枚举 | `com/yf/lib/bluetooth/b/g.java`（`WELOOPHEY3S`）|
| 请求类型枚举 | `com/yf/lib/bluetooth/c/b.java`（`syncTime`）|

## 8. 其他命令（未在本工具实现，仅记录）

`com.yf.lib.bluetooth.b.c.b.r` 中还包含其他命令号，供后续扩展参考：

| 命令号 | 用途 |
|--------|------|
| 112 | 初始化系统（含时间戳，配置设备时用）|
| 113 | 设置用户卡片信息 |
| 114 | 背光模式 |
| 116 | 设置城市 |
| 117 | 单位切换 |
| 128-149 | 各种设置（闹钟、心率、抬手亮屏、震动等）|
| 163 | 重启 |
| 164 | 重置 |
