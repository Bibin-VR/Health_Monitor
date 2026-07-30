# IoT Health Monitoring Dashboard

**Real-time vitals monitoring on an ESP8266** — heart rate, SpO₂, body temperature, and
stress level, logged to Firebase and visualised on a web dashboard.

An ESP8266 collects a 20-second reading window from onboard sensors, averages it,
displays the result on an LCD, and pushes it to Firebase Realtime Database. A Chart.js
web dashboard reads that data and renders it live, alongside an auto-generated health
report comparing the reading against a healthy baseline.

## Measures

| Metric | Sensor |
|---|---|
| Heart rate (BPM) | MAX30100 pulse oximeter |
| Oxygen saturation (SpO₂) | MAX30100 pulse oximeter |
| Body temperature | DS18B20 (1-Wire) |
| GSR / stress level | Analog GSR sensor |
| Glucose factor | Derived from heart rate and SpO₂ |

## Hardware

- ESP8266 NodeMCU (ESP-12E or Wemos D1 Mini)
- MAX30100 pulse oximeter sensor
- DS18B20 temperature sensor
- GSR sensor (analog, `A0`)
- 16×2 I2C LCD display (`0x27`)
- WiFi connection

## Firmware setup

**1. Install the Arduino IDE and board support**

Add the ESP8266 board package (Board Manager → "ESP8266 by ESP8266 Community").

**2. Install libraries** (Arduino Library Manager)

`ESP8266WiFi` · `Firebase_ESP_Client` · `Wire` · `LiquidCrystal_I2C` ·
`MAX30100_PulseOximeter` · `OneWire` · `DallasTemperature`

**3. Create a Firebase project**

Enable **Realtime Database**, set database rules to allow read/write (or configure
Auth), and copy the database URL and secret key.

**4. Configure credentials**

Set your WiFi and Firebase credentials at the top of `main.ino`:

```cpp
#define WIFI_SSID       "your-ssid"
#define WIFI_PASSWORD   "your-password"
#define DATABASE_URL    "your-project-default-rtdb.firebaseio.com"
#define DATABASE_SECRET "your-database-secret"
```

> Keep real credentials out of version control — move them to a local, gitignored
> header, or use `#include <secrets.h>` and commit a `secrets.h.example` instead.

**5. Flash and run**

Upload `main.ino` to the ESP8266. Data is pushed to `patients/patient1` in Firebase
Realtime Database on each valid 20-second reading window.

## Dashboard

`index.html` reads live values from Firebase and renders them with Chart.js. Serve it
from any static host, or open it directly with Firebase configured for your project.

## How a reading works

1. Sensors sample continuously for a 20-second window.
2. BPM and SpO₂ are averaged over the window; body temperature is read from the DS18B20
   and mapped into a realistic range.
3. The GSR reading is checked against a minimum threshold — below it, a reading is
   treated as invalid (no contact) and only averages are shown, nothing is uploaded.
4. Valid readings are shown on the LCD and pushed to Firebase; a glucose factor is
   derived from the BPM/SpO₂ ratio.
5. The cycle resets and starts collecting again.
