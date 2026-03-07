# Mercedes-W447
This is a repository for my work around the W447 V-class
# 🚗 LIN Bus HVAC Reverse Engineering Project

This repository contains the protocol documentation and ESPHome configuration for intercepting and monitoring the LIN bus traffic of a vehicle's HVAC system. 

## 🛠 Project Overview
The goal is to map the button presses, LED states, and auxiliary heater (STH) metrics into Home Assistant. This allows for real-time monitoring of climate controls and heater status.

### Hardware Requirements
* **Microcontroller:** ESP32 (Recommended for Hardware UART support).
* **LIN Transceiver:** TJA1020 or MCP2003/2004 module.
* **Power:** 12V to 5V DC-DC Buck Converter.



---

## 📑 PID Documentation Index
Each LIN Message ID (PID) has been isolated and mapped.

| PID | Component | Primary Function | Status |
| :--- | :--- | :--- | :--- |
| [**0x0D**](./PIDs/0x0D.md) | Rear HVAC Panel | Passenger Temp & Fan controls | **Verified** |
| [**0x49**](./PIDs/0x49.md) | Central Buttons | Menu, Aux Heat, Demist, Circulation | **Verified** |
| [**0x4C**](./PIDs/0x4C.md) | Front HVAC Panel | Fan Speed, Dual-zone Temp, Direction | **Verified** |
| [**0x76**](./PIDs/0x76.md) | Y141 Valve | Coolant valve position feedback (%) | **Verified** |
| [**0x78**](./PIDs/0x78.md) | Aux Heater (STH) | Coolant Temp, Operating Mode | **Partial** |
| [**0x8B**](./PIDs/0x8B.md) | Status LEDs | Physical indicator lights | **Verified** |
| [**0xDD**](./PIDs/0xDD.md) | System Debug | High-traffic state data | **Researching** |

---

## ⚠️ Known Issues & Unknowns
* **Fuel Pump (PID 0x78):** Byte 3 is confirmed to change with pump activity, but the scaling factor is **Unknown**. Do not assume `* 0.05` is accurate.
* **PID 0xDD:** This PID is under active investigation. It appears to be a global state broadcast.

## 🚀 Usage
To use this in ESPHome, refer to the logic snippets provided in each PID file or the sample `hvac_lin.yaml` in the root directory.
