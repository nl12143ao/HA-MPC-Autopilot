```mermaid

flowchart TB
    start[Solar or Battery] --> forecast[Get forecast Solar]
    start --> prices[Get today Prices]

    forecast --> decide{Decide <BR> strategy }
    prices --> decide{Decide <BR> strategy }

    decide -->|Bad| charge_grid[Charge fa:fa-battery-empty <BR> in Morning <BR> by fa:fa-network-wired grid]
    decide -->|Fair| charge_solar_m[Charge fa:fa-battery-half <BR> in Morning <BR> by fa:fa-solar-panel solar]
    decide -->|Good| export[fa:fa-bolt Export to Grid <BR> in morning]
    export --> charge_solar_mid[Charge fa:fa-battery-full <BR> Midday <BR> by fa:fa-solar-panel solar]

    charge_grid --> next_check{Midday reached?}
    charge_solar_m --> next_check
    charge_solar_mid --> next_check

    next_check --> next_forecast[Get solar forecast <BR> for tomorrow]
    next_forecast --> final_decide{Decide discharge}

    final_decide -->|Good| discharge_30["Discharge to 30%"]
    final_decide -->|Fair| discharge_50["Discharge to 50%"]
    final_decide -->|Bad| discharge_60["Discharge to 60%"]
