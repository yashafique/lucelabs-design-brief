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

Both sensor readings feed one decision together: 
Hot and dry soil triggers a longer watering cycle. 
Cool and moist soil skips irrigation entirely.


## Contact

Yahya Shafique | yahyashafique05@gmail.com
