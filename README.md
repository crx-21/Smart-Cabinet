<div align="center">
  
# Smart Cabinet System

<img src="https://img.shields.io/badge/Arduino-Uno%20R3-00979D?style=for-the-badge&logo=arduino&logoColor=white"/>
<img src="https://img.shields.io/badge/React%20Native-Expo-000020?style=for-the-badge&logo=expo&logoColor=white"/>
<img src="https://img.shields.io/badge/Firebase-RTDB-FFCA28?style=for-the-badge&logo=firebase&logoColor=black"/>
<img src="https://img.shields.io/badge/Language-C%20%2F%20TypeScript-blue?style=for-the-badge&logo=c&logoColor=white"/>
<img src="https://img.shields.io/badge/Status-Prototype%20v0.1-brightgreen?style=for-the-badge"/>


**IoT-enabled remote cabinet lock control — accessible from anywhere on Earth.**

*Arduino Uno R3 · ESP8266 · DS3231 RTC · Firebase RTDB · React Native + Expo*

</div>

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Tech Stack](#-tech-stack)
- [Mobile Application](#-mobile-application)
  - [Screens](#screens)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
- [Roadmap](#-roadmap)

---

## 🌐 Overview

The **Smart Cabinet System** is a cloud-connected IoT prototype that gives you secure, remote control over physical cabinet locks — from across the room or across the country (30 km+, limited only by internet reach).

The system consists of **3 independently addressable cabinet units**, each fitted with a 12V solenoid lock. An **Arduino Uno** running bare-metal C firmware drives the locks via an I2C relay expander. An **ESP8266 Wi-Fi module** bridges the Arduino to **Firebase Realtime Database**, and a **React Native (Expo)** mobile app provides the user interface.

---

## ✨ Key Features

| Feature | Detail |
|---|---|
| 🌍 **Remote Access** | Unlock / lock any cabinet from anywhere via the mobile app — cloud-routed, no VPN or port-forwarding needed |
| ⏰ **RTC Scheduling** | Program cabinets to unlock at specific dates and times using the DS3231 real-time clock module |
| 📴 **Offline-Safe** | Scheduled events fire from Arduino EEPROM even when Wi-Fi or Firebase is temporarily unreachable |
| 📱 **Mobile App** | Cross-platform React Native app (iOS + Android) with real-time lock status via Firebase listeners |
| 🔐 **Authenticated Access** | Firebase Authentication gates all commands — no anonymous access |
| 📈 **Built to Scale** | I2C bus supports up to 128 cabinets; Firebase schema and app UI are dynamically driven by cabinet count |
| 🛠️ **Bare-Metal Firmware** | Written in direct C with AVR-GCC — no Arduino HAL overhead, full register-level control |

---

## 🏗 System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         MOBILE APP (Expo)                           │
│              React Native · TypeScript · Firebase SDK               │
└────────────────────────────┬────────────────────────────────────────┘
                             │  Firebase Auth + HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   FIREBASE REALTIME DATABASE                        │
│                                                                     │
└────────────────────────────┬────────────────────────────────────────┘
                             │  HTTPS REST polling (500 ms)
                             ▼
┌───────────────────────────────────────────┐
│           ESP8266 ESP-01 (Wi-Fi)          │
│   Polls Firebase · Forwards via UART      │
└──────────────────┬────────────────────────┘
                   │  UART 9600 baud (JSON)
                   ▼
┌───────────────────────────────────────────┐
│           Arduino Uno R3                  │
│           C · I2C Host ·                  │
└───┬──────────────┬────────────────────────┘
    │ I2C           │ I2C
    ▼               ▼
┌────────┐    ┌──────────────────────────────────────┐
│DS3231  │    │  PCF8574 I/O Expander (0x20)         │
│  RTC   │    │  8 relay outputs → 3 solenoid locks  │
└────────┘    └──────┬──────────┬───────────┬─────────┘
                     │          │           │
               ┌─────▼─┐  ┌────▼──┐  ┌────▼──┐
               │ Lock 1│  │ Lock 2│  │ Lock 3│
               │  12V  │  │  12V  │  │  12V  │
               └───────┘  └───────┘  └───────┘
```

### Communication Flow

**Scenario 1:**

1. **User taps Unlock** in the mobile app
2. App writes command to unlock specific cabinet in Firebase RTDB
3. **ESP8266** polls Firebase over HTTPS every ~500 ms, detects the new command
4. ESP8266 forwards command to the Arduino over **UART**
5. Arduino parses the JSON, calls unlock function, activates the **relay** via PCF8574 over I2C
6. Solenoid releases. Arduino writes `status: "unlocked"` back to Firebase via ESP8266
7. **App UI updates in real time** via Firebase `onValue()` listener — no polling needed on the client

**Scenario 2:**
1. **User sets date and time** for a cabinet in the mobile app.
2. App writes command to set scheduled unlock in Firebase RTDB.
3. **ESP8266** polls Firebase over HTTPS every ~500 ms, detects the new command
4. ESP8266 forwards command to the Arduino over **UART**
5. Arduino parses the JSON, calls set function, activates the **relay** via PCF8574 over I2C
6. Arduino writes `status: "set"` back to Firebase via ESP8266
7. **App UI updates in real time** via Firebase `onValue()` listener — no polling needed on the client

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| **Microcontroller** | Arduino Uno R3 — ATmega328P (16 MHz, 32KB Flash, 2KB SRAM, 1KB EEPROM) |
| **Firmware Language** | C — bare-metal AVR-GCC (no Arduino HAL) |
| **Wi-Fi Bridge** | ESP8266 ESP-01 (AT firmware over UART, HTTPS capable) |
| **Real-Time Clock** | DS3231 (I2C, ±2 ppm accuracy, battery-backed) |
| **I/O Expander** | PCF8574 (I2C, 8-bit, addresses 0x20–0x27) |
| **Cloud Database** | Firebase Realtime Database (JSON tree, WebSocket sync) |
| **Authentication** | Firebase Authentication (email/password) |
| **Mobile Framework** | React Native with Expo SDK |
| **Mobile Language** | TypeScript |
| **State Management** | React Query + Zustand |
| **Navigation** | Expo Router (file-based) |
| **Target OS** | iOS 15+ · Android 10+ |

---

## 📱 Mobile Application

### Screens

| Screen | Description |
|---|---|
| **Login** | Firebase email/password auth. No anonymous access permitted. |
| **Dashboard** | All 3 cabinets as status cards with live lock/unlock state via Firebase listener. |
| **Cabinet Detail** | Per-cabinet manual unlock / lock buttons, status badge, and schedule list. |
| **Schedule Manager** | Add / delete unlock schedules with a date-time picker. Syncs to Firebase → Arduino EEPROM within 30 s. |
| **Settings** | User profile, sign out, notification preferences. |

## 🚀 Getting Started

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) or `avrdude` CLI + `avr-gcc` toolchain
- [Node.js](https://nodejs.org/) v18+
- [Expo CLI](https://docs.expo.dev/get-started/installation/) — `npm install -g expo-cli`
- A [Firebase project](https://console.firebase.google.com/) with Realtime Database + Authentication enabled
- Physical hardware

## 🗺 Roadmap

- [x] Core hardware architecture (Arduino + ESP8266 + DS3231 + PCF8574)
- [x] PRD & system design documentation
- [ ] I2C driver implementation (`i2c.c`)
- [ ] DS3231 RTC driver (`rtc_ds3231.c`)
- [ ] PCF8574 + relay control (`pcf8574.c`, `relay_ctrl.c`)
- [ ] EEPROM schedule manager (`schedule.c`)
- [ ] ESP8266 UART bridge + Firebase polling (`esp_comm.c`)
- [ ] React Native dashboard with live Firebase status
- [ ] Schedule creation UI + Firebase sync
- [ ] Firebase Authentication integration
- [ ] End-to-end integration test (3 cabinets, 30 km+ distance)
- [ ] Push notifications on scheduled unlock events
- [ ] Audit log (timestamped Firebase event history)
- [ ] Multi-user role support (Firebase Custom Claims)
- [ ] OTA firmware update via ESP8266
- [ ] Custom PCB design (move off breadboard)

---

<div align="center">
Built for family and personal use.📈
</div>
