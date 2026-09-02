```mermaid

graph TD
    %% Stijl definities
    classDef startClass fill:#ffe6cc,stroke:#d79b00,stroke-width:2px,color:#000;
    classDef conditionClass fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px,color:#000;
    classDef actionBlue fill:#1ba1e2,stroke:#0066cc,stroke-width:2px,color:#fff;
    classDef actionRed fill:#e51400,stroke:#b20000,stroke-width:2px,color:#fff;

    Start["Energiegebruik thuis<br><b>Scenario: Opwek en thuisbatterij</b>"]:::startClass

    Start --> C3_1{"Te veel<br>opwek?"}:::conditionClass
    
    %% Tak: Te veel opwek = Ja
    C3_1 -- "Ja" --> C3_2{"Terugleverprijs<br>gunstig?"}:::conditionClass
    
    %% Tussen Terugleverprijs gunstig? en Terugleveren aan het net zetten we 'Batterij ontladen zinvol?'
    C3_2 -- "Ja" --> C3_X{"Batterij ontladen<br>zinvol?"}:::conditionClass
    C3_X -- "Ja" --> A3_X["Batterij ontladen"]:::actionBlue
    C3_X -- "Nee" --> A3_1["Terugleveren<br>aan het net"]:::actionBlue

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
    C3_6 -- "Nee" --> A3_7["Gebruik maximaal /<br>Minimaal verbruik"]:::actionBlu
