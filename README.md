# 🎂 ESP32 Musical Birthday Gift Box

![Platform](https://img.shields.io/badge/platform-ESP32-blue)
![Display](https://img.shields.io/badge/display-SSD1306%200.96%22%20OLED-lightgrey)
![Framework](https://img.shields.io/badge/framework-Arduino-00979D)
![License](https://img.shields.io/badge/license-MIT-green)

A DIY electronic birthday surprise built with an **ESP32**, a **0.96" OLED**, a **passive buzzer** and **6 LEDs**.
It counts down to **12:00 PM**, flashes **"It's Time!"**, then plays the **Happy Birthday** melody note by note while the lyrics, a cake animation and every LED react to the music, and finally shows **"Make a Wish!"** before looping again.

> Made by **Soumik** for the [TechTadka360](https://github.com/Mr-soumik) maker channel.

<!-- Add your demo GIF or video link here, for example:
[![Watch the demo](thumbnail.jpg)](https://your-video-link)
-->

---

## ✨ Features

- ⏱️ **Live countdown timer** `11:59:56 → 11:59:59` with a beep, a screen flash and a rolling seconds digit on every tick
- 🎉 **"12:00 PM – It's Time!"** screen with an alert trill, wavy text, party-popper confetti and an LED mirror row on the OLED
- 🎵 **Accurate Happy Birthday melody** in C major (C4–C5), tuned note by note
- 📝 **Lyrics synced to every note** (Happy · Birthday · to · You · Dear **NAME**), sliding in smoothly
- 🎂 **Cake animation** with flickering candles, floating balloons, falling confetti and rising music notes
- 💡 **6-LED light show** that follows the countdown, the alert and every note of the song
- 🌟 **"Make a Wish!" finale**: candles blow out, sparkles, then the whole show loops forever
- ⚙️ **Easy to customise**: name, tempo, pitch and countdown start are single lines at the top of the code
- ✅ Works with **ESP32 Arduino core 2.x and 3.x** (LEDC API handled automatically)

---

## 🧰 Hardware

| Part | Qty | Notes |
|---|---|---|
| ESP32 Dev Board (30/38 pin) | 1 | Any standard ESP32 DevKit |
| 0.96" OLED, SSD1306, I2C (4-pin) | 1 | 128×64, address `0x3C` |
| Passive buzzer (2-pin) | 1 | **Must be passive**, an active buzzer cannot play a melody |
| LEDs (any colours) | 6 | Red/green alternating looks great |
| 220 Ω resistors | 6 | One per LED |
| Breadboard + jumper wires | 1 set | |

> **Active or passive?** Connect the buzzer straight to 3.3V. A passive buzzer only clicks, an active one beeps continuously.

---

## 🔌 Wiring (all 3.3V safe)

| Component | Pin | ESP32 |
|---|---|---|
| OLED | VCC | 3V3 |
| OLED | GND | GND |
| OLED | SDA | GPIO 21 |
| OLED | SCL | GPIO 22 |
| Buzzer | + | GPIO 25 |
| Buzzer | − | GND |
| LED 1 | long leg → 220 Ω | GPIO 23 |
| LED 2 | long leg → 220 Ω | GPIO 19 |
| LED 3 | long leg → 220 Ω | GPIO 18 |
| LED 4 | long leg → 220 Ω | GPIO 5 |
| LED 5 | long leg → 220 Ω | GPIO 4 |
| LED 6 | long leg → 220 Ω | GPIO 15 |

Every LED's **short leg goes to GND**. Use one common GND rail on the breadboard.

```
                 ┌──────────────┐
   OLED  SDA ────┤ GPIO21       │
         SCL ────┤ GPIO22       │
         VCC ────┤ 3V3          │
         GND ────┤ GND          │
                 │      ESP32   │
 BUZZER  (+) ────┤ GPIO25       │
         (−) ────┤ GND          │
                 │              │
 LED1 ─[220Ω]────┤ GPIO23       │
 LED2 ─[220Ω]────┤ GPIO19       │
 LED3 ─[220Ω]────┤ GPIO18       │
 LED4 ─[220Ω]────┤ GPIO5        │
 LED5 ─[220Ω]────┤ GPIO4        │
 LED6 ─[220Ω]────┤ GPIO15       │
                 └──────────────┘
      (all LED short legs → GND)
```

> GPIO 5, 4 and 15 are boot-strapping pins. With a 220 Ω LED on them the ESP32 still boots normally, the LEDs may just blink briefly at power-up.

---

## 📦 Software setup

1. Install the **Arduino IDE** and add ESP32 support (Boards Manager → *esp32 by Espressif*, version 2.x or 3.x).
2. Install these libraries from **Library Manager**:
   - `Adafruit GFX Library`
   - `Adafruit SSD1306`
   (`Wire` and the fonts used by the code come with the ESP32 core and Adafruit GFX.)
3. Open `Birthday_Gift_Box_Reel/Birthday_Gift_Box_Reel.ino` (the folder name must match the file name).
4. Select **Tools → Board → ESP32 Dev Module** and your COM port.
5. Click **Upload**. The show starts automatically after the upload and repeats forever.

---

## 🎛️ Customise it

Everything you may want to change is at the top of the sketch:

```cpp
#define NAME_TEXT "Friend"            // name in "Happy birthday dear ____" (max ~10 letters)
const uint8_t  OCTAVE_SHIFT = 0;      // 0 = original pitch, 1 = one octave higher (louder on tiny buzzers)
const uint32_t UNIT_MS      = 281;    // length of one eighth-note, lower = faster song
const uint8_t  START_SEC    = 56;     // countdown starts at 11:59:56
```

Other tunable values: `TIMER_MS`, `TIME_MS`, `OUTRO_MS` (phase lengths) and `LED_PINS[]` (LED pins).

---

## ⚙️ How it works

The ESP32's two cores split the job. **Core 1** runs the audio and LED state machine, **core 0** runs the OLED animation task at about 30 FPS, so the sound timing is never delayed by drawing.

| Phase | Duration | Screen | Sound | LEDs |
|---|---|---|---|---|
| **1. Timer** | 4 s | `11:59:56 → 11:59:59`, rolling digit, progress bar | 1 kHz beep each second | Outside-in flash, all LEDs on the last tick |
| **2. It's Time** | 3.2 s | `12:00 PM`, wavy "It's Time!", confetti poppers | 1.5 kHz alert trill | Fast back-and-forth sweep |
| **3. Birthday** | ≈ 14 s | Cake, candles, balloons, synced lyrics | Happy Birthday melody | One pair of LEDs per note |
| **4. Outro** | 5.6 s | "Make a Wish!", candles blow out, sparkles | Ta-da fanfare | Alternating blink |

Melody (C major, 1 unit = one eighth-note):

```
Happy birthday to you      C C D C F E
Happy birthday to you      C C D C G F
Happy birthday dear NAME   C C C' A F E D
Happy birthday to you      Bb Bb A F G F
```

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| Every note sounds the same | The buzzer is **active**. Use a **passive** buzzer. |
| Melody is too quiet | Set `OCTAVE_SHIFT = 1`. |
| OLED stays blank | Check SDA/SCL (21/22), power on 3V3, and the I2C address `0x3C`. If the first LED blinks forever, the OLED was not found. |
| Compile error about the board | Install the ESP32 board package and select *ESP32 Dev Module*. |
| Animation not smooth enough | Change `Wire.setClock(400000)` to `800000` if your OLED supports it. |

---

## 📁 Project structure

```
ESP32-Musical-Birthday-Gift-Box/
├── README.md
└── Birthday_Gift_Box_Reel/
    └── Birthday_Gift_Box_Reel.ino
```

---
## 📄 License

Released under the **MIT License**. Free to use, modify and share. A credit to **TechTadka360** is appreciated. ⭐ Star the repo if you liked it!

## 🔗 Follow TechTadka360

More DIY electronics projects on **YouTube · Instagram · Facebook** (search **TechTadka360**).
