# Hybrid Infrastructure Homelab Project

![Project Status](https://img.shields.io/badge/status-active-success.svg)
![Environment](https://img.shields.io/badge/environment-hybrid%20cloud-blue.svg)

Profesjonalne laboratorium inżynierskie oparte na architekturze hybrydowej, łączące zasoby on-premise (Raspberry Pi 5) z chmurą publiczną (Azure, GCP). Projekt służy do symulacji, wdrażania i testowania rozwiązań klasy enterprise w skalowalnym i bezpiecznym środowisku.

## 👨‍💻 O mnie

Jestem studentem ostatniego semestru Informatyki i pracującym Junior System Administratorem. Moja pasja to budowanie bezpiecznej i zautomatyzowanej infrastruktury IT. Ten projekt to mój poligon doświadczalny, gdzie wiedzę akademicką zamieniam na praktyczne wdrożenia technologii używanych w nowoczesnych środowiskach komercyjnych.

## 🛠️ Tech Stack

### Infrastruktura & Chmura
![Raspberry Pi](https://img.shields.io/badge/-RaspberryPi-C51A4A?style=for-the-badge&logo=Raspberry-Pi)
![Microsoft Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Google Cloud](https://img.shields.io/badge/GoogleCloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white)

### Wirtualizacja & Orkiestracja
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Portainer](https://img.shields.io/badge/Portainer-13BEF9?style=for-the-badge&logo=Portainer&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

### Sieci & Bezpieczeństwo
![Tailscale](https://img.shields.io/badge/Tailscale-FFFFFF?style=for-the-badge&logo=Tailscale&logoColor=black)
![Nginx Proxy Manager](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Wazuh](https://img.shields.io/badge/Wazuh-00A8E6?style=for-the-badge&logo=Wazuh&logoColor=white)
![Passbolt](https://img.shields.io/badge/Passbolt-E22A56?style=for-the-badge&logo=Passbolt&logoColor=white)

### Obserwowalność (Monitoring)
![Zabbix](https://img.shields.io/badge/Zabbix-D40000?style=for-the-badge&logo=Zabbix&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)

---

## 🏗️ Architektura i Komunikacja

Poniższy diagram przedstawia przepływ ruchu sieciowego oraz logiczną separację warstw w modelu hybrydowym.

```mermaid
graph TD
    %% Definicje stylów dla lepszej czytelności
    classDef cloud fill:#e3f2fd,stroke:#1565c0,stroke-width:2px;
    classDef onprem fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef vpn fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,stroke-dasharray: 5 5;
    classDef storage fill:#fff3e0,stroke:#e65100,stroke-width:2px;

    Internet((🌐 Public Internet\npi-passbolt-rc.duckdns.org))

    subgraph Tailscale[🔐 Tailscale Mesh VPN Network]
        direction TB

        subgraph Clients[💻 Client Devices]
            Admin[Admin Laptop / Mobile]
        end

        subgraph Azure[☁️ Azure VM 2Core 1GB RAM]
            NPM[Nginx Proxy Manager\nSecure Gateway]
            AzAgents[Zabbix & Wazuh Agents]
            AzPA[Portainer Agent]
        end

        subgraph GCP[☁️ Google Cloud 2Core 1GB RAM]
            GCPAgents[Zabbix & Wazuh Agents]
            GCPPA[Portainer Agent]
        end

        subgraph Local[🏠 Raspberry Pi 5 8GB RAM]
            Portainer[Portainer Central]
            Passbolt[(Passbolt DB)]
            Zabbix[Zabbix Server]
            Wazuh[Wazuh Manager]
        end
    end

    subgraph AzureStorage[🗄️ Cloud Storage]
        SMB[(Azure File Share 100GB)]
    end

    %% Ruch użytkownika do Passbolta (Zewnętrzny)
    Admin -- "1. DNS Resolution\n2. HTTPS Request" --> Internet
    Internet -->|HTTPS :444| NPM

    %% Ruch wewnętrzny przez Proxy
    NPM -->|Reverse Proxy via VPN :8081| Passbolt

    %% Dostęp administracyjny (Wewnętrzny VPN)
    Admin -.->|HTTPS :9443| Portainer
    Admin -.->|HTTP :8080| Zabbix
    Admin -.->|HTTPS :443| Wazuh

    %% Centralne zarządzanie Portainerem
    Portainer -->|TCP :9001| AzPA
    Portainer -->|TCP :9001| GCPPA

    %% Przepływ logów i telemetrii
    AzAgents -->|TCP :10051| Zabbix
    AzAgents -->|TCP :1514 / :1515| Wazuh
    GCPAgents -->|TCP :10051| Zabbix
    GCPAgents -->|TCP :1514 / :1515| Wazuh

    %% Montowanie dysków sieciowych
    SMB <-->|SMB :445| Local
    SMB <-->|SMB :445| Azure

    %% Aplikacja stylów
    class Azure,GCP cloud;
    class Local onprem;
    class Tailscale vpn;
    class SMB storage;
