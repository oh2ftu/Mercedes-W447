# LIN-Bus Protocol Documentation: Vehicle Stationary Heater (STH)

This documentation tracks the reverse-engineered LIN-bus PIDs for automotive auxiliary/stationary heaters (STH). It is intended for use with ESPHome, Arduino, or other DIY sniffing tools to integrate factory heater data into smart home platforms like Home Assistant.

## Overview
- **Protocol:** LIN-Bus
- **Primary PID:** `0x5E` (Timer Data, Mode, and Runtime)
- **Secondary PID:** `0x8B` (Panel Indicators/LEDs)

---

## PID 0x5E: Master Timer & Status
This frame is the primary source of heater scheduling information. It transmits at a regular interval and contains the configuration for all three timer slots.

### Byte Structure

| Byte | Parameter | Bitmask | Description |
| :--- | :--- | :--- | :--- |
| **B0** | **Selection & Timer A Hour** | `0x60` | **Selection Bits:** `01`=A, `10`=B, `11`=C |
| | | `0x1F` | **Hour:** (0–23) |
| **B1** | **Timer A Minutes** | `0x3F` | **Minutes:** (0–59) |
| **B2** | **Timer B Hour** | `0x1F` | **Hour:** (0–23) |
| **B3** | **Timer B Minutes** | `0x3F` | **Minutes:** (0–59) |
| **B4** | **Timer C Hour** | `0x1F` | **Hour:** (0–23) |
| **B5** | **Operating Mode** | `N/A` | `0x0A`: Starting / Glow Phase<br>`0x4A`: Active / Countdown phase |
| **B6** | **Remaining Time** | `N/A` | Minutes remaining in current cycle. `0x3F` (63) = Idle. |

> [!NOTE]
> **Data Masking:** The high bits in the Hour and Minute bytes often contain status flags (like whether the timer is currently being edited). Always apply the bitmask (e.g., `payload[0] & 0x1F`) to extract the clean time value.

---

## PID 0x8B: Indicator Status
This PID tracks the state of the LEDs on the physical HVAC/Heater control panel.

- **Byte 5, Bit 2 (`0x04`):** **Timer Armed.** Indicates the heater is "Waiting" for a scheduled start time.
- **Byte 5, Bit 0 (`0x01`):** **Heater Running.** Indicates the heater is actively burning.

---

## ESPHome Implementation Logic

To accurately show which timer is active, the logic must combine the "Armed" bit from `0x8B` with the "Selection" bits from `0x5E`.

### Logic Flow
1. **Format Strings:** Convert the raw hex bytes for A, B, and C into `HH:MM` strings using `sprintf`.
2. **Determine Active Slot:** Right-shift Byte 0 of `0x5E` by 5 bits (`(B0 >> 5) & 0x03`).
3. **Master Check:** If `0x8B` Bit 2 is high, display the string for the corresponding active slot. Otherwise, report "OFF".

### Code Snippet (C++)
```cpp
// --- PID 0x5E Processing ---
if (pid == 0x5E && payload.size() >= 7) {
    char timeA[8], timeB[8], timeC[8];
    
    // Extract times with bitmasks
    sprintf(timeA, "%02d:%02d", payload[0] & 0x1F, payload[1] & 0x3F);
    sprintf(timeB, "%02d:%02d", payload[2] & 0x1F, payload[3] & 0x3F);
    sprintf(timeC, "%02d:%02d", payload[4] & 0x1F, payload[5] & 0x3F);

    // Update individual sensors
    id(sth_timer_a).publish_state(timeA);
    id(sth_timer_b).publish_state(timeB);
    id(sth_timer_c).publish_state(timeC);

    // Update Status with Selection Logic
    uint8_t selection = (payload[0] >> 5) & 0x03;
    if (id(led_aux_tmr).state) { // Check if Timer is Armed (from 0x8B)
        if (selection == 1)      id(sth_timer_status).publish_state("Timer A: " + std::string(timeA));
        else if (selection == 2) id(sth_timer_status).publish_state("Timer B: " + std::string(timeB));
        else if (selection == 3) id(sth_timer_status).publish_state("Timer C: " + std::string(timeC));
    } else {
        id(sth_timer_status).publish_state("OFF");
    }

    // Remaining Runtime
    id(sth_time_remaining).publish_state(payload[6] == 0x3F ? 0 : payload[6]);
}
