# Energy Balance Model (Mixed-Integer Linear Programming)

In a Model Predictive Control (MPC) framework using a **mixed model**, the core mathematical constraint is the node power balance at each time step $t$. 
This ensures that energy is conserved across all sources, loads, and storage components.

## Power Balance Equation

At any given time step $t$, the net power balance equation is formulated as:

Power Balance Equation:
P_grid(t) + P_solar(t) - P_load(t) - P_batt_charge(t) + P_batt_discharge(t) = 0

Where:
- P_grid(t): Net power imported from (+) or exported to (-) the grid
- P_solar(t): Solar PV generation
- P_load(t): Household energy consumption
- P_batt_charge(t): Power flowing into the battery (>= 0)
- P_batt_discharge(t): Power flowing out of the battery (>= 0)

---

## Binary State Constraints (Mixed-Integer Logic)

To prevent the optimizer from simultaneously charging and discharging the battery (which is physically impossible and economically inefficient), binary decision variables are introduced:
delta_charge(t) is in {0, 1}
delta_discharge(t) is in {0, 1}

Subject to the mutual exclusivity constraint:
delta_charge(t) + delta_discharge(t) <= 1

And bounded by the maximum charge/discharge power limits (P_max):
0 <= P_batt_charge(t) <= P_max_charge * delta_charge(t)
0 <= P_batt_discharge(t) <= P_max_discharge * delta_discharge(t)

---

## Battery State of Charge (SoC) Evolution:

SoC(t+1) = SoC(t) + (Δt / C_capacity) * (η_charge * P_batt_charge(t) - (1 / η_discharge) * P_batt_discharge(t)) * 100

Where:
- SoC(t): State of charge as a percentage at time step t
- C_capacity: Total usable energy capacity of the battery (in kWh)
- Δt: Duration of the time step (e.g., 1 hour or 10 minutes)
- η_charge, η_discharge: Charging and discharging efficiencies
