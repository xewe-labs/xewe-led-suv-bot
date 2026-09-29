# sUv Bot — solar-charged cocktail stand with glass-triggered LED animations

Personal project (built for a bar) · 2022 (uploaded to GitHub 2023-11-18) · Solo: Max Dokukin · Status: Completed

Arduino based bot for animated glass illumination.

![IMG_0549](https://github.com/xeweva/sUv-Bot/assets/54597813/dddf0378-ee2e-4326-b7a7-6a228ef5232c)

## Overview

sUv Bot is a small, portable stand that lights a cocktail glass from below. A glass sensor in the cup well tells an
Arduino Nano when a glass is set down. The firmware then plays a welcome animation across six LED groups, settles into
a slow "breathing" fade while the glass stays, and fades out smoothly when the glass is lifted. The electronics run from
a single-cell battery charged by a solar panel on the lid (or micro-USB) through a TP4056 charger, all inside a
3D-printed, spray-painted case. The firmware is 416 lines of Arduino C++ in five files.

## Highlights

- Four-state firmware (no glass → welcome → continuous → waiting for the glass back) in `sUv_Bot.ino`; every animation loop re-checks the sensor so lifting the glass interrupts it at any brightness
- Six PWM LED channels (D3, D5, D6, D9, D10, D11), 12 LEDs, two per group — pin table in [`IMG_0245`](static/media/resources/IMG_0245.webp)
- AVR timer prescalers set to 1 for 62.5 kHz PWM on D5/D6 and 31.37 kHz on D3/D11 and D9/D10 (code comments); `Delay()` multiplies by 64 to keep animation timing in real milliseconds
- Solar panel → L7806CV → TP4056 charger → battery → on/off switch → Arduino, and transistor LED drivers with 1 kΩ / 10 kΩ gate resistors — hand-drawn schematic in [`IMG_0244`](static/media/resources/IMG_0244.webp)

## How it works

```
glass placed (sensor pin LOW)
  → state 1  welcome: groups 0+4, then 1+5, then 2+3 fade 0→255 (5 ms/step), then all fade down to 3
  → state 2  continuous: all groups breathe 3→255→3 (10 ms/step) while the glass stays
glass lifted (pin HIGH) at any step → fade out from current brightness (3 ms/step)
  → state 3  waiting: dim glow (PWM 2/255) for ~10 s; glass back → state 2, else all off → state 0
```

- **`sUv_Bot.ino`** — pin map (`SENS_PIN 2`, `LED_GR_0..5` on 3, 5, 6, 9, 10, 11), timer prescaler setup, serial at 9600 baud, and the `switch(state)` main loop with a `delay(5)` tick.
- **`GeneralFunctions.h`** — `PWM(ch, pow)` for one group, `PWMOPP(ch, pow)` for the paired groups (0+4, 1+5, 2+3), `PWMALL(pow)`, `Delay(ms)` (= `delay(ms * 64)` to compensate for the sped-up Timer0), and the `fadeOut` / `fadeOutAll` / `fadeTo` ramps.
- **`WelcomeAnimation.h`** — the three-pair build-up; each step checks the sensor and, if the glass is gone, fades the lit groups out and returns to state 0.
- **`ContiniousAnimation.h`** — `fadeInOut()` breathing loop (active) and `fadeCircular()`, a 360-step rotating fade built from piecewise-linear `map()` curves (present but disabled).
- **`WaitingGlass.h`** — holds a dim glow for 640,000 `millis()` ticks (≈10 s real time at the ×64 timer speed) waiting for the glass to come back.

Hardware (from the photographed notes in `static/media/resources/`):

| Part | Role |
|---|---|
| Arduino Nano | runs the sketch |
| Glass sensor | in the centre of the cup well; `INPUT_PULLUP`, LOW when a glass is present (code: pin 2; the schematic drawing shows D8) |
| 12 LEDs in 6 groups | two LEDs per PWM pin, each group switched low-side by a transistor (1 kΩ gate resistor, 10 kΩ pull-down) |
| Solar panel + L7806CV | passive charging from the lid |
| TP4056 + battery | single-cell charger (also micro-USB), output to the Arduino through an on/off switch |
| Case | 3D-printed, spray-painted |

## Getting started

```text
1. Open sUv_Bot.ino in the Arduino IDE (the .h files sit next to it and are included by the sketch).
2. Tools → Board → Arduino Nano (ATmega328P); pick the port.
3. Upload. Open the Serial Monitor at 9600 baud: "Welcome" is printed each time the welcome animation starts.
```

Requirements: Arduino IDE (no external libraries). Wire the LED groups and sensor as in the pin table above; the timer
setup in `setup()` targets the AVR timers of an ATmega328P-based board. Because Timer0 runs 64× faster, use `Delay()`
(not `delay()`) for any new animation timing.

## Documents

- [Schematic (hand-drawn)](static/media/resources/IMG_0244.webp)
- [LED group / pin map](static/media/resources/IMG_0245.webp)
- Build photos: [case and LED wiring](static/media/resources/IMG_0546.webp), [electronics bay](static/media/resources/IMG_0544.webp), [finished stand](static/media/resources/IMG_0550.webp)
- Videos: [glass lit in the bar](static/media/resources/IMG_0278_optimized.mp4), [printing the case](static/media/resources/IMG_0243_optimized.mp4)
- Related LED project: XeWe LED OS — [project page](https://maxdokukin.com/projects/xewe-led-os) · [repo](https://github.com/xewe-labs/xewe-led-os)
