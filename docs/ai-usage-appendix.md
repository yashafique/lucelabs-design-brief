# AI Usage Appendix

## Tools used

| Tool | Used for |
|---|---|
| Claude (claude.ai) | 	Repo structure, milestone planning, ESPHome and Home Assistant YAML, control logic validation |
| Gemini (gemini.com) | Sourcing research, GitHub workflow guidance, bench-test drafting |
| Whimsical (whimsical.com) | Final system diagram for the integration and control design |
|Docker (docker.com)| HA Instance testing platform |

---

## Prompts that mattered

**Prompt 1:**
> *(Walk me through the component section.)*

What it produced and how I used/modified it: This gave me the first draft of the product comparison table for both sensors and the pump relay. I checked every link myself to confirm the products were real and in stock. I also rewrote the reasoning behind each pick in my own words.

---

**Prompt 2:**
> *(How would I be able to test it)*

What it produced and how I used/modified it: I had written a bench-test order in the design doc, but I hadn't actually worked out how to run any of it myself. This question got me a concrete testing two-stage plan, using HA helpers to fake sensor values and test the control logic without any hardware, and a second path for real calibration once I have the actual parts. 
---

**Prompt 3:**
> *(Paste a third real prompt)*

What it produced and how I used/modified it: TBD

---

## Where AI was wrong — and how I caught it

**Case 1: Outdated ESPHome syntax for the DS18B20 temperature sensor**

The first ESPHome YAML Claude generated used `platform: dallas` for the DS18B20. That syntax was removed in ESPHome v2024.6.0. Any ESPHome install from mid-2024 onward will throw a compile error on that config. The correct current syntax requires a `one_wire:` bus block and `platform: dallas_temp` under it.

How I caught it: I cross-checked the generated YAML against the current ESPHome documentation before my final commit. The ESPHome changelog for v2024.6.0 explicitly lists the dallas platform removal as a breaking change.

What I did: rewrote the YAML block with the correct one wire syntax and added a comment in the file explaining the change, so anyone reviewing the config understands why it differs from older examples they might find online.

---

## What I deliberately did NOT delegate to AI

- Verifying every product link and price manually
- The actual buy-vs-build judgment (AI can list tradeoffs, but the call is mine)
- The failure-mode table (requires thinking through real operational context)
- Final system diagram via whimsical.com to create the design
- Ai was used to provide research materials, but I went ahead and read the documentation related to study the functions and use-cases.
