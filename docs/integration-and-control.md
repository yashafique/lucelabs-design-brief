# Integration and Control Design

---

## Block diagram

```
<img width="1973" height="798" alt="image" src="https://github.com/user-attachments/assets/5a6a5152-6ec5-41c7-9707-bb0c08e62b6e" />

```
Created via Whimsical.com

**Plain description:** Both sensors wire into a single ESP32 board. The ESP32 runs ESPHome firmware, which translates the raw sensor signals into two readable values and sends them to Home Assistant over WiFi. Home Assistant runs the control logic and sends on/off commands to the Shelly relay. The Shelly relay switches the pump.

---

## How each device joins Home Assistant

### ESP32 bridge (carries both sensors)

The ESP32 runs ESPHome. When it powers on it connects to WiFi and announces itself to Home Assistant automatically. No manual pairing steps; HA sees it as a device with two sensors.

Entities it exposes:
- `sensor.rack_soil_moisture` - moisture reading as a 0-100% value
- `sensor.rack_temperature` - temperature in degrees Celsius

Both entities appear in HA exactly like any bought smart sensor. You can use them in automations, history graphs, and dashboards without any extra setup.

### Shelly 1PM Gen3 (pump relay)

Plug it in and open the Shelly app once to connect it to your WiFi. After that, Home Assistant discovers it automatically through the built-in Shelly integration. No cloud account required; everything runs locally.

Entities it exposes:
- `switch.rack_pump` - turns the pump on or off
- `sensor.rack_pump_power` - live watt reading; used to confirm the pump is actually running

---

## Bridging design: SEN0193 moisture sensor

### Why it needs bridging

The SEN0193 is not a smart device. It has no WiFi or Zigbee radio. It puts out a plain electrical voltage between 1.2V (wet soil) and 3.1V (dry soil) on two wires. Home Assistant has no way to read that directly.

### Hardware

Wire the SEN0193 to the ESP32 as follows:

| SEN0193 pin | ESP32 pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| AOUT (signal) | GPIO34 |

GPIO34 is one of the analog-capable pins on the ESP32. A 4.7k resistor between GPIO4 and 3.3V is also needed for the DS18B20 temperature probe on the same board.

### Firmware config

See `/ha-config/esphome-bridge.yaml`.

The key section is the `adc` sensor block. ESPHome reads the raw voltage from GPIO34 sixty times per second, averages it, and applies a calibration map that converts the voltage to a 0-100% moisture scale. The result is sent to HA as `sensor.rack_soil_moisture`.

### How it appears in Home Assistant

Once the ESP32 connects to WiFi, HA shows it under Settings > Devices with two sensor entities. The moisture entity has a unit of `%`, an icon, and a history graph. It is indistinguishable from a native smart sensor. You can swap the ESP32 or swap the SEN0193 independently without changing any HA automations, because the entity names stay the same.

---

## Control logic

### The decision

The pump runs when two conditions are both true at the same time:
1. Soil moisture is below a dry threshold
2. Temperature is above a minimum level

Temperature on its own does not trigger the pump. Moisture on its own does not trigger the pump. Both are required. This satisfies the brief requirement that both parameters genuinely drive the single output.

### How temperature modifies the decision

Temperature controls how long the pump runs per cycle. The logic works like this:

| Soil moisture | Temperature | Pump action |
|---|---|---|
| Above 40% (wet enough) | Any | Pump OFF |
| Below 40% and above 50C (implausible) | Any | Pump OFF; sensor fault alert |
| Below 40% | Below 15C | Short cycle: 30 seconds |
| Below 40% | 15C to 25C | Medium cycle: 60 seconds |
| Below 40% | Above 25C | Long cycle: 90 seconds |

Warmer air means faster evaporation from the soil, so the plant needs more water. Cooler air means slower evaporation, so a shorter cycle is enough. Both readings drive the single decision together.

### Hysteresis

The pump does not turn on the instant moisture drops below 40%. It turns on at 40% and does not turn off until moisture recovers to 50%. This 10-point gap prevents the pump from rapidly switching on and off around the threshold.

### Anti-short-cycling

Once the pump runs a cycle, it cannot run again for at least 10 minutes. This protects the pump motor from wear caused by rapid restarts and gives the moisture reading time to stabilize after watering.

---

## Failure modes and safe states

The safe state in every case is: **pump off**. Running the pump with bad data risks overwatering. Doing nothing is always the safer error.

| Failure | How HA detects it | Safe state | Alert sent |
|---|---|---|---|
| Moisture sensor offline | `sensor.rack_soil_moisture` goes to `unavailable` for 5+ minutes | Pump stays off | Yes; persistent notification in HA |
| Temperature sensor offline | `sensor.rack_temperature` goes to `unavailable` for 5+ minutes | Pump stays off | Yes |
| Moisture reading implausible | Value above 100% or below 0% | Treat as offline; pump stays off | Yes |
| Temperature reading implausible | Value above 50C or below -10C | Treat as offline; pump stays off | Yes |
| Pump commanded ON but power draw is 0W | Shelly power sensor reads 0W within 5 seconds of pump being switched on | Log fault; try once more; then alert | Yes |
| Pump stuck ON (power draw when commanded OFF) | Shelly power sensor reads above 5W when switch is off | Log fault; alert immediately | Yes; high priority |
| HA restart | Automation reloads on boot | Pump defaults to OFF on startup | No |
| WiFi loss on ESP32 | Entities go unavailable | Pump stays off; recovers automatically when WiFi returns | Yes if offline for 10+ minutes |
| Shelly relay loses WiFi | `switch.rack_pump` goes unavailable | HA cannot command pump; logs unavailable state | Yes |

### Note on the stuck-pump case

The Shelly 1PM Gen3 was chosen specifically because it has a built-in watt meter. Most smart relays do not have this. The power reading is what makes stuck-pump detection possible without adding any extra hardware.

---

## Bench-test order

Test in this order. Each step must pass before moving to the next.

1. Flash ESPHome to the ESP32. Confirm it appears in HA under Settings > Devices with two sensor entities showing real values.

2. Put the SEN0193 in dry soil; note the moisture reading. Put it in wet soil; note the reading. Confirm the calibration in the YAML maps these to sensible percentages.

3. Compare the DS18B20 temperature reading against a known thermometer. Confirm it is within 1 degree.

4. In HA, use Developer Tools to manually set the moisture entity to a value below 40% and the temperature entity to a value above 25C. Confirm the automation fires and the pump switch turns on for 90 seconds.

5. Repeat step 4 for each temperature band (below 15C; 15-25C; above 25C). Confirm the pump runtime matches the table above.

6. Disconnect the ESP32 from power. Wait 6 minutes. Confirm HA sends an alert notification and the pump stays off.

7. Reconnect the ESP32. Confirm entities recover and the alert clears.

8. Command the pump ON in HA. Confirm the Shelly power sensor reads above 0W within 5 seconds.

9. Run the full system unattended for 24 hours. Check the history graph for any sensor dropouts or unexpected pump cycles.

10. Simulate an HA restart. Confirm the pump defaults to off on startup and the automation reloads correctly.
