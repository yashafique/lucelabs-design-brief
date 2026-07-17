# Luce Labs — Sensing & Actuation Rack Slice
**Candidate Design Brief Submission**  
Robotics & Embedded Engineering Intern

---

## What this is

A complete design for one sensing-and-actuation slice of a controlled growing rack, built within the Luce Labs IRIS architecture:

- **Two sensed parameters:** Air Temperature + Relative Humidity
- **Derived control variable:** VPD (Vapour Pressure Deficit) — a single number combining both inputs
- **One actuator:** Exhaust fan (speed-controlled)
- **Control logic lives in:** Home Assistant (on-site, unattended 21-day operation)

The design favors off-the-shelf components connected over open protocols, with one non-native sensor bridged cleanly into HA via ESP32 + ESPHome.

---

## Repo structure

```
/docs
  component-selection-matrix.md   ← sensor & actuator sourcing, scoring, buy-vs-build
  integration-and-control.md      ← block diagram, bridging design, control logic, failure modes
  ai-usage-appendix.md            ← AI tools used, real prompts, where AI was wrong

/ha-config
  esphome-bridge.yaml             ← firmware config for the ESP32 bridge node
  automation.yaml                 ← HA automation implementing VPD → fan control
  template-sensors.yaml           ← VPD calculation and helper sensors
  dashboard.yaml                  ← Lovelace dashboard sketch

README.md                         ← this file
milestone-tracker.md              ← time log and progress tracking
```

---

## How to review the design

Start with **`/docs/integration-and-control.md`** for the block diagram and system overview, then **`/docs/component-selection-matrix.md`** for sourcing decisions, then **`/ha-config/`** for the working configuration files.

---

## Time spent

*See `milestone-tracker.md` for a phase-by-phase breakdown.*

---

## Contact

Yahya Shafique | Yahyashafique05@gmail.com
