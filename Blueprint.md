
https://community.home-assistant.io/t/home-energy-autopilot-charge-the-battery-when-power-is-cheap-run-the-house-off-it-when-its-expensive/1018059 

Home Energy Autopilot — charge the battery when power is cheap, run the house off it when it’s expensive
Blueprints Exchange
post by CihanDE on Jul 20

Been running this at home for a month and finally cleaned it up into a blueprint.

What it does: 
it’s the decision layer. It watches your dynamic price sensor and decides CHARGE / DISCHARGE / IDLE. 
You wire your own battery’s control into three action inputs, so it works with any battery — Anker Solix, Marstek, Victron,
Sonnen, whatever you can already control from HA. The brain decides, your actions execute.

I split it that way because every battery integration is different, but the logic is always the same. 
This way nobody has to fork it for their hardware.

Inputs: <BR> 
- [ ] Price sensor (Nord Pool, Tibber, aWATTar, EPEX, …)
- [ ] Cheap threshold — charge at or below this price
- [ ] Expensive threshold — discharge at or above
- [ ] Battery SoC sensor (optional — leave empty to skip the guards)
- [ ] Stop charging above X % (default 95)
- [ ] Keep a reserve, don’t discharge below X % (default 20)
- [ ] Charge / discharge / idle actions
- [ ] The gap between the two thresholds is a neutral band on purpose (prevents flapping).
- [ ] Triggers on price change and every 5 minutes.

Honest caveat: this only makes sense on an hourly/dynamic tariff. On a flat rate there’s nothing to arbitrage and the battery won’t pay for itself. 
Solar helps but isn’t required.

Real numbers from my own setup (Germany, balcony solar + one small battery):
15.67 EUR last week, about 2.50 a day. Not spectacular, but it’s fully hands-off. 

Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.

https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=[[https%3A%2F%2Fcihanoezkaya.de%2Fblueprint%2Fenergy_autopilot.yaml]

Note: Not on GitHub https://cihanoezkaya.de/blueprint/energy_autopilot.yaml

Feedback on the logic welcome — especially from anyone running a battery I haven’t tested against.

post by CihanDE on Jul 25

Since a few of you have imported it: the part that trips people up is the three action inputs, so here’s a concrete example for a battery exposed as a number entity.

Charge action:

- action: number.set_value  target: { entity_id: number.battery_charge_power }  data: { value: 800 }

Discharge action: same call, but the discharge power entity.
Idle action: set both to 0, or whatever your battery calls “self-use”.

If you want the reasoning behind the thresholds and the numbers I ended up with, I wrote up the whole build here: The Home Energy Autopilot — cut your power bill with Home Assistant





