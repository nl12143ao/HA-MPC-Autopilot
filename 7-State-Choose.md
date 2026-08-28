first-match-wins: 

Veiligheid (1 & 2) staan bovenaan, _
_want als de accu leeg is of onder de reserve duikt, is de stroomprijs of de zonvoorspelling volstrekt irrelevant._

Prijssturing & Pieken (3 & 4) komen direct daarna. 
__Een naderende piek (HOLD) of een reeds dure stroomprijs (DISCHARGE) dicteert direct wat je met je 
energievoorraad wilt doen voordat je goedkope uren gaat toekennen_

Laden op prijs of solar (5) zit pas lager. 
_Hier kun je de forecast slim in verweven: als de API aangeeft dat er bakken zonne-energie aankomt (solar forecast > X) 
én je accu is nog niet vol, dwing je hiermee de CHARGE-staat af, tenzij de hogere prijsregels al getriggerd zijn
_
Eindstaten (6 & 7) vangen alles af wat overblijft: 
_als de accu vol is of er is simpelweg geen actie vereist_ 

action:
  - choose:
      ## 1. EMERGENCY (Laagste SoC-grens, bijv < 10% of kritiek) -> Veiligheid gaat voor alles
      - conditions:
          - condition: template
            value_template: "{{ states('sensor.battery_soc') | float < 10 }}"
        sequence:
          - service: script.set_state_EMERGENCY

      ## 2. FLOOR (Onder de reserve < 25%) -> Geen ontlading meer toestaan, net dekt de load
      - conditions:
          - condition: template
            value_template: "{{ states('sensor.battery_soc') | float < 20 }}"
        sequence:
          - service: script.set_state_FLOOR 

      ## 3. HOLD (Piek komt eraan / duur uur nabij) -> Voorkom dat de batterij leegloopt vóór de dure uren
      - conditions:
          - condition: template
            value_template: "{{ states('sensor.peak_coming_soon') | default(false, true) }}"
        sequence:
          - service: script.set_state_HOLD

      ## 4. DISCHARGE (Huidige prijs is hoog > ~20 ct) -> Ontladen tijdens de piek
      - conditions:
          - condition: template
            value_template: "{{ states('sensor.current_energy_price') | float > 0.20 }}"
        sequence:
          - service: script.set_state_DISCHARGE

      ## 5. CHARGE (Goedkoop uur OF overschot van solar forecast) -> Laden als het goedkoop is of de zon volop schijnt
      - conditions:
          - condition: template
            value_template: >
              {{ is_state('sensor.cheap_hours', 'on') 
                 or (states('sensor.solar_forecast_next_hour') | float > 2000 and states('sensor.battery_soc') | float < 90) }}
        sequence:
          - service: script.set_state_CHARGE

      ## 6. FULL (Accu zit zo goed als vol ~95%+) -> Geen laadacties meer nodig
      - conditions:
          - condition: template
            value_template: "{{ states('sensor.battery_soc') | float >= 95 }}"
        sequence:
          - service: script.set_state_FULL

    default:
      ## 7. IDLE / SLEEP (Niks te doen, 's nachts of stabiele situatie)
      - service: script.set_state_IDLE 
   
    
