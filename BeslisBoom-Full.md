```mermaid
graph TD
    %% Stijl definities
    classDef startClass fill:#ffe6cc,stroke:#d79b00,stroke-width:2px,color:#000;
    classDef scenarioClass fill:#d5e8d4,stroke:#82b366,stroke-width:2px,color:#000;
    classDef conditionClass fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px,color:#000;
    classDef actionBlue fill:#1ba1e2,stroke:#0066cc,stroke-width:2px,color:#fff;
    classDef actionRed fill:#e51400,stroke:#b20000,stroke-width:2px,color:#fff;

    Start["Ieder KWARTIER dient vanaf 1-1-2027 deze afweging te worden gemaakt:<br><b>Energiegebruik thuis</b>"]:::startClass

    %% 4 Hoofdscenario's
    Start --> S1["Geen opwek<br>en geen thuisbatterij"]:::scenarioClass
    Start --> S2["Alleen thuisbatterij"]:::scenarioClass
    Start --> S3["Opwek en thuisbatterij"]:::scenarioClass
    Start --> S4["Opwek"]:::scenarioClass

    %% SCENARIO 1: Geen opwek en geen thuisbatterij
    S1 --> C1_1{"Leverprijs<br>gunstig?"}:::conditionClass
    C1_1 -- "Ja" --> A1_1["Zo weinig mogelijk<br>elektriciteit gebruiken"]:::actionBlue
    C1_1 -- "Nee" --> A1_2["Gebruik maximaal inzetten<br>(Wasmachine aanzetten, accu auto opladen etc!)"]:::actionBlue

    %% SCENARIO 2: Alleen thuisbatterij
    S2 --> C2_1{"Leverprijs<br>gunstig?"}:::conditionClass
    C2_1 -- "Ja" --> C2_2{"Batterij laden<br>zinvol?"}:::conditionClass
    C2_1 -- "Nee" --> C2_3{"Batterij ontladen<br>zinvol?"}:::conditionClass
    
    C2_2 -- "Ja" --> A2_1["Batterij laden"]:::actionBlue
    C2_2 -- "Nee" --> A2_2["Zo weinig mogelijk<br>elektriciteit gebruiken"]:::actionBlue
    C2_3 -- "Ja" --> A2_3["Batterij ontladen"]:::actionBlue
    C2_3 -- "Nee" --> A2_4["Zo weinig mogelijk<br>elektriciteit gebruiken"]:::actionBlue

    %% SCENARIO 3: Opwek en thuisbatterij
    S3 --> C3_1{"Te veel<br>opwek?"}:::conditionClass
    
    %% Tak: Te veel opwek = Ja
    C3_1 -- "Ja" --> C3_2{"Terugleverprijs<br>gunstig?"}:::conditionClass
    C3_2 -- "Ja" --> A3_1["Terugleveren<br>aan het net"]:::actionBlue
    C3_2 -- "Nee" --> C3_3{"Batterij laden<br>zinvol?"}:::conditionClass
    C3_3 -- "Ja" --> A3_2["Batterij laden"]:::actionBlue
    C3_3 -- "Nee" --> A3_3["Omvormer uitzetten"]:::actionRed

    %% Tak: Te veel opwek = Nee
    C3_1 -- "Nee" --> C3_4{"Leverprijs<br>gunstig?"}:::conditionClass
    C3_4 -- "Ja" --> C3_5{"Batterij laden<br>zinvol?"}:::conditionClass
    C3_4 -- "Nee" --> C3_6{"Batterij ontladen<br>zinvol?"}:::conditionClass

    C3_5 -- "Ja" --> A3_4["Batterij laden"]:::actionBlue
    C3_5 -- "Nee" --> A3_5["Zo weinig mogelijk<br>elektriciteit gebruiken"]:::actionBlue
    C3_6 -- "Ja" --> A3_6["Batterij ontladen"]:::actionBlue
    C3_6 -- "Nee" --> A3_7["Gebruik maximaal /<br>Minimaal verbruik"]:::actionBlue

    %% SCENARIO 4: Alleen opwek
    S4 --> C4_1{"Te veel<br>opwek?"}:::conditionClass
    C4_1 -- "Ja" --> C4_2{"Terugleverprijs<br>gunstig?"}:::conditionClass
    C4_1 -- "Nee" --> C4_3{"Leverprijs<br>gunstig?"}:::conditionClass

    C4_2 -- "Ja" --> A4_1["Terugleveren<br>aan het net"]:::actionBlue
    C4_2 -- "Nee" --> A4_2["Omvormer uitzetten"]:::actionRed

    C4_3 -- "Ja" --> A4_3["Zo weinig mogelijk<br>elektriciteit gebruiken"]:::actionBlue
    C4_3 -- "Nee" --> A4_4["Gebruik maximaal inzetten<br>(Wasmachine, auto opladen etc!)"]:::actionBlue
