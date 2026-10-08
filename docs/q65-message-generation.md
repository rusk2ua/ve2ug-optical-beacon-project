# Generating your Q65 message

Original text by Rene Barbeau, VE2UG (2025). Revised by Rus Healy, K2UA (2026.10.08).

The `Q65[85]` array is the trickiest part of the setup. The beacon does not contain a Q65 encoder (calculating parity, forward error correction, and bit manipulation is a big job), so you generate the 85 channel symbols with a small program that ships with WSJT-X and paste them into the sketch.

## Step 1: run q65code

`q65code.exe` is in the WSJT-X install folder, typically `C:\WSJT\wsjtx\bin`. The message limit is **13 characters**.

```text
C:\WSJT\wsjtx\bin>q65code.exe "VE2UG FN35GS"
```

The program prints several blocks. The last block, **Channel symbols (85 total)**, is the one you need:

```text
Channel symbols (85 total)
 0  1 41 18 14 60 45 45  0 50 63  0  0 51  0 39 63  1  7 63
52  0  0  9 15  0  0 15 61 37 21  3  0  3  0 52 36  0 64 33
21 21 42 26 64  0  4 48  3  0 37 37 39  2  0  2 42 24  7  0
32  0  7 61 17  0 49 21  0 21 59  7 26  0 26  0 44 19 60 59
 5  9 54 37  0
```

## Step 2: add commas and paste

Copy the 85 channel symbols into the `Q65[85]` array in the sketch and **add a comma between each element**:

```cpp
int Q65[85] = {
  0,  1, 41, 18, 14, 60, 45, 45,  0, 50, 63,  0,  0, 51,  0, 39, 63,  1,  7, 63,
 52,  0,  0,  9, 15,  0,  0, 15, 61, 37, 21,  3,  0,  3,  0, 52, 36,  0, 64, 33,
 21, 21, 42, 26, 64,  0,  4, 48,  3,  0, 37, 37, 39,  2,  0,  2, 42, 24,  7,  0,
 32,  0,  7, 61, 17,  0, 49, 21,  0, 21, 59,  7, 26,  0, 26,  0, 44, 19, 60, 59,
  5,  9, 54, 37,  0 };
```

The example above is for `VE2UG FN35GS`, as in the original documentation.

## Step 3: match the CW message

Set `CWMessage` to the same text, with a trailing space, so the Q65 and CW parts of the transmission agree:

```cpp
char CWMessage[] = "VE2UG FN35GS ";
```

## Step 4: set the submode

Set `ToneSpacing` for the submode you want (Q65-30: A = 3.33, B = 6.67, C = 13.33, D = 26.67). In this sketch the submode only changes the spacing used to turn each symbol into a frequency.

## Checking your work

- The array must contain **exactly 85 values**, each from **0 to 64**.
- Every symbol is turned into a tone with `CarrierFreq + BaseFreq + symbol x ToneSpacing`. Make sure the highest tone stays below 20 kHz.
- Count the values if the sketch compiles but the message will not decode. A missing or extra comma is the usual cause.

## What ships in this repository

The sketch in `firmware/` is configured for **`K2UA FN12FR`**, with a matching `Q65[85]` array and CW message. Generate your own for your call sign and grid square and replace both.
