
| Input / Parameter | Description / Function | Default / Type |
| :--- | :--- | :--- |
| **Price sensor**<br>`price_sensor` | A sensor whose state is the current electricity price (e.g. in currency/kWh). | Entity (`domain: sensor`) |
| **Cheap threshold**<br>`cheap_threshold` | When the current price is at or below this value, the battery charges. | `0.20` (Number, step 0.01) |
| **Expensive threshold**<br>`expensive_threshold` | When the current price is at or above this value, run the house off the battery. | `0.30` (Number, step 0.01) |
| | | |
| **Battery SoC sensor**<br>`soc_sensor` | Optional state-of-charge sensor. Leave empty to skip the SoC guards. | Entity (Optional) |
| **Max charging SoC**<br>`soc_stop_charging` | Stop charging above this percentage to protect or limit battery capacity. | `95` % |
| **Min discharge reserve**<br>`soc_keep_reserve` | Keep a reserve; don't discharge below this percentage. | `20` % |
| | | |
| **Charge action**<br>`action_charge` | Action/script executed when the system decides to charge the battery. | Action block |
| **Discharge action**<br>`action_discharge` | Action/script executed when the system decides to run the house off the battery. | Action block |
| **Idle action**<br>`action_idle` | Action/script executed when no charging or discharging is required. | Action block |


sensor.nord_pool_nl_next_price
0.05
0.20

number.battery_sb31_soc_min_05_20
95
12

service: select.select_option
target:
  entity_id: select.battery_sb31_usage_mode
data:
  option: Your_Mode_Here

select.battery_sb31_ac_input_limit

