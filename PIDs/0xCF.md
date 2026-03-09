## PID 0x0D: Rear HVAC Control (Request)
**Logic:** Absolute state reporting.
| Byte | Function | Values |
| :--- | :--- | :--- |
| 2 | Fan Speed | 00=Off, 11=S1 ... 77=S7 |
| 3/4 | Temp L/R | Raw + 0x60 (69 = 20°C) |

## PID 0xCF: Rear HVAC Display (Status)
**Logic:** Instructs the LCD hardware what to show.
| Byte | Function | Values / Notes |
| :--- | :--- | :--- |
| 0/1 | Temp L/R | Raw + 0x40 (40=LO, 49=20°C, 5E=HI) |
| 2 | Fan Speed | 00 triggers "OFF" text; 11-77 shows speed bars |
| 3 | Icons | Airflow direction symbols |
| 4 | Config | 03 = Celsius. Likely 0x83 or similar for AUTO |
