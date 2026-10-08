# Hardware

Original text by Rene Barbeau, VE2UG (2025). Revised by Rus Healy, K2UA (2026.10.08).

## Parts

| Part | Notes |
|---|---|
| WEMOS / LOLIN D1 mini (ESP8266), or clone | Chosen for low cost (about five for $15 at the time of the original write-up). |
| GPS module, 3.3 V TTL serial | NEO-6M, NEO-7M, or NEO-8M. Optional if you only use NTP mode. |
| Jumper (JP1) | Selects GPS or NTP mode. |
| Tone output connector (J1) | Carries the squarewave output and ground. |
| 5 V supply (J2) | USB or an external 5 V source. |
| LED and resistor, or a current-limited LED driver | For the optical output. See [Output stage](#output-stage). |

### Microcontroller

The ESP8266 D1 mini has 4 MB of flash, 11 I/O pins that support interrupts, PWM, I2C and 1-Wire, and one analog pin. It has **Wi-Fi** (the original document also mentions Bluetooth, but the ESP8266 does not have Bluetooth). It runs from 5 V and uses 3.3 V internally. The Arduino board package provides embedded functions for Wi-Fi and time.

### GPS module

If you are using the beacon in the field you will need a GPS. Any GPS with a 3.3 V TTL serial port will do. The original design uses a NEO-6M for its low cost. For better performance and a faster time to lock, a NEO-7M or NEO-8M can be used.

There are many variants, some with integrated antennas and some with separate ones. The original author suggests buying one with a detachable antenna, which gives more freedom to mount the module in a box. The RF connector on those modules provides DC voltage to power an external active antenna.

## Wiring

| From | To (D1 mini) | Notes |
|---|---|---|
| +5 V supply (J2 pin 1) | **5V** | |
| Ground (J2 pin 2) | **GND** | |
| GPS **VCC** | **3V3** | |
| GPS **GND** | **GND** | |
| GPS **TX** | **D7** (GPIO13, MOSI) | The sketch's `RXPin`. |
| GPS **RX** | **D6** (GPIO12, MISO) | The sketch's `TXPin`. |
| JP1, one side | **D5** (GPIO14, SCK) | Mode select. |
| JP1, other side | **GND** | Jumper **open** = GPS mode, **closed** = NTP mode. |
| Tone output (J1 pin 1) | **D8** (GPIO15, CS) | 3.3 V squarewave, 50% duty cycle. |
| Tone output ground (J1 pin 2) | **GND** | |

The sketch also drives the on-board LED (`LED_BUILTIN`, active low) to show tone activity.

> The original schematic was an image in the source PDF and is not reproduced here. The wiring table above is derived from it and matches the pin definitions in the sketch (`D7` = GPS RX, `D6` = GPS TX, `D5` = mode, `D8` = output).

## Output stage

The tone output is a **3.3 V squarewave**. You can connect a resistor and LED to test your receiver, or go QRO with a current-limited driver and a large LED array for real DX. In the original field tests, a single 940 nm LED reached 4 miles at dusk.

### Strapping-pin warning for D8

**D8 (GPIO15) is a boot strapping pin on the ESP8266 and must be low at boot.** The idle state of the tone output is low, so the stock design is fine. But if your output circuit (a driver stage, a pull-up, an LED network referenced to the supply) ever pulls D8 high during power-up or reset, the board will fail to boot. If you design a driver stage, make sure D8 is not pulled high at power-up, for example by adding a pull-down resistor to ground.

## Power

The board runs from 5 V on the **5V** pin or from its USB port. The GPS module is powered from the board's **3V3** pin in the original schematic. If you add a high-current LED driver, power it separately from the D1 mini's regulator.
