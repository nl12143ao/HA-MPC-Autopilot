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

**Spaghetti automation**
I soon found out that the initial automations in HA quickly end up in a spagethi of triggers, conditions and actions. 
It was rapidly a mix of triggers based on price, solar, SOC, trim solar for negative prices, ...
Suddenly the solar was trimmed down unnecessary to avoid export, but .. by what, why ? 
How to keep track of triggers based on price ahead, solar forecast, net export at low price ?

**State machine to the resque**
Back at school we discovered the same for automations using PLC. Filling a water tank using a sensor and a valve is nice. 
Adding a PID control makes it even better and all stays structured. But brewing beer as a complete proces is impossible like this.
What we need is a pre-defined set of machine states. A single automation that controls only these main states. Some states may require a sub-state. 
And in every state a local control is possible. The amount of states is a compromise between fine grained and complexity. Call it KISS. 
An example is a car: most basic is states are STOP, IDLE and RUN. In each state some controls are active, but keep these out of the model! 
Do not inlude the gear in the model, unless you really have to. Or all the driver actions when in RUN. 

**Desk research**
Doing the intial desk research i found following article excactly saying this. 

**Basic STATES**
I came up with various way to models the machine states. Based on grid activityn based on battery activity. I settled for battery based 

| Grid based | Battery based | MIXED | Reference |
| :--- | :--- | :--- | :--- |
| **Balance** | | **BALANCE-Grid/Load** | Net-ZERO, IDLE, SLEEP |
| | | **CHARGE-Solar** | CHARGE |
| **Import** | | | |
| * Import for loads | | **IMPORT-Load** | -- |
| * Import for battery | | **CHARGE-Grid** | EMERGENCY, FLOOR |
| **Export** | | | |
| * Export-Solar | | **EXPORT-Solar** | -- |
| * Export-Battery | | **EXPORT-Battery** | DISCHARGE, FULL |



Grid based                Battery based       MIXED                        Reference              Typical 

Balanced                  Balanced            BALANCE-Grid                 IDLE, SLEEP            Evening, night, steady, zero-grid
                          Charge on solar     CHARGE-Solar                 CHARGE                 Morning, P low, SOC 20, F good 
                                              EXPORT-Solar-Trim                                   Midday, P low, S high, trim solar     
Import                                                    
 Import for loads         Discharge for load  IMPORT-Load                  --                     Midday, P low, S low, washing machine              
 Import for battery       Charge on grid      CHARGE-Grid                  EMERGENCY, FLOOR       Midday, P low, S low, F bad 
Export 
 Export-Solar                                 EXPORT-Solar                 Reverse of HOLD        Morning, P high, SOC 40-12, F good 
 Export-Battery           Discharge to grid   EXPORT-Battery               DISCHARGE, FULL        Evening, SOC 80, P high,   
### Legend

| Parameter | Levels / Values | Unit |
| :--- | :--- | :--- |
| **Fc** (Forecast solar) | Good, Fair, Low, Bad | kWh/day | <BR>
| **Pr** (Price) | High (100), Even (50), Low (5), NEG (0) | euro/MWh | <BR>
| **Sol** (Solar) | Low (20), Medium (50), High (80) | % | <BR>
| **SOC** (SOC) | Bottom (5), Low (20), Good (40), Medium (60), High (80) | % | <BR>

Notes: 
Night needs about SOC of 35% to cover baseloads, fridge, routers, lights until 08:00
EXPORT-Battery allows for discharge to grid in evening to 35%, if price high and SOC high. 
Morning needs about SOC of 12% to cover baseloads, and some water heaters until 10:00
EXPORT-Solar allows for a further discharge to grid, until 12%, to grid; If P high, Forecast good.  


Legend: 
F = Forecast solar Good, Fair, Low, Bad                kWh/day 
P = Price High 100, Even 50, Low 5, NEG 0              euro/MWh 
S = SOC Bottom 5, Low 20, Good 40, Medium 60, High 80  %

                                                          
--------------------------------------------
**Reference** 
https://community.home-assistant.io/t/home-energy-autopilot-a-7-state-machine-that-buys-power-on-the-cheap-hours-architecture-real-numbers-lessons/1016310

*Home Energy Autopilot: a 7-state machine that buys power on the cheap hours (architecture, real numbers, lessons)*
post by CihanDE on Jul 6

After years of just paying the electricity bill, I built a system where Home Assistant does the shopping: 
it buys energy when it is cheap, stores it, and spends it when the grid is expensive. 
Sharing the architecture here because most of the pieces are standard HA — the value is in how they are wired together.

Measured result: a typical month went from ~€180 to €113. Averaged over the year we save €50–75/month; 
battery arbitrage alone contributes €0.60–1.00/day. 
The principles apply anywhere day-ahead prices exist in machine-readable form.)

The four layers
**Hardware** (on site): solar array, home battery (Anker Solix), smart meter, and the household loads. 
_The battery is zero-export: surplus goes to storage, nothing is gifted to the grid — self-consumed kWh are worth 3–4× the feed-in rate here.
_**Data**: hourly exchange prices (day-ahead), weather forecast, a learned consumption profile per weekday, and live system status (SOC, solar W, grid W).
**Brain**: Home Assistant + a few small helper scripts in Docker on a small VPS (€4.49/month — works for me only because the battery is cloud-connected anyway; 
purely local hardware belongs on a local machine).
**Actions**: steer the battery mode, shift loads (washing machine gets slot suggestions via Telegram, family taps one), buy from the grid only in planned cheap hours.

**The 7-state machine** (every 5 minutes, one decision)

# State Machine Strategy: 7-State Energy Management System

**The 7-state machine** (every 5 minutes, one decision)

| State | When | Description / Action |
| :--- | :--- | :--- |
| **EMERGENCY** | Battery critically empty | Charge immediately from grid to protect battery health regardless of price. |
| **FLOOR** | Below reserve (25%) | Hold charge; grid covers full household load to avoid deep discharge. |
| **CHARGE** | Cheap-hours plan | Current hour falls within dynamic cheap-hours window; grid charges battery. |
| **FULL** | Target reached (~95%) | Stop active charging; prevent overcharging and degradation. |
| **DISCHARGE** | Expensive peak hour (>~38 ct) | Export / supply home load from battery during high-tariff periods. |
| **HOLD** | Peak coming soon | Reserve/preserve stored energy; don't waste charge on standard loads before peak. |
| **IDLE/SLEEP** | Nothing to do | Default state when no active conditions are met (especially at night). |

Hysteresis everywhere (in at 25% / out at 55%), otherwise it flaps. "Sleep" only discharges at night if price, remaining charge AND a cheap recharge window all line up.

The price pipeline
Fetch prices hourly (one JSON file as single source of truth) → compute tomorrow's plan in the afternoon (cheapest hours covering expected net demand = learned consumption minus expected solar) → replan every 30 minutes with a 48h view → the state machine executes on the 5-minute beat.

What cost me the most time (honest lessons)
Cloud latency & hangs: the battery ignores a mode change now and then, sensors freeze with identical timestamps. Fix: a properly rated smart plug in front of the battery — 90s off/on forces a real reboot. My self-healing triggers it automatically when needed.
Freshness checks before every action. A frozen sensor plus an eager automation is how you buy expensive power at 7 pm.
Quality gates over features: every 15 min a checklist (prices loaded? plan computed? charged in the cheap window? SOC ready before peak?) rolls up into one health percentage on the dashboard. Most "failures" were bugs in my own checks — verify first, then repair.
Only use officially supported battery modes. Undocumented ones hung the system for hours.
Happy to answer questions about any layer — especially the state machine and the charging-plan logic. I also wrote the whole build up as a beginner-friendly field report (ebook) since friends kept asking; happy to point to it if that's within the forum rules.
