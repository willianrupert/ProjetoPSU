````
flowchart TD
    %% DEFINIÇÃO DE CORES (Legenda Visual)
    classDef pwrAC fill:#ffcccc,stroke:#cc0000,stroke-width:2px,color:#000
    classDef pwrBAT fill:#ffe6cc,stroke:#ff8800,stroke-width:2px,color:#000
    classDef pwr12V fill:#ffffcc,stroke:#cccc00,stroke-width:2px,color:#000
    classDef pwr5V fill:#ccffcc,stroke:#00aa00,stroke-width:2px,color:#000
    classDef data fill:#e1d5e7,stroke:#9673a6,stroke-width:2px,color:#000

    %% ==========================================
    %% BLOCOS DE AGRUPAMENTO
    %% ==========================================
    
    subgraph G1 ["⚡ REDE 220V (Parede)"]
        Tomada["Tomada 220V"]:::pwrAC
        Carregador["Carregador Lítio 42V 2A"]:::pwrBAT
    end

    subgraph G2 ["🔋 SISTEMA DE BATERIA (42V)"]
        Bat["Bateria Nicoll 36V"]:::pwrBAT
        Buck200["Step-Down 200W (42V -> 12V)"]:::pwrBAT
    end

    subgraph G3 ["🔌 GERAÇÃO E DISTRIBUIÇÃO (12V e 5V)"]
        Fonte150["Fonte Industrial 150W (12V)"]:::pwr12V
        Diodo["Módulo Diodo Ideal Duplo"]:::pwr12V
        Bus["Barramento 6 Furos (12V)"]:::pwr12V
        Buck5V["Step-Down LM2596 (12V -> 5V)"]:::pwr5V
    end

    subgraph G4 ["⚙️ ATUADORES (Chaveamento)"]
        Rele1["Relé 1 (NC) - Fonte Principal"]:::data
        Rele2["Relé 2 (NO) - Carregador"]:::data
        Rele3["Relé 3 (NO) - Corte Bateria"]:::data
        Rele4["Relé 4 (NO) - Monitor"]:::data
        Mosfet["Módulo MOSFET AOD4184"]:::data
    end

    subgraph G5 ["📡 SENSORES (Leituras)"]
        ZMPT["ZMPT101B (Mede Rede AC)"]:::data
        Shunt["Shunt 10A (Mede Corrente DC)"]:::data
        Div["Divisor 100k/10k (Mede Volts DC)"]:::data
        ADS["Módulo ADS1115 (Conversor I2C)"]:::data
        DHT["Sensor DHT22 (Temp/Umid)"]:::data
    end

    subgraph G6 ["🧠 CÉREBRO (Interface e Lógica)"]
        ESP["Tela Guition ESP32-S3 Capacitiva"]:::data
        Botao(("Botão LED Frontal")):::data
    end

    subgraph G7 ["💻 CONSUMIDORES (Cargas)"]
        Pico["PicoPSU 160W"]:::pwr12V
        Servidor["PC Servidor i5-8500T"]:::pwr12V
        DVR["DVR + Câmeras Intelbras"]:::pwr12V
        Twibi["Twibi Fast / Giga"]:::pwr12V
        Monitor["Monitor Samsung"]:::pwr12V
        Fans(("2x Fans 120mm Push-Pull")):::pwr12V
    end

    %% ==========================================
    %% LIGAÇÕES DE POTÊNCIA GROSSA (Linhas Contínuas ==>)
    %% ==========================================
    
    %% Energia da Rua
    Tomada == "Fase/Neutro" ==> ZMPT
    Tomada == "Fase 220V" ==> Rele1
    Tomada == "Fase 220V" ==> Rele2
    Rele1 == "NC (Sempre Ligado)" ==> Fonte150
    Rele2 == "NO (Controlado)" ==> Carregador
    
    %% Energia da Bateria
    Carregador == "Carga 42V" ==> Bat
    Bat == "Fio Grosso Negativo" ==> Shunt
    Bat == "Lê 42V" ==> Div
    Bat == "Fio Grosso Positivo" ==> Rele3
    Rele3 == "NO (Corte Subtensão)" ==> Buck200
    
    %% Encontro das Energias
    Fonte150 == "12V Primário" ==> Diodo
    Buck200 == "12V Secundário" ==> Diodo
    Diodo == "12V Estável e Contínuo" ==> Bus
    
    %% Distribuição a partir do Barramento 12V
    Bus == "12V" ==> Pico
    Pico == "Cabo ATX 24 pin" ==> Servidor
    Bus == "12V" ==> DVR
    Bus == "12V / 9V" ==> Twibi
    Bus == "12V" ==> Buck5V
    
    %% Alimentação das Cargas Controláveis
    Bus == "12V Bruto" ==> Rele4
    Rele4 == "12V Chaveado" ==> Monitor
    Bus == "12V Bruto" ==> Mosfet
    Mosfet == "12V Chaveado PWM" ==> Fans

    %% Distribuição dos 5V (Energia Fina)
    Buck5V == "5V Constante" ==> ESP
    Buck5V == "5V (Pinos VCC)" ==> Rele1
    Buck5V == "5V (Pinos VCC)" ==> Rele2
    Buck5V == "5V (Pinos VCC)" ==> Rele3
    Buck5V == "5V (Pinos VCC)" ==> Rele4
    Buck5V == "5V" ==> ADS

    %% ==========================================
    %% LIGAÇÕES DE SINAIS E DADOS (Linhas Pontilhadas -.->)
    %% ==========================================
    
    %% Sensores Brutos -> ADS1115
    ZMPT -. "Sinal AC Analógico" .-> ADS
    Shunt -. "MiliVolts DC" .-> ADS
    Div -. "Máx 3.8V DC" .-> ADS
    
    %% Comunicação de Dados -> ESP32
    ADS -. "I2C SDA/SCL (Conector P4/P5)" .-> ESP
    DHT -. "Sinal IO6 (Conector P3)" .-> ESP
    Botao -. "Sinal IO15 (Conector P3)" .-> ESP

    %% Comandos ESP32 -> Atuadores
    ESP -. "Sinal IO46 (Conector P2)" .-> Rele1
    ESP -. "Sinal IO9 (Conector P2)" .-> Rele2
    ESP -. "Sinal IO14 (Conector P2)" .-> Rele3
    ESP -. "Sinal IO5 (Conector P2)" .-> Rele4
    ESP -. "Sinal PWM IO7 (Conector P3)" .-> Mosfet

    %% Comunicação Servidor HA
    ESP -. "Cabo USB Data-Only (Fio Vermelho Cortado)" .-> Servidor
    ```