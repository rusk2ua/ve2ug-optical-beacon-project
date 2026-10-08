# Theory of operation

Original text by Rene Barbeau, VE2UG (2025). Revised and extended by Rus Healy, K2UA (2026.10.08).

The program is simple. It waits for a timed trigger point, then transmits a Q65-30 message, then a CW message, then a carrier of a set length. It then returns to waiting for the next trigger point.

Q65 transmissions must be accurately timed, so the beacon needs precise time from either GPS or NTP (Network Time Protocol).

## The transmit sequence

```mermaid
flowchart LR
    A[Trigger second reached] --> B[Q65 message 85 symbols]
    B --> C[Pause 1 s]
    C --> D[CW message]
    D --> E[Pause 0.5 s]
    E --> F[Carrier]
    F --> G[Wait for next trigger]
```

In GPS mode the software serial port to the GPS is disabled before the Q65 message and re-enabled after the carrier. This prevents serial interrupts from adding jitter to the `tone()` timing.

### Timing budget (default settings)

| Step | Calculation | Time |
|---|---|---|
| Q65 message | 85 symbols x 0.300 s | 25.500 s |
| Pause | fixed `delay(1000)` | 1.000 s |
| CW message `K2UA FN12FR ` | 18 WPM, dot = 66 ms (integer `1200/SPEED`) | about 9.372 s |
| Pause before carrier | fixed `delay(500)` in `Carrier()` | 0.500 s |
| Carrier | `CarrierLength` = 10 s | 10.000 s |
| **Total** | | **about 46.4 s** |

That leaves about 13.6 s of margin inside a 60 s minute. If you lengthen the CW message, slow the CW speed, or lengthen the carrier, recompute the total so it stays inside the window you want. The CW time depends on the message: every dit and dash is followed by a one-dot gap, every character by a two-dot gap, and every space is seven dots.

The Q65 part is fixed by the mode. **The second half of the minute is timed by hand.** The original author's advice still applies: start with a short CW message, high speed, and a short carrier, then adjust to fit your window.

## A word about Q65

A standard Q65 message is made of 85 symbols, each one of 65 possible tones. The message length (the T/R period) can be anywhere from 15 s up to 300 s. For this beacon the period is 30 s, which works out to **0.300 s per symbol** (85 x 0.3 = 25.5 s of tones, which leaves the remainder of the 30 s period unused).

You can also choose a **submode**, from A to E, which sets the tone spacing. Wider spacing occupies more bandwidth but helps decoding when scattering is expected.

This table from the WSJT-X `Q65_Quick_Start.pdf` shows the settings:

| T/R period (s) | A spacing (Hz) | A width (Hz) | B spacing | B width | C spacing | C width | D spacing | D width | E spacing | E width |
|---|---|---|---|---|---|---|---|---|---|---|
| 15 | 6.67 | 433 | 13.33 | 867 | 26.67 | 1733 | N/A | N/A | N/A | N/A |
| **30** | **3.33** | **217** | **6.67** | **433** | **13.33** | **867** | **26.67** | **1733** | N/A | N/A |
| 60 | 1.67 | 108 | 3.33 | 217 | 6.67 | 433 | 13.33 | 867 | 26.67 | 1733 |
| 120 | 0.75 | 49 | 1.50 | 98 | 3.00 | 195 | 6.00 | 390 | 12.00 | 780 |
| 300 | 0.29 | 19 | 0.58 | 38 | 1.16 | 75 | 2.31 | 150 | 4.63 | 301 |

For **Q65-30**, only submodes **A to D** are available.

For Q65-30 submode A, the tone spacing is 3.33 Hz. The lowest tone is 3.33 x 0 = 0 Hz and the highest tone is 3.33 x 64 = 213.12 Hz. Higher submodes are used when scattering of the signal is expected.

Generating Q65 data (parity, forward error correction involving Galois fields, and bit manipulation) is beyond the scope of this project. Fortunately WSJT-X provides a small program, `q65code.exe`, that generates the symbol list. See [Generating your Q65 message](q65-message-generation.md).

### Tone range with the default frequencies

The sketch computes each tone as `CarrierFreq + BaseFreq + symbol x ToneSpacing`. With the defaults (`CarrierFreq` = 14000, `BaseFreq` = 1000), the highest tone (symbol 64) is:

| Submode | Spacing (Hz) | Highest tone (Hz) |
|---|---|---|
| A | 3.33 | 15,213 |
| B | 6.67 | 15,426 |
| C | 13.33 | 15,853 |
| D | 26.67 | 16,706 |

All are comfortably below the 20 kHz `tone()` limit. See [Baseband or subcarrier](#baseband-or-subcarrier-audio) below.

## The program

The sketch is organized into these sections:

- **Configuration list.** Where you enter the operating data: GPS baud rate, Wi-Fi SSID and password, NTP server and time zone, the Q65 message array, base frequency, carrier frequency, tone spacing, start second, CW message, CW speed, and carrier length. See [Software setup](software-setup.md).
- **Main program.** Chooses GPS or NTP mode at boot and runs the trigger loop.

### Mode selection

Mode is read once, in `setup()`, from **D5** (GPIO14), which has its internal pull-up enabled:

- **D5 open** reads high, so the sketch selects **GPS mode**.
- **D5 jumpered to GND** reads low, so the sketch selects **NTP mode**.

Change the jumper only with the board powered down, or press reset afterward. The sketch prints `GPS mode` or `NTP mode` at the end of `setup()` so you can confirm which one it chose.

### GPS mode

This mode uses two libraries. **TinyGPSPlus** is an object-oriented library for parsing NMEA data from the GPS. **SoftwareSerial** (included with the Arduino IDE) builds a serial port on any pair of pins, so the GPS can talk on D7/D6 while the hardware serial port stays free for USB monitoring.

At startup the sketch opens the software serial port to the GPS at 9600 baud and starts gathering NMEA data. Once the GPS reports that the time is valid, the sketch watches `gps.time.second()`. At the trigger point (`Start`, second 0 or 30) it runs the transmit sequence:

1. Disable the software serial port (reduces `tone()` jitter).
2. Send the Q65 message.
3. Pause 1 second.
4. Send the Morse message.
5. Send the carrier.
6. Restart the software serial port to reconnect to the GPS.

It then returns to watching the seconds for the next trigger point.

While waiting, GPS mode prints a once-a-second heartbeat (`GPS chars processed: N  time valid: yes/no`) and, once time is valid, a per-second counter (`58.`, `59.`, `00.`). See [Testing and troubleshooting](testing-and-troubleshooting.md).

### NTP mode

This mode uses **ESP8266WiFi** for the Wi-Fi connection and the standard **time** functions for the clock. Note that the time is read differently than in GPS mode: the sketch uses `tm.tm_sec` instead of `gps.time.second()`.

At startup the sketch calls `configTime()` with the time zone and NTP server, then connects to the Wi-Fi network using the configured SSID and password. It waits up to 20 s for the connection and reports success or failure.

In `loop()` the sketch reads the current time and checks `tm.tm_sec`:

- **Until the clock has been set by SNTP** (epoch time later than `NTP_VALID_AFTER`), the sketch does nothing. Without this guard, `time()` returns a value near 1970 after boot, and the seconds field would be meaningless.
- **When the clock is valid**, it prints `NTP clock synced` once. At the trigger point (`tm.tm_sec == Start`) it runs the transmit sequence: Q65, 1 s pause, CW, carrier.

NTP mode is convenient for bench work and testing. For field use, GPS is the better time source: it needs no network, and in GPS mode the software serial port is explicitly disabled during transmission to reduce tone jitter.

## Baseband or subcarrier audio?

The choice depends on the application.

**Baseband.** If the optical receiver goes straight to a speaker or a sound card, baseband audio is fine. Set `CarrierFreq` to 0 and set `BaseFreq` where you want it, for example 800 Hz. This is good for decoding CW by ear. Baseband audio is noisier than a subcarrier because of AC hum from lights and electronics.

**Subcarrier.** With a subcarrier, a VLF receiver or SDR can be used for reception. You get all the features of a modern receiver, and the noise level is lower. Some optical receivers have bandwidth limits and may attenuate the signal, so check yours. SAQrx is a simple software SDR for a sound card. It works with standard 44.1 kHz sound cards or newer 96 or 192 kHz cards, with maximum receive frequencies of 22, 48, and 96 kHz respectively.

To use subcarrier mode, set `CarrierFreq` up to 16 kHz and `BaseFreq` typically to 1500 Hz (this repository's defaults are 14000 and 1000).

> **Limit:** the **aggregate tone frequency must stay under 20 kHz.** This is a hard limit in the `tone()` function. For example, with a base frequency of 1500 Hz and Q65-30D, the maximum carrier is 16.767 kHz. A carrier of 16 kHz or less is a safe bet.
