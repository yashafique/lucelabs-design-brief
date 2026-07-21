# Component Selection Matrix


---

## Design summary

**Moisture sensor** reads whether the soil is dry. **Temperature sensor** reads how hot it is at the root zone. Together they control one output: the irrigation pump. Moisture decides whether to water at all. Temperature decides how long the cycle runs, because warmer air pulls water out of the substrate faster. Both readings drive the same decision, as this is not two separate loops.

---

## Sensor 1 - Substrate Moisture

This sensor is not a smart device. It puts out a plain electrical voltage, not a WiFi or Zigbee signal. A small microcontroller (ESP32) reads that voltage and translates it into a value Home Assistant can use.

| | **Option A** | **Option B** | **Option C** |
|---|---|---|---|
| **Product** | Adafruit STEMMA Soil Sensor | DFRobot Gravity Capacitive Moisture Sensor (SEN0193) | Seeed XIAO Soil Moisture Sensor |
| **Vendor** | Adafruit | DFRobot | Seeed Studio |
| **Price** | $7.50 | $5.90 | $9.90 |
| **Link** | [adafruit.com/product/4026](https://www.adafruit.com/product/4026) | [dfrobot.com/product-1385.html](https://www.dfrobot.com/product-1385.html) | [seeedstudio.com/XIAO-Soil-Sensor-p-6452.html](https://www.seeedstudio.com/XIAO-Soil-Sensor-p-6452.html) |
| **How it reaches HA** | I2C to ESP32; needs a custom external component; then a HA entity | Analog voltage to ESP32 ADC pin; standard ESPHome `adc` block; then a HA entity | Pre-flashed firmware; joins HA over WiFi on its own |
| **Reading quality** | Relative units 200-2000; also reports temperature | Relative voltage 1.2-3.1V; qualitative, not calibrated volumetric water content | Three states only: dry, almost dry, normal |
| **21-day reliability** | Good; capacitive, so no metal corrosion. Depends on a third-party component staying maintained | Good; capacitive, no corrosion. No firmware layer to break | Good; but the sensing element and the radio fail together |
| **Total cost** | $7.50 + $6 ESP32 = $13.50 | $5.90 + $6 ESP32 = $11.90 | $9.90 all-in |
| **Availability** | In stock | In stock | In stock |

**Selected: DFRobot SEN0193 - Non-Native (Option B)**


First, the signal path is fully inspectable. It is a voltage on a wire. If the reading looks wrong I can put a meter on it and know if the fault is the probe, the wiring, or the calibration. 
Options A and C both hide the sensing element behind firmware I did not write and cannot easily probe.

Second, it has no software dependency. The Adafruit STEMMA needs a external component because its I2C registers are not covered by stock ESPHome. That is a maintenance liability on a rack expected to run unattended, and it puts a unnecessary third party between me and a working sensor.

Third, replaceability. At $5.90 with two wires and a power pin, this part can be swapped by anyone with no HA changes, because the entity is defined on the ESP32, not on the probe. However, using Seeed XIAO is the opposite: swapping it means re-onboarding a new WiFi device.

The tradeoff I am accepting is that Option C would be less work to set up and Option A gives a second temperature reading for free. Yet owning the signal path is most preferable.

---

## Sensor 2 - Root Zone Temperature

This sensor shares the same ESP32 as the moisture probe, so it adds no hardware beyond the part itself.

| | **Option A** | **Option B** | **Option C** |
|---|---|---|---|
| **Product** | Adafruit Waterproof DS18B20 Probe | DFRobot Waterproof DS18B20 (DFR0198) | Seeed Grove DHT20 Temp + Humidity |
| **Vendor** | Adafruit | DFRobot | Seeed Studio |
| **Price** | $9.95 | $9-12 via distributors | $4-6 |
| **Link** | [adafruit.com/product/381](https://www.adafruit.com/product/381) | [dfrobot.com](https://www.dfrobot.com) | [seeedstudio.com](https://seeedstudio.com) |
| **How it reaches HA** | 1-Wire to a single ESP32 GPIO; ESPHome `one_wire` bus with a `dallas_temp` sensor; then a HA entity | Same chip, same path | I2C to the same ESP32 |
| **Accuracy** | +/- 0.5 C over the range that matters here | Same chip, same spec | +/- 0.3 C, and adds humidity |
| **21-day reliability** | Strong; sealed stainless probe rated for continuous burial and immersion. Adafruit screens for counterfeit chips, which are common on this part | Same chip; less screening | Not waterproof; will not survive at substrate level near an irrigation line |
| **Total cost** | No added cost; shares the ESP32 | No added cost | No added cost |
| **Availability** | Out of stock at Adafruit; orderable through the DigiKey link on the product page | In stock via RS and Mouser | In stock |

**Selected: DS18B20 waterproof probe - Non-Native (Option A)**

The decision here is placement, not accuracy. What matters is that the probe sits in the substrate next to the moisture sensor, because root zone temperature is what drives evaporation from the medium; ambient air temperature at the top of a rack can be several degrees off. Only the sealed probe formats would thrive in the environment. Therefore, The DHT20 is cheaper and adds humidity, but it would have to be mounted away from the water, which measures the wrong thing.

Between the two DS18B20 options I chose the Adafruit part specifically because counterfeit DS18B20 chips are widespread, resultin in inaccurate behaviours at the edges. While, Adafruit particularly screens for this. Meaning a few dollars extra would be worthwhile if we plan to run it unattended for periods of time. 

---

## Actuator - Irrigation Pump Relay

The relay is the smart switch. Home Assistant controls the relay; the relay switches mains power to the pump.

| | **Option A** | **Option B** | **Option C** |
|---|---|---|---|
| **Product** | Shelly 1PM Gen3 | Sonoff BASIC R4 | Sonoff MINIR4M (Matter) |
| **Vendor** | Shelly | Sonoff | Sonoff |
| **Price** | $19.24 | $7-10 | $12-15 |
| **Link** | [us.shelly.com/products/shelly-1pm-gen3](https://us.shelly.com/products/shelly-1pm-gen3) | [itead.cc/product/sonoff-basicr4-wi-fi-smart-switch](https://itead.cc/product/sonoff-basicr4-wi-fi-smart-switch/) | [sonoff.tech](https://sonoff.tech) |
| **How it reaches HA** | WiFi; built-in Shelly integration; runs entirely on the local network | WiFi; routes through the vendor cloud unless reflashed with Tasmota | WiFi and Matter; local |
| **Max load** | 16A | 10A | 10A |
| **Power monitoring** | Yes | No | No |
| **21-day reliability** | Strong; no internet dependency, overload and thermal protection, 3-year warranty | Cloud dependency is a real outage risk for unattended operation | Local, but a newer product with less field history |
| **Availability** | In stock | In stock | In stock |

**Selected: Shelly 1PM Gen3 (Option A)**

The power meter is the reason, and it proves essential. Without it, "pump on" means only that HA sent a command. With it, HA can confirm the pump actually drew current, which is a significant reason to use the shelly 1pm gen3. A pump failing silently; is realistically a worst case scenario in the design, because nothing else in the system would notice. 

The Sonoff R4 is half the price but routes through a vendor cloud by default, making it local means reflashing it, which adds a failure point and voids the simple story. The MINIR4M is local and cheaper, but has no current sensing, so I would be trading away the failure detection to save seven dollars on a rack running unattended.

---

## Total cost

| Part | Product | Price |
|---|---|---|
| Moisture sensor | DFRobot SEN0193 | $5.90 |
| Temperature sensor | DS18B20 waterproof probe | $9.95 |
| Bridge | Generic ESP32 dev board | ~$6.00 |
| Pump relay | Shelly 1PM Gen3 | $19.24 |
| **Total** | | **~$41** |

---

## Weakest buy, and when I would build instead

**Weakest component: the SEN0193 moisture sensor.**

It reports a relative value, not a real one. It can tell me the soil got wetter or drier than it was; it cannot tell me the substrate is at 22% volumetric water content. The reading also shifts with substrate type, insertion depth, and to a lesser degree temperature. For a binary "is it dry enough to water" trigger with a wide deadband, that is acceptable. It is not acceptable for anything quantitative.

**The threshold:** I would stop buying and start building the moment a decision downstream needs a number rather than a direction. Consider two triggers, nutrient dosing calculated from water content, or comparing moisture behaviour across racks running different substrates. Either one makes an uncalibrated relative reading actively misleading rather than merely imprecise.

**The build fallback:** If I needed a real moisture percentage instead of a relative reading, I would keep the same ESP32 and sensor pins. I would just add a calibration step in ESPHome: measure the voltage in dry air, then in fully soaked soil, for each type of substrate used. That gives a real 0-100% water content number instead of a rough estimate. That ongoing cost is why I would not do this unless the moisture reading actually needed to be precise, not just directional.
