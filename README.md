# Luce Labs - Sensor & Actuator Module
**Candidate Design Brief Submission**
Robotics and Embedded Engineering Intern

---

## What this is

This design controls one slice of a growing rack. It reads two things from the environment and uses both readings together to decide when to run an irrigation pump.

- **Moisture sensor:** reads how wet or dry the soil is
- **Temperature sensor:** reads how hot it is near the plant
- **Pump relay:** turns the irrigation pump on or off
- **Control logic:** lives inside Home Assistant, running on-site

Both sensor readings feed one decision together. Hot and dry soil triggers a longer watering cycle. Cool and moist soil skips irrigation entirely.

---

## Folder structure

```
/docs
  component-selection-matrix.md   - product options, scores, and final picks
  integration-and-control.md      - how everything connects, control logic, failure responses
  ai-usage-appendix.md            - AI tools used, real prompts, one case where AI was wrong

/ha-config
  esphome-bridge.yaml             - config that makes the moisture sensor work with Home Assistant
  automation.yaml                 - the automation that runs the pump based on sensor readings
  template-sensors.yaml           - helper calculations
  dashboard.yaml                  - simple visual display in Home Assistant

README.md                         - this file
milestone-tracker.md              - time log
```

---

## Where to start reading

Open `/docs/integration-and-control.md` first for the big picture. Then `/docs/component-selection-matrix.md` for the product choices. Then `/ha-config/` for the actual working code.

---

## Time spent

See `milestone-tracker.md` for a full breakdown.

---

## Contact

Daoud | daoud@roboticapro.com
