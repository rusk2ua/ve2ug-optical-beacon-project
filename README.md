# Q65/Morse Generator for Optical Beacons

An ESP8266 (WEMOS/LOLIN D1 mini) beacon that transmits a **Q65-30** message, then a **CW (Morse) message**, then a **plain carrier**, on a schedule locked to **GPS** or **NTP** time. The output is a 3.3 V squarewave on one pin, intended to drive an LED for optical (light-beam) communication, with the signal carried on an audio or VLF-range subcarrier (typically around 14 kHz).

Originally designed by **Rene Barbeau, VE2UG**, for experiments with **Paul, W1VLF**, whose SDR-based optical receiver digs the subcarrier out of the noise. This repository is a maintained, revised copy with additional robustness and diagnostics added by **Rus Healy, K2UA**.

**Repository:** <https://github.com/rusk2ua/ve2ug-optical-beacon-project>

---

## Version information

| Item | Value |
|---|---|
| **Current release** | **2026.10.08** |
| Versioning scheme | Date-based (`YYYY.MM.DD`) |
| Based on | VE2UG original release, 2025 (documentation file dated 2025-08-24) |
| Revisions in this release by | Rus Healy, K2UA |
| Target board | WEMOS / LOLIN D1 mini (ESP8266), including clones |
| Arduino board package | `esp8266` by ESP8266 Community. The original author used **3.1.2**. |
| Required library | `TinyGPSPlus` by Mikal Hart. The original author used **1.1.0**. |
| Other libraries | `SoftwareSerial`, `ESP8266WiFi`, `time.h` (all come with the IDE or the board package) |
| Sketch | [`firmware/Q65_CW_Beacon/Q65_CW_Beacon.ino`](firmware/Q65_CW_Beacon/Q65_CW_Beacon.ino) |
| Serial monitor rate | **9600 baud** |

### What changed in 2026.10.08

All changes below are by **Rus Healy, K2UA**. Full details are in [CHANGELOG.md](CHANGELOG.md).

- **NTP sync guard.** The beacon no longer transmits until SNTP has actually set the clock. Previously, right after boot `time()` returns a value near 1970, and a transmission could fire at the wrong moment.
- **Wi-Fi connection timeout (20 s).** A bad SSID or password now produces a clear error message instead of printing dots forever. The ESP8266 keeps retrying in the background.
- **NTP server pool changed** from `at.pool.ntp.org` to `us.pool.ntp.org`. Change it to the pool nearest you.
- **Mode announcement.** `setup()` now prints `GPS mode` or `NTP mode`, so you know right away which branch the jumper selected.
- **GPS heartbeat.** Once a second in GPS mode, the sketch prints `GPS chars processed: N  time valid: yes/no`. This separates "no data from the GPS at all" (wiring or power) from "data is flowing but no fix yet".
- **Documentation rewrite.** The original PDF was converted to Markdown and extended with a timing budget, a serial-output reference, a troubleshooting guide, and an explanation of the ESP8266 boot banner. See [Documentation](#documentation).

### Test status of this release

| Area | Status |
|---|---|
| NTP mode: Wi-Fi connect, NTP sync alert, status messages | Verified by K2UA on a WEMOS D1 mini clone (2026-10-08) |
| GPS mode: heartbeat and transmit sequence | Not yet tested with GPS hardware attached |
| Tone output on D8 / optical or RF output | Not recorded for this release |

Please update this table as you test.

---

## How it works (short version)

At a trigger second (second `0` of the minute by default), the beacon sends:

1. A **Q65-30** message (85 symbols x 0.300 s = 25.5 s).
2. A 1 s pause.
3. A **CW** message at 18 WPM by default.
4. A 0.5 s pause, then a **carrier** for 10 s by default.

Then it waits for the next trigger. With the default settings the whole sequence takes about **46.4 s**, which leaves about 13.6 s of margin inside the 60 s minute. See [Theory of operation](docs/theory-of-operation.md) for the timing budget.

Timing comes from one of two sources, selected with a jumper:

| Jumper on D5 | Mode | Time source |
|---|---|---|
| **Open** | GPS | NEO-6M/7M/8M on SoftwareSerial (D7 RX, D6 TX) |
| **Closed to GND** | NTP | Wi-Fi + SNTP, with time zone and DST handled by `MY_TZ` |

GPS mode is the one to use in the field. NTP mode is convenient for the bench and for testing without a GPS.

---

## Quick start

1. Clone the repository and install the Arduino IDE, the **esp8266** board package, and the **TinyGPSPlus** library. See [Software setup](docs/software-setup.md).

   ```bash
   git clone https://github.com/rusk2ua/ve2ug-optical-beacon-project.git
   ```

2. Open `firmware/Q65_CW_Beacon/Q65_CW_Beacon.ino`.
3. Edit the **CONFIGURATION LIST** at the top of the sketch:
   - `STASSID` / `STAPSK` (NTP mode only).
   - `MY_TZ` and `MY_NTP_SERVER`.
   - `CarrierFreq`, `BaseFreq`, `ToneSpacing`, `Start`.
   - `CWMessage` and `SPEED`.
   - The `Q65[85]` array, generated for **your** message. See [Generating your Q65 message](docs/q65-message-generation.md).
4. Select the board **LOLIN(WEMOS) D1 mini (clone)** and upload.
5. Open the Serial Monitor at **9600 baud**.
6. **Bench test in NTP mode first:** jumper D5 to GND, reset, and watch for `NTP mode`, dots, `WiFi connected`, and `NTP clock synced`. The first transmission starts at the top of the next minute.

> **The shipped sketch is configured for K2UA's callsign and grid** (`K2UA FN12FR`, with a matching `Q65[85]` array). Replace the message, the array, and the Wi-Fi settings with your own before use. The Wi-Fi credentials in the sketch are placeholders (`"redacted"`). Never commit real credentials to a public repository.

---

## Documentation

| Document | Contents |
|---|---|
| [docs/theory-of-operation.md](docs/theory-of-operation.md) | Q65 basics, submodes, GPS and NTP behavior, transmit sequence, timing budget |
| [docs/hardware.md](docs/hardware.md) | Parts, wiring table, GPS module notes, output stage notes, strapping-pin warning |
| [docs/software-setup.md](docs/software-setup.md) | IDE, board package, libraries, every configuration field explained |
| [docs/q65-message-generation.md](docs/q65-message-generation.md) | Using `q65code.exe` and turning its output into the `Q65[85]` array |
| [docs/testing-and-troubleshooting.md](docs/testing-and-troubleshooting.md) | Expected serial output, boot banner and baud rates, GPS and NTP diagnostics, field timing |
| [CHANGELOG.md](CHANGELOG.md) | Release history |

---

## Repository layout

```text
.
├── README.md
├── CHANGELOG.md
├── docs/
│   ├── theory-of-operation.md
│   ├── hardware.md
│   ├── software-setup.md
│   ├── q65-message-generation.md
│   └── testing-and-troubleshooting.md
└── firmware/
    └── Q65_CW_Beacon/
        └── Q65_CW_Beacon.ino
```

---

## Known limitations

- **One trigger second.** `Start` selects either second 0 or second 30. The sketch does not transmit at both.
- **The second half of the minute is timed by hand.** The Q65 portion is fixed at 25.5 s, but the CW message length and carrier length are up to you. Keep the total under the time window you want. See the timing budget.
- **No on-board Q65 encoder.** Messages must be generated with `q65code.exe` from WSJT-X and pasted into the sketch. The original author lists a built-in encoder and menu-driven configuration as future work.
- **`tone()` limit.** The sum of carrier, base frequency, and the highest Q65 tone must stay under 20 kHz. Staying at or below a 16 kHz carrier is a safe bet.
- **GPS mode prints two kinds of lines.** The once-a-second heartbeat and the per-second counter both appear, so the monitor is busier than in the original. That is expected.

---

## Credits and attribution

- **Rene Barbeau, VE2UG.** Original design, sketch, and documentation (2025). Project page: <https://sites.google.com/view/ve2ug-microwave-projects>
- **Paul, W1VLF.** Optical receiver work, field testing, and the motivation for the project. Field-test video: <https://www.youtube.com/watch?v=zmvFsv9W6x8>
- **Mark VandeWettering, K6HX.** The "Simple Arduino Morse Beacon" keyer routines (<https://brainwagon.org>). His header, retained in the sketch, releases that code without restrictions and asks for a brief acknowledgement in derivatives.
- **noiasca (Werner Rothschopf).** The "NTP TZ DST - bare minimum" example the NTP section is based on (2020-09-22): <https://werner.rothschopf.net/202011_arduino_esp8266_ntp_en.htm>
- **Mikal Hart.** The TinyGPSPlus library.
- **Rus Healy, K2UA.** Revision 2026.10.08: NTP sync guard, Wi-Fi timeout, NTP pool change, mode announcement, GPS heartbeat, and the Markdown documentation set.
- **SAQrx**, a simple sound-card SDR receiver suited to subcarrier reception: <https://gitlab.com/jeayne/saqrx>

---

## License

**Not yet specified.** The original VE2UG release does not state a license, so no `LICENSE` file is included. The K6HX Morse routines carry their own "no restrictions" notice in the sketch header, and the NTP example is by noiasca. Until the original author's wishes are confirmed, treat this repository as shared for amateur radio experimentation with attribution, and check with VE2UG before redistributing it more widely.
