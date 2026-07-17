# Integration & Control Design

*Status: In progress*

## Block diagram

*(Insert diagram image here — draw in Excalidraw or draw.io, export as PNG, drop in /docs/assets/)*

```
[Temp/RH Sensor] ──Matter/Zigbee──┐
                                   ├──► [Home Assistant] ──► [Automation] ──► [Fan Controller]
[Non-native Sensor]─I2C─[ESP32]─ESPHome/WiFi─┘
```

---

## How each device joins HA

### Native temp/RH sensor
- Joins via: TBD
- Entities exposed: `sensor.rack_temperature`, `sensor.rack_humidity`

### Non-native sensor (ESP32 bridge)
- Joins via: ESPHome + WiFi
- Entities exposed: TBD
- Appears in HA as: a first-class entity, indistinguishable from a native device

### Fan actuator
- Joins via: TBD
- Entities exposed: `fan.rack_exhaust`

---

## Bridging design (non-native sensor)

### Hardware
TBD

### ESPHome firmware config
*(See `/ha-config/esphome-bridge.yaml`)*

### How it appears in HA
TBD

---

## Correlation logic — VPD → fan control

**VPD** (Vapour Pressure Deficit) is a single number derived from both temperature and humidity.  
Formula: `VPD = (1 - RH/100) × 0.6108 × exp(17.27 × T / (T + 237.3))`

| VPD Range | Plant stress level | Fan action |
|---|---|---|
| < 0.4 kPa | Too humid | Fan OFF |
| 0.4 – 1.2 kPa | Ideal | Fan at low speed |
| > 1.2 kPa | Too dry | Fan at high speed |

**Hysteresis:** TBD (e.g. 0.1 kPa deadband)  
**Anti-short-cycling:** TBD (e.g. minimum 5-min run time)

---

## Failure modes & safe states

| Failure | Detection | Safe state |
|---|---|---|
| Temp sensor offline | Entity unavailable in HA | Hold last known fan state; alert |
| RH sensor offline | Entity unavailable in HA | Hold last known fan state; alert |
| Stuck/implausible reading | Value out of expected range | Treat as offline |
| Fan stuck on | TBD | TBD |
| HA restart | Automation re-loads on boot | Fan defaults to low speed |
| Network loss (ESP32) | Entity goes unavailable | HA holds last state; alert |

---

## Bench-test order

1. Confirm ESP32 flashes and appears in HA as an entity
2. Confirm native sensor pairs and reports realistic values
3. Confirm VPD template calculates correctly against manual formula check
4. Trigger automation manually using HA helpers; confirm fan responds
5. Simulate sensor dropout; confirm safe state engages
6. Run for 24 hours unattended; check for dropouts or drift
7. Simulate HA restart; confirm automation reloads and fan defaults correctly
