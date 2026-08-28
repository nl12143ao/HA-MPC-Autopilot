**Introduction**
This repository is for my notes to create an auto-pilot to control te setpoints of the devices in my EMS system. 
It will make use of already existing predictive models for the solar production today, and tomorrow. And day-ahead prices. 
It will make use of a state machine to avoid a spaghetti of automations. 

After testing my Anker SolarBank 3, E2700 for a week I realised that even in AI mode all my solar energy was used to 
fill up the batterij in the morning, sell my solar for free in the afternoon, and use only 30 % during evening, and night. 

Starting to play around the various settings in the app I found out that it could be made far more economical. 
But using AI or SELF consumption mode (net zero with P1) does not allow for export. For this, one needs to change 
to DYNAMIC or CUSTOM mode. But when export is finished these modes start using the grid instead using my full battery?
Tried again the AI/SMART mode and it behaves better since recent updates but for me it is a too black box. 
Does it allow to export solar in the mroning instead already charging a 50% full battery ? 
When will it export or import, till how low or hign state of charge (SOC) ? 

**Need self control**
Looking around I found advanced control systems like zero-grid, solar-PID, HASAAS, but these control everything. 
And not in the way 
I only need to provide long term (hours, minutes) setpoints for devives: solar, battery, e-boiler. 
These smart devices control the short term (seconds) stuff themselves. 

So my idea is to setup a setpoint controller. Soon found out that it it needs some kind of forecast for the solar. 
A forecast for the load is nice, but for now of secondairy importance. 

**State machine**
I soon found out that automations in HA quickly end up in a spagethi of triggers, conditions and actions. 
Suddenly the solar is trimmed down to avoid export, but by what flow, why? 
How to keep track of triggers based on price, solar forecast, 


--------------------------------------------
**Reference** 
https://community.home-assistant.io/t/home-energy-autopilot-a-7-state-machine-that-buys-power-on-the-cheap-hours-architecture-real-numbers-lessons/1016310

Home Energy Autopilot: a 7-state machine that buys power on the cheap hours (architecture, real numbers, lessons)
post by CihanDE on Jul 6

After years of just paying the electricity bill, I built a system where Home Assistant does the shopping: 
it buys energy when it is cheap, stores it, and spends it when the grid is expensive. 
Sharing the architecture here because most of the pieces are standard HA — the value is in how they are wired together.

Measured result: a typical month went from ~€180 to €113. Averaged over the year we save €50–75/month; battery arbitrage alone contributes €0.60–1.00/day. (I live in Germany — hourly dynamic tariffs are widely available across Europe. The principles apply anywhere day-ahead prices exist in machine-readable form.)

The four layers
Hardware (on site): solar array, home battery (Anker Solix), smart meter, and the household loads. The battery is zero-export: surplus goes to storage, nothing is gifted to the grid — self-consumed kWh are worth 3–4× the feed-in rate here.
Data: hourly exchange prices (day-ahead), weather forecast, a learned consumption profile per weekday, and live system status (SOC, solar W, grid W).
Brain: Home Assistant + a few small helper scripts in Docker on a small VPS (€4.49/month — works for me only because the battery is cloud-connected anyway; purely local hardware belongs on a local machine).
Action: steer the battery mode, shift loads (washing machine gets slot suggestions via Telegram, family taps one), buy from the grid only in planned cheap hours.
The 7-state machine (every 5 minutes, one decision)
State	When
EMERGENCY	battery critically empty → charge now
FLOOR	below reserve (25%) → hold, grid covers house
CHARGE	current hour is on the cheap-hours plan
FULL	charge target (~95%) reached
DISCHARGE	expensive peak hour (>~38 ct)
HOLD	peak coming soon → don't waste the charge
IDLE/SLEEP	nothing to do → especially at night
Hysteresis everywhere (in at 25% / out at 55%), otherwise it flaps. "Sleep" only discharges at night if price, remaining charge AND a cheap recharge window all line up.

The price pipeline
Fetch prices hourly (one JSON file as single source of truth) → compute tomorrow's plan in the afternoon (cheapest hours covering expected net demand = learned consumption minus expected solar) → replan every 30 minutes with a 48h view → the state machine executes on the 5-minute beat.

What cost me the most time (honest lessons)
Cloud latency & hangs: the battery ignores a mode change now and then, sensors freeze with identical timestamps. Fix: a properly rated smart plug in front of the battery — 90s off/on forces a real reboot. My self-healing triggers it automatically when needed.
Freshness checks before every action. A frozen sensor plus an eager automation is how you buy expensive power at 7 pm.
Quality gates over features: every 15 min a checklist (prices loaded? plan computed? charged in the cheap window? SOC ready before peak?) rolls up into one health percentage on the dashboard. Most "failures" were bugs in my own checks — verify first, then repair.
Only use officially supported battery modes. Undocumented ones hung the system for hours.
Happy to answer questions about any layer — especially the state machine and the charging-plan logic. I also wrote the whole build up as a beginner-friendly field report (ebook) since friends kept asking; happy to point to it if that's within the forum rules.
