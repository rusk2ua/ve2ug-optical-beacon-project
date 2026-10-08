# Changelog

All notable changes to this project are documented here. Versions are date-based (`YYYY.MM.DD`).

## [2026.10.08] - 2026-10-08

Revisions by **Rus Healy, K2UA**. Base: VE2UG original release (2025).

### Added

- **NTP sync guard.** In NTP mode, `loop()` returns early until `time()` is later than `NTP_VALID_AFTER` (1700000000, which is 2023-11-14 22:13:20 UTC). Until SNTP sets the clock, `time()` is near 1970, and the old code could fire a transmission at the wrong moment right after boot. Prints `NTP clock synced` once when the clock is set.
- **Wi-Fi connection timeout.** `WIFI_TIMEOUT_MS` (20 s). On timeout the sketch prints `WiFi FAILED after timeout - check SSID, password and signal. Still retrying in background.` and continues. The ESP8266 keeps trying to connect, and the sync guard prevents any transmission until the clock is valid.
- **Mode announcement.** Last line of `setup()` prints `GPS mode` or `NTP mode`.
- **GPS heartbeat.** Once a second in GPS mode, prints `GPS chars processed: N  time valid: yes/no`. `0` characters means no data from the GPS (wiring or power). A climbing count with `no` means data is flowing but there is no time fix yet.
- **Version header** in the sketch with original-author and revision attribution.
- **Comments** in the sketch: D8 (GPIO15) is a boot strapping pin that must be low at boot, and the ESP8266 boot banner runs at 74880 baud while the sketch runs at 9600.

### Changed

- **NTP server pool:** `at.pool.ntp.org` to `us.pool.ntp.org` (lower latency from the US east coast). Change it to the pool nearest you.
- **Shipped configuration** uses K2UA's message, `K2UA FN12FR`, for both the Q65 array and the CW message. Replace both with your own.

### Documentation

- Converted the original PDF to a Markdown documentation set under `docs/` plus a new `README.md`.
- Corrected the board description: the ESP8266 has Wi-Fi only (no Bluetooth).
- Added a timing budget for the transmit sequence (about 46.4 s with the default settings).
- Added an explanation of the ESP8266 boot banner and the 74880 vs 9600 baud behavior, with expected serial output for both modes.
- Added a troubleshooting guide for NTP and GPS modes.
- Added a Q65-30 tone-range table for submodes A to D with the default carrier and base frequency.
- Added explicit test-status tracking in the README.

### Unchanged

- Q65 symbol-to-frequency mapping, symbol timing, CW keyer, carrier routine, pin assignments, and the GPS/NTP jumper logic are as in the original.

## [VE2UG original] - 2025

Original design, sketch, and documentation by **Rene Barbeau, VE2UG** (documentation file dated 2025-08-24).

- ESP8266 D1 mini with GPS (NEO-xM) or NTP time synchronization, selected by jumper.
- Q65-30 message from a `q65code.exe`-generated symbol array, submodes A to D.
- Custom CW message and carrier after the CW message.
- 50% duty-cycle squarewave output via `tone()`, subcarrier up to 16 kHz.
- Morse keyer routines by Mark VandeWettering, K6HX. NTP example by noiasca.
