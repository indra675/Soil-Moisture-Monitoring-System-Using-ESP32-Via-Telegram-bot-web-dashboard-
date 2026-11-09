<div align="center">

# 🌱 ESP32 Soil Moisture Monitor

### Live web dashboard + Telegram bot alerts, all running on a single ESP32

![Platform](https://img.shields.io/badge/Platform-ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white)
![Arduino](https://img.shields.io/badge/Framework-Arduino-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![Telegram](https://img.shields.io/badge/Alerts-Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)
![Cloud](https://img.shields.io/badge/Cloud-Not%20Needed-2E9E5B?style=for-the-badge)

[Features](#-features) • [Hardware](#-hardware) • [Setup](#-quick-start) • [How it works](#-how-it-works) • [Results](#-results) • [Roadmap](#-future-scope)

</div>

---

## 📖 About

Watering plants too early or too late wastes water and harms them. This project measures soil moisture continuously and tells you **only when action is needed**.

The ESP32 reads a soil probe, converts the value into a percentage, and shares it two ways at the same time:

| 🖥️ Local Dashboard | 🤖 Telegram Bot |
|---|---|
| Hosted on the ESP32 itself | Works from anywhere with internet |
| Live gauge, chart, statistics | Dry alert + recovery message |
| No cloud, no subscription | `/moisture` and `/status` commands |

Everything lives in **one Arduino sketch**: `Soil_Moisture.ino`.

---

## ✨ Features

- 📊 **Live dashboard** with gauge, 20-minute history chart, alert log and device stats
- 🌗 **Responsive** layout with automatic light/dark mode
- 🔔 **Smart alerts** using hysteresis (30 % / 40 %) and a 30-minute cooldown, so no spam
- 💬 **Telegram commands:** `/start`, `/moisture`, `/status`
- 🔒 **Secure:** only your authorised chat ID gets replies
- 🎯 **Stable readings:** each value is the average of 20 samples
- 📶 **Auto-reconnect** if Wi-Fi drops
- ⚡ **No external server:** the web page is stored in the ESP32's flash

---

## 🔧 Hardware

| # | Component | Purpose |
|:-:|---|---|
| 1 | ESP32 development board | Controller, Wi-Fi, web server |
| 2 | Soil moisture probe + LM393 module | Analog moisture output |
| 3 | Jumper wires and breadboard | Connections |
| 4 | USB cable and 5 V supply | Power and programming |

### 🔌 Wiring

| Sensor pin | ESP32 pin |
|:-:|:-:|
| `VCC` | `3V3` |
| `GND` | `GND` |
| `AOUT` | `GPIO34` |

> 💡 GPIO34 is on **ADC1**, which keeps working while Wi-Fi is on. ADC2 pins do not.

<div align="center">
<img src="images/block-diagram.png" alt="System block diagram" width="85%">
<br><sub><b>System block diagram</b></sub>
</div>

<br>

<table>
<tr>
<td align="center"><img src="images/prototype-test.jpg" alt="Prototype in test container"><br><sub>Probe in test container</sub></td>
<td align="center"><img src="images/prototype-air.jpg" alt="Prototype in open air"><br><sub>Probe in open air</sub></td>
</tr>
</table>

---

## 📚 Libraries

| Library | Author |
|---|---|
| `UniversalTelegramBot` | Brian Lough |
| `ArduinoJson` 6.21.x | Benoit Blanchon |
| `WiFi.h`, `WiFiClientSecure.h`, `WebServer.h` | Included with the ESP32 board package |

---

## 🚀 Quick Start

**1. Create your Telegram bot**
Message `@BotFather` on Telegram to get a **bot token**, and `@myidbot` to get your **chat ID**.

**2. Install the libraries** from the Arduino Library Manager.

**3. Wire the sensor** as shown above.

**4. Edit the settings** at the top of `Soil_Moisture.ino`:

```cpp
const char* WIFI_SSID     = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
const char* BOT_TOKEN     = "YOUR_BOT_TOKEN";
const char* CHAT_ID       = "YOUR_CHAT_ID";
```

**5. Calibrate** your sensor:

```cpp
int DRY_VALUE = 3200;   // raw reading in open air
int WET_VALUE = 1400;   // raw reading in a glass of water
```

**6. Upload** and open the Serial Monitor at `115200` baud to see the IP address.

**7. Open** `http://<ESP32-IP>/` on any phone or PC on the same Wi-Fi. 🎉

> ⚠️ **Never commit your real Wi-Fi password, bot token or chat ID.** Keep placeholders in the repo.

---

## ⚙️ How It Works

The probe's output voltage **falls as the soil gets wetter**, so a lower ADC value means wetter soil. The percentage comes from a two-point linear calibration, limited to 0-100 %:

```
Moisture (%) = (Raw − DRY_VALUE) × 100 ÷ (WET_VALUE − DRY_VALUE)
```

### 🎨 Condition levels

| Moisture | Condition | Colour | Action |
|:-:|---|:-:|---|
| Below 30 % | Dry, needs water | 🔴 | Alert sent |
| 30 – 59 % | Moist (good) | 🟢 | None |
| 60 % and above | Wet | 🔵 | None |

### 🔔 Alert logic (hysteresis)

```
        moisture drops below 30%                 moisture rises above 40%
  OK ───────────────────────────►  DRY  ───────────────────────────►  OK
        📩 dry alert sent          (repeat only after 30 min)       📩 recovery message
```

Between 30 % and 40 % nothing is sent, so small fluctuations don't spam you.

### ⏱️ Timing

| Task | Interval |
|---|:-:|
| Sensor reading | 10 s |
| Telegram polling | 1 s |
| Dashboard refresh | 5 s |
| History buffer | 120 readings (about 20 min, in RAM) |
| Alert cooldown | 30 min |

### 🌐 Endpoints and commands

| Endpoint | Returns |
|---|---|
| `/` | Dashboard page (HTML/CSS/JS from flash) |
| `/data` | JSON: raw value, RSSI, uptime, thresholds, history |

| Command | Reply |
|---|---|
| `/start` | Command list and dashboard address |
| `/moisture` | Moisture %, condition, raw ADC |
| `/status` | Moisture, Wi-Fi signal, uptime, dashboard address |

---

## 📸 Results

### 📊 Dashboard

<table>
<tr>
<td align="center"><img src="images/dashboard-moist.png" alt="Moist"><br><b>🟢 Moist (58 %)</b></td>
<td align="center"><img src="images/dashboard-dry.png" alt="Dry"><br><b>🔴 Dry + alert banner</b></td>
<td align="center"><img src="images/dashboard-wet.png" alt="Wet"><br><b>🔵 Wet (68 %)</b></td>
</tr>
</table>

### 💬 Telegram

<div align="center">
<img src="images/telegram-bot.png" alt="Telegram bot alerts" width="45%">
<br><sub>Start-up message, dry alerts, recovery messages and command replies</sub>
</div>

### 🧪 Observed readings

| Situation | Raw ADC | Moisture | Result |
|---|:-:|:-:|---|
| Probe in open air | 4095 | 0 % | 🔴 Dry, alert sent |
| Dry soil | 2855 | 19 % | 🔴 Dry |
| Moist soil | 2141 | 58 % | 🟢 Moist (good) |
| Wet soil | 1969 | 68 % | 🔵 Wet, recovery sent |

✅ Telegram alerts and the dashboard banner appeared together
✅ Recovery messages came only after moisture crossed 40 %
✅ `/moisture` and `/status` matched the dashboard values

---

## ⚠️ Limitations

- Dashboard works only on the same Wi-Fi (unless you set up port forwarding or a VPN)
- History is kept in RAM and resets on reboot
- Resistive probes corrode over time, so readings can drift
- Calibration depends on the sensor and soil type
- Monitoring only, it does not water the plant

## 🗺️ Future Scope

- [ ] Relay + pump for automatic watering
- [ ] LED / buzzer for local alerts
- [ ] Cloud or SD card logging for long-term trends
- [ ] DHT11 for temperature and humidity
- [ ] Battery + solar power with deep sleep
- [ ] Change thresholds from the dashboard or Telegram

---

<div align="center">

## 🙋 Author

**Galla Indranag**

B.Tech ECE
Embedded Systems & IoT Analyst (ESSCI-certified), (SRM AP)

GitHub: @indra675
LinkedIn: linkedin.com/in/gallaindranag

⭐ If this helped you, give the repo a star!

</div>
