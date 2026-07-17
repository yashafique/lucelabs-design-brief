# Component Selection Matrix

*Status: Complete. All links verified July 2026.*

---

## Design summary

**Moisture sensor** reads whether the soil is dry. **Temperature sensor** reads how hot it is. Together they control one output: the irrigation pump. Hot and dry soil means a longer watering cycle. Cool and moist soil means the pump stays off. Both readings drive the same decision; this is not two separate loops.

---

## Sensor 1 - Substrate Moisture (non-native; bridged through ESP32)

This is the required non-native sensor. It puts out a plain electrical voltage, not a WiFi or Zigbee signal. A small microcontroller (ESP32) reads that voltage and translates it into something Home Assistant can understand.

| | **Option A** | **Option B** | **Option C** |
|---|---|---|---|
| **Product** | Adafruit STEMMA Soil Sensor | DFRobot Gravity Capacitive Moisture Sensor (SEN0193) | Seeed XIAO Soil Moisture Sensor |
| **Vendor** | Adafruit | DFRobot | Seeed Studio |
| **Price** | $7.50 | $5.90 | $9.90 |
| **Link** | [adafruit.com/product/4026](https://www.adafruit.com/product/4026) | [dfrobot.com/product-1385.html](https://www.dfrobot.com/product-1385.html) | [seeedstudio.com/XIAO-Soil-Sensor-p-6452.html](https://www.seeedstudio.com/XIAO-Soil-Sensor-p-6452.html) |
| **How it connects to HA** | I2C wire to ESP32; requires a custom driver; appears in HA as a sensor | Analog voltage to ESP32 pin; standard ESPHome config; appears in HA as a sensor | Comes pre-programmed with WiFi; shows up in HA automatically |
| **Reading quality** | Relative moisture units (200-2000); also reads temperature | Relative moisture voltage (1.2-2.5V); no metal exposed; corrosion-resistant | Three levels only: dry, almost dry, normal |
| **21-day reliability** | Good; no metal corrosion risk; needs a non-standard driver | Good; simplest wiring; no firmware complications | Best; battery-powered; pre-calibrated |
| **Total cost** | $7.50 + $6 ESP32 = $13.50 | $5.90 + $6 ESP32 = $11.90 | $9.90 all-in |
| **Availability** | In stock | In stock | In stock |

**Selected: DFRobot SEN0193 (Option B)**

It outputs a simple voltage that requires no special driver and no workarounds. You wire it to the ESP32, write a short config in ESPHome, and it shows up in Home Assistant like any other sensor. This is exactly the bridging work the brief is testing. Option A has a chip with non-standard registers that ESPHome does not fully support. Option C comes pre-programmed and auto-connects to HA; it skips the bridging challenge entirely and defeats the purpose of the exercise.

---

## Sensor 2 - Air Temperature (natively supported)

This sensor plugs into the same ESP32 as the moisture sensor. No extra hardware needed.

| | **Option A** | **Option B** | **Option C** |
|---|---|---|---|
| **Product** | Adafruit Waterproof DS18B20 Temp Sensor | DFRobot Waterproof DS18B20 (DFR0198) | Seeed Grove DHT20 Temp + Humidity |
| **Vendor** | Adafruit | DFRobot | Seeed Studio |
| **Price** | $9.95 | $9-12 via distributors | $4-6 |
| **Link** | [adafruit.com/product/381](https://www.adafruit.com/product/381) | [dfrobot.com/product-689)](https://www.dfrobot.com/product-689.html) |[ [seeedstudio.com/Grove-Temperature-Humidity-Sensor-V2-0-DHT20](https://www.seeedstudio.com/Grove-Temperature-Humidity-Sensor-V2-0-DHT20-p-4967.html?srsltid=AfmBOopA1hSmKQyWOShcOMEyVEURTUnyo936wlyeZ1ju2JuFtBjgMYoC)|
| **How it connects to HA** | Single wire to ESP32; built-in ESPHome support; shares same ESP32 as moisture sensor | Same as Option A | I2C wire to same ESP32 |
| **Accuracy** | +/- 0.5 C across most of its range | Same chip, same specs | +/- 0.3 C typical; also reads humidity |
| **21-day reliability** | Excellent; waterproof stainless steel probe; Adafruit screens for counterfeit chips | Good; same chip; less screening | No waterproofing; exposed to water splash |
| **Total cost** | No added cost; shares the ESP32 already purchased | No added cost | No added cost |
| **Availability** | Out of stock at Adafruit; order through DigiKey link on product page | In stock via RS/Mouser | In stock |

**Selected: DS18B20 via DigiKey (Option A)**

The probe is waterproof and can sit in the soil right next to the moisture sensor. It measures temperature where it actually matters for irrigation decisions, not ambient air temperature. ESPHome has built-in support; no custom driver needed. Important note: Adafruit's own page shows this out of stock; you must click through to DigiKey. The matrix AI check caught this discrepancy on manual verification.

---

## Actuator - Irrigation Pump Relay

The relay is a smart switch that turns the pump on and off. Home Assistant controls the relay; the relay controls the pump.

| | **Option A** | **Option B** | **Option C** |
|---|---|---|---|
| **Product** | Shelly 1PM Gen3 | Sonoff BASIC R4 | Sonoff MINIR4M (Matter) |
| **Vendor** | Shelly USA | Sonoff | Sonoff |
| **Price** | $19.24 (currently 30% off) | $7-10 | $12-15 |
| **Link** | [us.shelly.com/products/shelly-1pm-gen3](https://us.shelly.com/products/shelly-1pm-gen3) | [itead.cc/product/sonoff-basicr4-wi-fi-smart-switch](https://itead.cc/product/sonoff-basicr4-wi-fi-smart-switch/) | [sonoff.tech](https://sonoff.tech) |
| **How it connects to HA** | WiFi; native HA integration; works without internet | WiFi; requires cloud account or firmware swap for local use | WiFi + Matter; works without internet |
| **Max load** | 16A | 10A | 10A |
| **Power monitoring** | Yes; built-in watt meter; can detect if pump actually ran | No | No |
| **21-day reliability** | Excellent; no internet needed; 3-year warranty; built-in overload protection | Good; cloud dependency is a risk for unattended use | Good; Matter is local; newer product with less track record |
| **Availability** | In stock | In stock | In stock |

**Selected: Shelly 1PM Gen3 (Option A)**

It works fully without an internet connection. If the WiFi router loses internet for a day, the automation keeps running. The built-in power meter lets Home Assistant confirm the pump actually turned on and is drawing power; this enables real failure detection. The Sonoff R4 needs either a cloud account or a firmware swap to work locally. The MINIR4M is newer and has less real-world history.

---

## Total cost

| Part | Product | Price |
|---|---|---|
| Moisture sensor | DFRobot SEN0193 | $5.90 |
| Temperature sensor | DS18B20 waterproof probe | $9.95 |
| Bridge (ESP32 board) | Generic ESP32 dev board | ~$6.00 |
| Pump relay | Shelly 1PM Gen3 | $19.24 |
| **Total** | | **~$41** |

---

## Weakest buy and build fallback

**Weakest component:** DFRobot SEN0193 moisture sensor.

The SEN0193 gives a relative reading; it can tell you the soil got wetter or drier, but it cannot tell you the exact percentage of water in the soil. The reading also shifts depending on soil type and how deep the probe is inserted. For basic irrigation triggering this is acceptable, but it is not research-grade.

**When to build instead:** If the project needed precise, calibrated moisture levels (for example, to compare results across different soil types or to dose nutrients accurately), the SEN0193 would not be reliable enough. That is the threshold.

**Build fallback:** Keep the same ESP32. Replace the SEN0193 with a bare capacitive sensor circuit wired to the same analog pin. Add a three-point calibration routine in ESPHome (measure in air, in dry soil, in saturated soil) to map raw readings to real moisture percentages. This takes roughly two extra hours per sensor type but produces a calibration that holds across soil conditions. The rest of the design stays the same.
