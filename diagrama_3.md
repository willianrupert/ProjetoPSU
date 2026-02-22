````
%%{
  init: {
    "theme": "default",
    "flowchart": {
      "defaultRenderer": "elk",
      "nodeSpacing": 50,
      "rankSpacing": 50
    }
  }
}%%
flowchart TD
    %% ==========================================
    %% DEFINIÇÃO DE CORES E ESTILOS (Legenda Visual)
    %% ==========================================
    classDef pwrAC fill:#ffcccc,stroke:#cc0000,stroke-width:2px,color:#000
    classDef pwrBAT fill:#ffe6cc,stroke:#ff8800,stroke-width:2px,color:#000
    classDef pwr12V fill:#ffffcc,stroke:#cccc00,stroke-width:2px,color:#000
    classDef pwr9V fill:#e6f7ff,stroke:#0066cc,stroke-width:2px,color:#000
    classDef pwr5V fill:#ccffcc,stroke:#00aa00,stroke-width:2px,color:#000
    classDef data fill:#e1d5e7,stroke:#9673a6,stroke-width:2px,color:#000
    classDef network fill:#cce5ff,stroke:#0055ff,stroke-width:2px,color:#000
    classDef external fill:#f2f2f2,stroke:#666666,stroke-width:2px,stroke-dasharray: 5 5,color:#000

    %% ==========================================
    %% ZONA 1: ENTRADAS EXTERNAS E WAN
    %% ==========================================
    subgraph WAN ["🌐 Entradas de Internet (WAN)"]
        PON["Fiberhome PON (Fibra 500Mbps)"]:::network
        A15["Galaxy A15 5G (Failover USB)"]:::network
    end

    subgraph REDE_EXTERNA ["🔌 Alimentação Externa (Tomada Comum)"]
        TomadaExt["Tomada 220V (Fora do Nobreak)"]:::external
        FonteCNC["Fonte LinuxCNC"]:::external
        FonteZigbee["Fonte Hub Zigbee (5V)"]:::external
        FonteTwibi1["Fonte Twibi Force 1 (9V)"]:::external
        
        TomadaExt == "220V" ==> FonteCNC
        TomadaExt == "220V" ==> FonteZigbee
        TomadaExt == "220V" ==> FonteTwibi1
    end

    %% ==========================================
    %% ZONA 2: ENERGIA DO NOBREAK (Geração e Distribuição)
    %% ==========================================
    subgraph PWR_IN ["⚡ Geração de Energia (Nobreak)"]
        TomadaUPS["Tomada 220V (Principal)"]:::pwrAC
        Carregador["Carregador Lítio 42V"]:::pwrBAT
        Bat["Bateria Nicoll 36V"]:::pwrBAT
        Buck200["Step-Down 200W (42V->12V)"]:::pwrBAT
        Fonte150["Fonte Industrial 150W (12V)"]:::pwr12V
        Diodo["Diodo Ideal Duplo"]:::pwr12V
        Bus["Barramento Central (12V)"]:::pwr12V
        
        Buck5V["Step-Down (12V->5V)"]:::pwr5V
        Buck9V["Step-Down (12V->9V)"]:::pwr9V
    end

    %% ==========================================
    %% ZONA 3: CÉREBRO, SENSORES E RELÉS
    %% ==========================================
    subgraph LOGICA ["🧠 Controle, Automação e Sensores"]
        ESP["Tela Guition ESP32-S3"]:::data
        ADS["ADS1115 (Conversor I2C)"]:::data
        ZMPT["ZMPT101B (Mede 220V)"]:::data
        Shunt["Shunt 10A (Mede Corrente)"]:::data
        Div["Divisor de Tensão (Mede 42V)"]:::data
        
        Rele1["Relé 1 (NC) - Fonte Principal"]:::data
        Rele2["Relé 2 (NO) - Carregador"]:::data
        Rele3["Relé 3 (NO) - Corte Bateria"]:::data
        Rele4["Relé 4 (NO) - Monitor"]:::data
        Mosfet["MOSFET PWM - Fans"]:::data
    end

    %% ==========================================
    %% ZONA 4: O SERVIDOR PROXMOX (Detalhado)
    %% ==========================================
    subgraph CORE_TI ["🖥️ Servidor Proxmox (Roteador + NAS)"]
        Pico["PicoPSU 160W"]:::pwr12V
        
        CPU["Core i5-8500T + 8GB RAM"]:::data
        Arm["NVMe 128GB (OS) + 2x SSDs 2TB (NAS)"]:::data
        
        Onboard["Gigabit Nativa (WAN 1)"]:::network
        USB_Net["Porta USB (WAN 2)"]:::network
        PCIe["PCIe Gigabit (LAN Out)"]:::network
        USB_HA["Porta USB (Dados ESP32)"]:::data
    end

    %% ==========================================
    %% ZONA 5: REDES LOCAIS E CARGAS
    %% ==========================================
    subgraph REDES_LOCAIS ["💻 Distribuição Local e Outras Cargas"]
        SW["Switch TP-Link 5 Portas"]:::network
        Giga["Twibi Giga+ (Nó Central)"]:::network
        Twibi2["Twibi Force 2 (Ponta Direita)"]:::network
        
        DVR["DVR Intelbras"]:::network
        Monitor["Monitor Samsung"]:::pwr12V
        Fans(("2x Fans Push-Pull")):::pwr12V
    end

    subgraph EQUIP_EXTERNOS ["🖥️ Equipamentos Externos (Wall Powered)"]
        Hub["Hub Zigbee"]:::network
        Twibi1["Twibi Force 1 (Ponta Esquerda)"]:::network
        CNC["Máquina LinuxCNC"]:::network
    end

    %% ==========================================
    %% ROTEAMENTO DE ENERGIA ELÉTRICA (Linhas ==>)
    %% ==========================================
    TomadaUPS == "Fase 220V" ==> ZMPT
    TomadaUPS == "Fase" ==> Rele1
    TomadaUPS == "Fase" ==> Rele2
    Rele1 == "NC" ==> Fonte150
    Rele2 == "NO" ==> Carregador
    
    Carregador == "42V" ==> Bat
    Bat == "Negativo" ==> Shunt
    Bat == "Positivo" ==> Rele3
    Bat == "42V" ==> Div
    Rele3 == "NO" ==> Buck200
    
    Fonte150 == "12V" ==> Diodo
    Buck200 == "12V" ==> Diodo
    Diodo == "12V Estável" ==> Bus
    
    %% Barramento para Cargas
    Bus == "12V" ==> Buck5V
    Bus == "12V" ==> Buck9V
    Bus == "12V" ==> Pico
    Bus == "12V" ==> DVR
    Bus == "12V" ==> SW
    Bus == "12V" ==> Giga
    
    Bus == "12V" ==> Rele4
    Rele4 == "12V Chaveado" ==> Monitor
    Bus == "12V" ==> Mosfet
    Mosfet == "12V PWM" ==> Fans
    
    %% Alimentação Interna do Servidor
    Pico == "Cabo ATX 24P" ==> CPU
    Pico == "Cabo SATA Power" ==> Arm
    
    %% Energia Fina 5V e 9V
    Buck5V == "5V" ==> ESP
    Buck5V == "5V" ==> ADS
    Buck5V == "5V (Bobinas)" ==> Rele1
    Buck9V == "9V Seguro" ==> Twibi2

    %% Energia Externa (Equipamentos Isolados)
    FonteCNC == "Energia Própria" ==> CNC
    FonteZigbee == "5V" ==> Hub
    FonteTwibi1 == "9V" ==> Twibi1

    %% ==========================================
    %% ROTEAMENTO DE DADOS E LÓGICA INTERNA (Linhas ---)
    %% ==========================================
    %% Comunicação Interna da Placa-Mãe
    CPU --- Arm
    CPU --- Onboard
    CPU --- PCIe
    CPU --- USB_Net
    CPU --- USB_HA

    %% Roteamento de Rede e Internet
    PON --- |"Link Fibra"| Onboard
    A15 --- |"USB Tethering"| USB_Net
    
    PCIe --- |"Cabo LAN (Link Principal)"| Giga
    Giga --- |"Cabo LAN"| SW
    
    SW --- |"Backhaul"| Twibi1
    SW --- |"Backhaul"| Twibi2
    SW --- |"Cabo LAN"| Hub
    SW --- |"Cabo LAN"| DVR
    
    Twibi1 --- |"Cabo LAN"| CNC

    %% ==========================================
    %% SINAIS IOT E AUTOMAÇÃO (Linhas -.->)
    %% ==========================================
    ZMPT -. "Sinal AC" .-> ADS
    Shunt -. "mV DC" .-> ADS
    Div -. "Max 3.8V" .-> ADS
    ADS -. "I2C" .-> ESP
    
    ESP -. "Controla" .-> Rele1
    ESP -. "Controla" .-> Rele2
    ESP -. "Controla" .-> Rele3
    ESP -. "Controla" .-> Rele4
    ESP -. "PWM" .-> Mosfet
    
    %% O elo entre o Cérebro ESP32 e o Servidor
    ESP -. "Cabo USB Data-Only" .-> USB_HA
    ```