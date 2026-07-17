# AI Usage Appendix

## Tools used

| Tool | Used for |
|---|---|
| Claude (claude.ai) | Repo structure setup, milestone planning, ESPHome YAML drafting, VPD formula validation |
| *(add others as used)* | |

---

## Prompts that mattered

**Prompt 1:**
> *(Paste a real prompt you used — e.g. the one that generated your ESPHome YAML)*

What it produced and how I used/modified it: TBD

---

**Prompt 2:**
> *(Paste a second real prompt)*

What it produced and how I used/modified it: TBD

---

**Prompt 3:**
> *(Paste a third real prompt)*

What it produced and how I used/modified it: TBD

---

## Where AI was wrong — and how I caught it

**Case 1: Outdated ESPHome syntax for the DS18B20 temperature sensor**

The first ESPHome YAML Claude generated used `platform: dallas` for the DS18B20. That syntax was removed in ESPHome v2024.6.0. Any ESPHome install from mid-2024 onward will throw a compile error on that config. The correct current syntax requires a `one_wire:` bus block and `platform: dallas_temp` under it.

How I caught it: I cross-checked the generated YAML against the current ESPHome documentation before committing it. The ESPHome changelog for v2024.6.0 explicitly lists the dallas platform removal as a breaking change.

What I did: rewrote the YAML block with the correct one_wire syntax and added a comment in the file explaining the change, so anyone reviewing the config understands why it differs from older examples they might find online.

*(Add more cases here as you work through the rest of the config)*

---

## What I deliberately did NOT delegate to AI

- Verifying every product link and price manually
- The actual buy-vs-build judgment (AI can list tradeoffs, but the call is mine)
- The failure-mode table (requires thinking through real operational context, not generating plausible-sounding scenarios)
- *(add your own)*
