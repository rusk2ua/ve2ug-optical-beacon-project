# Testing and troubleshooting

Original field-test notes by Rene Barbeau, VE2UG (2025). Serial-output reference, boot-banner explanation, and troubleshooting by Rus Healy, K2UA (2026.10.08).

## Serial monitor basics

The sketch talks at **9600 baud**. Set the Serial Monitor to 9600.

### The ESP8266 boot banner

When the board resets, the ESP8266 boot ROM and SDK print a block of text **at 74880 baud**. If your monitor is set to 9600 you will see garbled characters at reset, then your sketch's output cleanly. That is normal. A typical banner looks like this:

```text
 ets Jan  8 2013,rst cause:2, boot mode:(3,7)

load 0x4010f000, len 3424, room 16
tail 0
chksum 0x2e
load 0x3fff20b8, len 40, room 8
tail 0
chksum 0x2b
csum 0x2b
v00047be0
~ld
rf cal sector: 1020
freq trace enable 0
rf[112] : 0
```

How to read it:

| Item | Meaning |
|---|---|
| `rst cause:2` | External reset (reset button, reset pin, or the serial reset that happens when you upload or open the monitor) or wake from deep sleep. This is a normal value to see. For the ROM boot log, Espressif lists `1` = power reboot, `2` = external reset or deep-sleep wake, and `4` = hardware watchdog reset. Software watchdog resets and software resets do not change the reported number. |
| `boot mode:(3,7)` | Booted from flash (normal run mode). |
| `load ... chksum ...` lines | The bootloader loading and verifying SDK segments. |
| `rf cal sector`, `freq trace enable`, `rf[112]` | Wi-Fi radio calibration messages. |

The garbage at the end of the banner (`rf[112] : 0` followed by odd characters) is where the boot ROM's 74880 baud output ends and your sketch's 9600 baud output begins.

To see the banner cleanly, set the monitor to 74880. Then your own output will look garbled. You cannot read both at once at one monitor setting. If you would rather see everything in one window, you can change `Serial.begin(9600)` to `Serial.begin(74880)` in the sketch.

If you see the banner **repeating** over and over, something is resetting the board. Look at the `rst cause` value, and check for exception dumps (`Exception (28)` with a stack trace, for example) or a power problem. A repeating `rst cause:4` points to a hardware watchdog reset.

## Bench test in NTP mode (recommended first test)

NTP mode needs no GPS, only Wi-Fi.

1. Jumper **D5 to GND**.
2. Set the Serial Monitor to 9600 and press reset.
3. Expected output:

   ```text
   NTP TZ DST - bare minimum
   .....
   WiFi connected
   NTP mode
   NTP clock synced
   ```

   The number of dots depends on how long Wi-Fi takes to connect.

4. At the top of the next minute (second 0), the transmit sequence starts:

   ```text
   Start of transmission
   K2UA FN12FR 
   Start of Carrier
   End of transmission
   ```

   The CW text appears as each character is keyed. The on-board LED is lit during Q65 tones, CW, and the carrier. The whole sequence takes about 46 s with the default settings.

5. If you have a scope, speaker, or receiver on **D8**, you should hear or see tones around 15 kHz.

### NTP-mode problems

| Symptom | Likely cause and fix |
|---|---|
| Dots forever, then `WiFi FAILED after timeout ...` | Wrong SSID or password, or weak signal. ESP8266 supports 2.4 GHz networks only. The sketch keeps retrying in the background. |
| `WiFi connected` but never `NTP clock synced` | No route to the NTP server, or the pool name is unreachable. Try a different `MY_NTP_SERVER`. Check that your network does not block UDP port 123. |
| Transmits at the wrong second of the minute | Wrong time source. Confirm `NTP clock synced` appeared before the first transmission. Time zone does not affect the seconds field. |
| Nothing prints after the banner | Monitor baud rate does not match `Serial.begin`. Use 9600. |

## Bench test in GPS mode

1. Wire the GPS ([Hardware](hardware.md)) and leave D5 **open**.
2. Set the monitor to 9600 and reset.
3. The sketch prints `GPS mode`, then once a second the heartbeat line:

   ```text
   GPS chars processed: 0  time valid: no
   ```

How to read the heartbeat:

| Heartbeat shows | Meaning |
|---|---|
| `chars processed: 0` and it never changes | No data is reaching the board. Check power, ground, the TX-to-D7 / RX-to-D6 wiring, and that the module outputs 9600 baud NMEA. |
| Count climbing, `time valid: no` | The GPS is sending data but has no time fix yet. Give it a clear view of the sky. A cold start can take minutes. |
| Count climbing, `time valid: yes` | Time is good. The per-second counter starts printing (`58.`, `59.`, `00.`). |

With no GPS attached the sketch idles silently apart from the heartbeat. That is correct behavior, because the loop only acts once `gps.time.isValid()` is true.

In GPS mode the monitor shows both the heartbeat and the per-second counter, so it is busier than in the original sketch.

At second 0 you will see `Start of transmission`, the CW text, `Start of Carrier`, and then the second counter resumes.

### GPS-mode problems

| Symptom | Likely cause and fix |
|---|---|
| Heartbeat shows 0 characters | Wiring swapped (GPS TX must go to D7), GPS not powered, or wrong baud rate. |
| Characters climb, time never valid | No fix. Move the antenna outdoors or near a window. |
| Count climbs, but transmissions never start | `Start` is not 0 or 30, or the sketch is in NTP mode (D5 grounded). Check the `GPS mode` / `NTP mode` line at boot. |

## Field timing

One way to check operation in the field is to watch the USB serial port at 9600 bps (GPS mode). You can see the system counting seconds up to 00, where the Q65 transmission starts. After that comes the CW message, then 10 s of carrier, and then the count resumes toward the next start of the minute. The on-board LED shows tone activity.

In the original author's field test with the message `VE2UG FN35GS`, the sequence finished with **13 seconds** to spare. That means you can lower the CW speed, add characters, or lengthen the carrier. Timing of the second half of the minute is adjusted manually, so start with a short CW message, high speed, and a short carrier, then adjust to fit your window.

```text
58.
59.
00.
Start of transmission
VE2UG FN35GS
47.
48.
...
59.
00.
Start of transmission
```

The per-second display is GPS mode only. In NTP mode you get Wi-Fi status messages instead.

## W1VLF field tests

Paul, W1VLF, made a video about his system and the field tests of the beacon at dusk: a single 940 nm LED was detected at 4 miles.
<https://www.youtube.com/watch?v=zmvFsv9W6x8>

## Going forward

From the original author: the next version should have menu-driven configuration and an internal Q65 symbol generator. A PCB is in the works.

## References

- SAQrx sound-card SDR receiver: <https://gitlab.com/jeayne/saqrx>
- Original author's software and project page: <https://sites.google.com/view/ve2ug-microwave-projects>
- Espressif, *ESP8266 Reset Causes and Common Fatal Exception Causes*: <https://documentation.espressif.com/esp8266_reset_causes_and_common_fatal_exception_causes_en.pdf>
