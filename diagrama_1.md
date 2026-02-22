```
flowchart TD
    subgraph WAN ["Entradas de Internet (WAN)"]
        PON["Fiberhome PON (Fibra 500Mbps)"]
        A15["Galaxy A15 5G (Failover USB)"]
    end

    subgraph Servidor ["Servidor Proxmox (Roteador + NAS)"]
        direction TB
        Onboard["Porta Gigabit Nativa (Entrada Fibra)"]
        USB_Failover["Porta USB (Entrada A15)"]
        
        CPU["Intel Core i5-8500T + 8GB RAM Kingston"]
        Armazenamento["NVMe 128GB (Sistema) + 2x SSDs 2TB (NAS)"]
        
        PCIe_LAN["Placa de Rede PCIe Gigabit (Saída LAN)"]

        Onboard --- CPU
        USB_Failover --- CPU
        CPU --- Armazenamento
        CPU --- PCIe_LAN
    end

    PON ---|"Cabo de Rede"| Onboard
    A15 ---|"Cabo USB (Tethering)"| USB_Failover

    subgraph Rede_Local ["Distribuição Mesh (O Formato em T)"]
        Giga["Twibi Giga+ (Nó Central)"]
        SW["Switch TP-Link 5 Portas"]
        
        Twibi1["Twibi Force 1 (Ponta Esquerda)"]
        Twibi2["Twibi Force 2 (Ponta Direita)"]
        
        Hub["Hub Zigbee"]
        DVR["DVR Intelbras"]
        
        CNC["Máquina LinuxCNC"]
    end
    
    Cams(("Câmeras Analógicas"))

    PCIe_LAN ---|"Cabo de Rede (Link Principal)"| Giga
    Giga ---|"Cabo de Rede"| SW
    
    SW ---|"Cabo de Rede (Backhaul)"| Twibi1
    SW ---|"Cabo de Rede (Backhaul)"| Twibi2
    SW ---|"Cabo de Rede"| Hub
    SW ---|"Cabo de Rede"| DVR
    
    DVR -.-|"Cabo Coaxial"| Cams
    Twibi1 ---|"Cabo de Rede (Porta LAN Livre)"| CNC
    ```