# PID 0xDD: Button Panel Backlight Control

**Length:** 8 Bytes  
**Function:** Controls the illumination intensity of the HVAC button panel and tracks the dimming state.

## Byte Breakdown

| Byte | Name | Description |
| :--- | :--- | :--- |
| 0 | Header | Static value `0xE4` |
| 1 | Sync A | Static value `0x28` |
| 2 | Sync B | Static value `0x85` |
| 3 | **Intensity** | Primary backlight level. Range: `0x00` to `0x63`. |
| 4 | **Status** | Secondary state or Target. Usually mirrors Byte 3 during active dimming. |
| 5 | Footer | Transitions between `0xE4` (Active) and `0xD4` (Standby/Off). |
| 6 | Padding | `0xFF` |
| 7 | Padding | `0xFF` |

## Intensity Mapping (Byte 3)

The intensity follows a linear scale from off to full brightness. While the technical range is a full byte, the software-defined limits observed are:

* **`0x63` (99 Dec):** 100% Brightness (Maximum).
* **`0x0C` (12 Dec):** Minimum active brightness.
* **`0x00` (00 Dec):** Backlight Off.

**Scaling Formula:** `Brightness % = (Byte_3 / 99) * 100`

## Observed Behavior

### 1. Active Dimming
During an active dimming event (e.g., using the instrument cluster menu or a light sensor trigger), Byte 3 and Byte 4 increment or decrement in sync.
* Example (High): `E4.28.85.63.63.E4.FF.FF`
* Example (Low): `E4.28.85.0C.0C.E4.FF.FF`

### 2. Standby / Override Mode
When the backlight is commanded off or enters a standby state, Byte 4 frequently switches to `0xFE` while Byte 5 shifts to `0xD4`. This indicates the panel is no longer actively following the illumination bus but is in a "Ready" or "Sleep" state.
