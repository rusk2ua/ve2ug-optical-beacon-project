# Software setup

Original text by Rene Barbeau, VE2UG (2025). Revised by Rus Healy, K2UA (2026.10.08).

## 1. Arduino IDE

Install the Arduino IDE if you have not already. The official tutorial is here:
<https://docs.arduino.cc/software/ide-v2/tutorials/getting-started/ide-v2-downloading-and-installing/>

You then need two more things: the ESP8266 board package, and the TinyGPSPlus library.

## 2. ESP8266 board package

1. In **File > Preferences > Additional boards manager URLs**, add this line:

   ```text
   http://arduino.esp8266.com/stable/package_esp8266com_index.json
   ```

2. Open **Tools > Board > Boards Manager**.
3. Search for **esp8266** and install **esp8266 by ESP8266 Community**. The original author used version **3.1.2**. This makes the ESP8266 family of boards available and provides extra libraries such as `ESP8266WiFi`.
4. Select the board **LOLIN(WEMOS) D1 mini (clone)**. This is the board used for development, and it works with genuine D1 minis too.

## 3. Libraries

Open **Tools > Manage Libraries...** and install **TinyGPSPlus** by Mikal Hart (the original author used version **1.1.0**). It provides an object-oriented way to handle NMEA sentences.

The other libraries (`SoftwareSerial`, `ESP8266WiFi`, `time.h`) are provided by the Arduino IDE or by the board package.

## 4. Configure the sketch

Open `firmware/Q65_CW_Beacon/Q65_CW_Beacon.ino` and edit the **CONFIGURATION LIST** near the top.

### Global settings

| Setting | Default | Meaning |
|---|---|---|
| `OutPin` | `D8` | Tone output pin. D8 is a boot strapping pin; see [Hardware](hardware.md). |
| `ModePin` | `D5` | Mode select. Open = GPS, grounded = NTP. |

### GPS

| Setting | Default | Meaning |
|---|---|---|
| `GPSBaud` | `9600` | Baud rate of the GPS module (9600 for the NEO-xM family). |
| `RXPin`, `TXPin` | `D7`, `D6` | Software serial pins. `RXPin` connects to the GPS **TX**. |

### NTP

| Setting | Default | Meaning |
|---|---|---|
| `STASSID` | `"redacted"` | Your Wi-Fi network name. |
| `STAPSK` | `"redacted"` | Your Wi-Fi password. |
| `MY_NTP_SERVER` | `"us.pool.ntp.org"` | NTP server or pool. Pick the pool nearest you. |
| `MY_TZ` | `"EST5EDT,M3.2.0,M11.1.0"` | POSIX time zone string with DST rules. This one is US Eastern. |
| `WIFI_TIMEOUT_MS` | `20000` | How long `setup()` waits for Wi-Fi before reporting an error. |
| `NTP_VALID_AFTER` | `1700000000` | Any epoch time earlier than this is treated as "clock not yet set". |

> Do not commit your real Wi-Fi password to a public repository. Keep the placeholders in version control and edit a local copy, or define `STASSID` and `STAPSK` before the sketch's `#ifndef STASSID` block (for example in a header you keep out of git).

### Q65

| Setting | Default | Meaning |
|---|---|---|
| `CarrierFreq` | `14000` | Subcarrier frequency in Hz. Set to 0 for baseband. Keep at 16 kHz or less. |
| `BaseFreq` | `1000` | Hz above the carrier for tone 0. |
| `SymbolLength` | `0.300` | Seconds per symbol. 0.300 is correct for Q65-30. |
| `ToneSpacing` | `3.33` | Hz between tones. A = 3.33, B = 6.67, C = 13.33, D = 26.67 for Q65-30. |
| `Start` | `0` | Trigger second: `0` (start of the minute) or `30`. |
| `Q65[85]` | K2UA FN12FR | The 85 channel symbols for your message. See [Generating your Q65 message](q65-message-generation.md). |

The **aggregate tone frequency** (`CarrierFreq + BaseFreq + 64 x ToneSpacing`) must stay under 20 kHz.

### Morse (CW)

| Setting | Default | Meaning |
|---|---|---|
| `SPEED` | `18` | CW speed in WPM. |
| `CWMessage` | `"K2UA FN12FR "` | CW text. **Keep the trailing space.** Supported characters are uppercase `A-Z`, `0-9`, `.`, `,`, `?`, and `/`. Anything else is sent as `?` on the serial monitor and skipped. |

### Carrier

| Setting | Default | Meaning |
|---|---|---|
| `CarrierLength` | `10` | Carrier length in seconds. |

Keep the total sequence inside the time window you want. See the timing budget in [Theory of operation](theory-of-operation.md#timing-budget-default-settings).

## 5. Upload and monitor

1. Connect the board over USB and select its port.
2. Upload the sketch.
3. Open the Serial Monitor and set it to **9600 baud**.

Next: [Testing and troubleshooting](testing-and-troubleshooting.md).

## Worked example (original author's)

These were the parameters in the original documentation:

| Parameter | Value |
|---|---|
| SSID | `MYSSID` |
| Password | `MYPASSWORD` |
| Carrier frequency | 14000 Hz |
| Tone (base) frequency | 1000 Hz |
| Symbol (tone) length | 0.300 s (Q65-30) |
| Tone spacing | 3.33 Hz (submode A) |
| Time to start | 0 s |
| Q65 message | `VE2UG FN35GS` |
| CW message | `VE2UG FN35GS` |
| CW speed | 18 WPM |
| Carrier length | 10 s |

The corresponding fields in the sketch:

```cpp
#ifndef STASSID
#define STASSID "MYSSID"
#define STAPSK  "MYPASSWORD"
#endif

int CarrierFreq = 14000;
int BaseFreq = 1000;
float ToneSpacing = 3.33;
float SymbolLength = 0.300;
int Start = 0;
```
