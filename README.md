# HALO - Smart Color-Changing Bulb

HALO is an Arduino-based smart RGB bulb project (Lifx-inspired) that lets you control the color of an RGB LED from your phone. A companion Android app and a built-in web interface send RGB values to a microcontroller, which drives the LED channels using PWM. The project supports two control paths: an **ESP8266 (WiFi)** path and a **Bluetooth + Arduino** path.

> Demo and write-up are included in this repo:
> - `Arduino based color changing bulb - converted with Clipchamp.mp4` - video demo
> - `Smart RGB Bulb Report.pdf` - full project report
> - `Color LED Controller.apk` - prebuilt Android control app

## How It Works

The system maps a chosen color to red, green, and blue intensity values (0-255 / 0-1023 depending on board) and writes them to three PWM output pins wired to an RGB LED. There are two firmware variants in this repo:

### 1. WiFi variant - `wifimodulecodefinal.ino` (ESP8266)
- The ESP8266 connects to a WiFi network and starts an HTTP web server on port 80 (`ESP8266WebServer`), plus an mDNS responder (`esp8266.local`).
- Opening the server's IP in a browser serves a self-contained HTML/JavaScript color picker. Tapping/dragging on a gradient canvas computes RGB values client-side and POSTs them back as `?r=&g=&b=` query arguments.
- `handleRoot()` reads those arguments and calls `analogWrite()` on the R/G/B pins to update the LED.
- On boot, `testRGB()` fades each channel in and out as a self-test.
- (Optional OLED status display via Adafruit SSD1306 is present but commented out in the source.)

### 2. Bluetooth variant - `RGB_Test.ino` (Arduino + Bluetooth + LDR)
- Reads color commands over the serial/Bluetooth link (e.g. an HC-05/HC-06 module) sent from the Android app.
- An LDR (light-dependent resistor) on the sensor pin acts as an ambient-light gate: if measured light is above a threshold (`>= 400`), the bulb stays off ("LIGHTS OFF"); below the threshold it accepts color commands.
- The incoming string is parsed to extract `red.green.blue` values between brackets, converted to integers, and written to the LED via `analogWrite()` (using `255 - value`, i.e. common-anode / inverted drive).

## Hardware Components

| Component | Used by | Notes |
|-----------|---------|-------|
| ESP8266 WiFi module / board | `wifimodulecodefinal.ino` | Hosts the web server |
| Arduino board (e.g. Uno/Nano) | `RGB_Test.ino` | Drives LED, reads serial |
| RGB LED (or RGB LED strip with controller) | both | The "bulb" |
| Bluetooth module (HC-05 / HC-06) | `RGB_Test.ino` | Serial link to phone |
| LDR (photoresistor) | `RGB_Test.ino` | Ambient-light gating |
| Resistors / breadboard / jumper wires | both | Standard prototyping |
| Android phone | both | Runs the control app |

## Wiring

### ESP8266 (`wifimodulecodefinal.ino`)
| Signal | GPIO |
|--------|------|
| Red    | GPIO14 |
| Green  | GPIO12 |
| Blue   | GPIO13 |

PWM range on the ESP8266 is 0-1023.

### Arduino + Bluetooth (`RGB_Test.ino`)
| Signal | Pin |
|--------|-----|
| Red    | D9 (PWM) |
| Green  | D10 (PWM) |
| Blue   | D11 (PWM) |
| LDR    | sensor pin (analog read) |
| Bluetooth | hardware/serial RX/TX |

PWM range on classic Arduino is 0-255. Connect each LED channel through an appropriate current-limiting resistor.

## Setup & Flashing

1. Install the [Arduino IDE](https://www.arduino.cc/en/software).
2. For the ESP8266 variant, add the ESP8266 board package (Boards Manager URL: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`) and install the **ESP8266** boards. The required libraries (`ESP8266WiFi`, `WiFiClient`, `ESP8266WebServer`, `ESP8266mDNS`) ship with that package.
3. Open the desired `.ino` file in the Arduino IDE.
4. **Before flashing the WiFi variant**, set your own WiFi `ssid` and `password` at the top of `wifimodulecodefinal.ino` (see Security note below).
5. Select the correct board and port, then click **Upload**.
6. **ESP8266:** open the Serial Monitor at `115200` baud to read the assigned IP address, then browse to that IP (or `http://esp8266.local`) from a phone on the same network to use the color picker.
7. **Arduino/Bluetooth:** open Serial Monitor at `9600` baud, pair the Bluetooth module with your phone, and use the Android app (`Color LED Controller.apk`) to send colors.

## Android App

`Color LED Controller.apk` is the prebuilt control app. Install it on an Android device (enable "install from unknown sources"), pair with the Bluetooth module (or connect to the ESP8266), and pick colors to send to the bulb.

## Tech Stack

- **Firmware:** C/C++ (Arduino framework)
- **WiFi/web:** ESP8266 Arduino core (`ESP8266WebServer`, mDNS), inline HTML/JavaScript color-picker UI
- **Connectivity:** WiFi (HTTP) and Bluetooth (serial)
- **Sensing:** LDR-based ambient-light gating (Bluetooth variant)
- **Client:** Android app (`.apk`)

## Repository Structure

```
.
├── wifimodulecodefinal.ino   # ESP8266 WiFi firmware + web color picker
├── RGB_Test.ino              # Arduino Bluetooth firmware + LDR gating
├── Color LED Controller.apk  # Prebuilt Android control app
├── Smart RGB Bulb Report.pdf # Project report / documentation
├── Arduino based color changing bulb - converted with Clipchamp.mp4  # Demo video
└── README.md
```

## Security Note

`wifimodulecodefinal.ino` currently contains **hardcoded WiFi credentials** committed to source. Before reusing this code, replace them with your own network details and avoid committing real credentials to version control. For a cleaner approach, move credentials into a separate, git-ignored config header (e.g. `secrets.h`) that is excluded via `.gitignore`.
