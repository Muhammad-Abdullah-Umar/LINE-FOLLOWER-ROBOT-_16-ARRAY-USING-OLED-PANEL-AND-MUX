# 16-IR PID Line Follower Robot

A PID-controlled line-following robot built around a 16-channel IR sensor array, an analog multiplexer, an OLED menu system, and a dual N20 motor drivetrain.

## Overview

This robot uses a dense 16-sensor IR array (read through a single analog MUX channel) to track a line with high positional resolution, feeding a PID control loop that adjusts differential motor speed in real time. A 1.3" OLED display with 6 push buttons provides an on-device menu for selecting modes, tuning PID constants, and starting/stopping runs without needing a laptop connected.

## Features

* 16x TCRT5000 IR reflectance sensors for fine-grained line position sensing
* CD74HC4067 16-channel analog multiplexer (reads all 16 sensors through a single analog input)
* PID-based line-following algorithm
* 1.3" OLED display with a 6-button menu (start/stop, mode select, PID tuning, calibration)
* Dual N20 motors (1000 RPM, with encoders) driven by a TB6612FNG dual H-bridge motor driver
* LiPo battery power (850 mAh) with buck converter for regulated 5V distribution
* Fully custom PCB (KiCad)

## Hardware Components

|Component|Qty|Notes|
|-|-|-|
|TCRT5000 IR sensor|16|220Ω LED current-limit + 10kΩ pull-up per sensor|
|CD74HC4067 16-ch analog MUX|1|Multiplexes all 16 sensors to one analog line|
|Microcontroller|1|ESP32 DEVKIT or Arduino Nano (see pin maps below)|
|1.3" OLED display (I2C, SH1106/SSD1306)|1|Menu and status display|
|Tactile push button|6|Menu navigation|
|TB6612FNG motor driver|1|Dual H-bridge|
|N20 motor w/ encoder, 1000 RPM|2|Drivetrain|
|LiPo battery, 850 mAh|1|Main power source|
|Buck converter module|1|Steps battery voltage down to 5V|
|Slide switch|1|Main power on/off|

## Pin Mapping — Arduino Nano

### IR Sensor MUX

|MUX pin|Nano pin|Notes|
|-|-|-|
|SIG|A6|`analogRead()` — analog-only pin, no digital I/O needed here|
|S0|D3|Digital output|
|S1|D9|Digital output|
|S2|A2|Digital output|
|S3|A3|Digital output|
|EN|GND|Always enabled|
|VCC|5V||
|GND|GND||

### OLED Display (I2C)

|OLED pin|Nano pin|
|-|-|
|VCC|5V|
|GND|GND|
|SCL|A5|
|SDA|A4|

### Menu Push Buttons (6x)

|Button|Nano pin|Mode|
|-|-|-|
|1|D10|`INPUT\\\_PULLUP`|
|2|D11|`INPUT\\\_PULLUP`|
|3|D12|`INPUT\\\_PULLUP`|
|4|D13|`INPUT\\\_PULLUP`|
|5|A0|`INPUT\\\_PULLUP`|
|6|A1|`INPUT\\\_PULLUP`|

Each button: one leg to the assigned pin, other leg to GND. Internal pull-ups used — no external pull-up resistors required on these six.

### Motor Driver (TB6612FNG)

|Driver pin|Nano pin|Notes|
|-|-|-|
|PWMA|D5|PWM-capable pin required|
|PWMB|D6|PWM-capable pin required|
|AIN1|D2|Direction control|
|AIN2|D4|Direction control|
|BIN1|D7|Direction control|
|BIN2|D8|Direction control|
|STBY|+5V (direct)|Hardwired HIGH to permanently enable the driver|
|VM|Battery rail (via buck/direct, per motor voltage)|Motor supply — **not** logic 5V|
|VCC|5V|Logic supply|
|GND|GND|Common ground|
|AO1 / AO2|Motor A terminals||
|BO1 / BO2|Motor B terminals||

### Reserved / Unused

* **D0 (RX), D1 (TX)** — reserved for Serial (USB upload + debugging), not used for any peripheral.
* **A7** — left free (analog-input-only pin; cannot drive digital outputs).

> \\\*\\\*Note:\\\*\\\* This pin map uses every available Nano digital/analog pin except D0/D1. There is no headroom left for additional peripherals on the Nano. If future features are planned (extra sensors, buzzer, Bluetooth, more buttons), consider migrating to an ESP32, which has significantly more usable GPIOs and native PWM on nearly any pin.

## Power Architecture

```
LiPo (850mAh) → Toggle Switch → Buck Converter → +5V rail
                                                 ├─ Microcontroller
                                                 ├─ OLED
                                                 ├─ IR sensor array + MUX
                                                 └─ Motor driver logic (VCC)
LiPo (raw voltage) ─────────────────────────────→ Motor driver VM (motor power)
```

Motor power (VM) is taken from the raw battery rail, separate from the regulated 5V logic rail, to avoid voltage sag on sensitive logic when motors draw current under load.

## Firmware Overview

1. **Calibration** — sweep the robot over the line at startup (or via menu) to record min/max sensor readings per channel for normalization.
2. **Sensor read loop** — cycle S0–S3 through all 16 states, `analogRead()` on SIG after each, normalize, and compute a weighted line-position error.
3. **PID loop** — compute correction from position error using tunable Kp/Ki/Kd, apply as a differential PWM offset between PWMA and PWMB.
4. **Menu system** — OLED + 6 buttons for navigating start/stop, mode selection, and live PID constant adjustment.

## Hardware Images

### Full Schematic

!\[Full schematic](images/schematic.png)

### PCB Layout (Top View)

!\[PCB layout](images/pcb\_layout.png)

### PCB Routing (Top + Bottom Copper)

!\[PCB routing](images/pcb\_routing.png)

### PCB 3D Render

!\[PCB 3D render](images/pcb\_3d\_render.png)

## Getting Started

1. Flash the firmware via USB (Arduino IDE or PlatformIO).
2. Power on via the slide switch.
3. Use the OLED menu to calibrate the sensor array on the actual track surface.
4. Tune PID constants via the menu, or hardcode starting values in firmware.
5. Select "Start" to begin autonomous line following.

## Repository Structure

```
/firmware       — Arduino/ESP32 source code
/hardware       — KiCad schematic and PCB files
/docs           — wiring diagrams, pin maps, BOM
README.md       — this file
```

## License

Add your preferred license here (e.g. MIT).

## Author

Muhammad Abdullah Umar



