# Hybrid Infrastructure Homelab Project

![Project Status](https://img.shields.io/badge/status-active-success.svg)
![Environment](https://img.shields.io/badge/environment-hybrid%20cloud-blue.svg)

Profesjonalne laboratorium inżynierskie oparte na architekturze hybrydowej, łączące zasoby on-premise (Raspberry Pi 5) z chmurą publiczną (Azure, GCP). Projekt służy do symulacji, wdrażania i testowania rozwiązań klasy enterprise w skalowalnym i bezpiecznym środowisku.

## 👨‍💻 O mnie

Jestem studentem ostatniego semestru Informatyki i Młodszym Administratorem Systemów. Moja pasja to budowanie bezpiecznej i skalowalnej infrastruktury IT. Ten projekt jest moim poligonem doświadczalnym, gdzie teorię zamieniam na praktyczne wdrożenia rozwiązań, z którymi stykam się w środowiskach komercyjnych.

---

## 🏗️ Architektura i Komunikacja

Infrastruktura działa w modelu hybrydowym. Centralny punkt zarządzania i przechowywania danych znajduje się on-premise (RPi5), natomiast usługi wystawione na świat (frontend) lub pomocnicze zostały wyniesione do chmury publicznej w celu zwiększenia bezpieczeństwa i separacji ruchu.

**Kluczowe elementy komunikacji:**

* **Tailscale Mesh VPN:** Wszystkie węzły (RPi, Azure VM, GCP VM) są połączone prywatną, szyfrowaną siecią mesh. Umożliwia to bezpieczną komunikację międzyusługową (np. agent Zabbixa z Azure do serwera na RPi) bez wystawiania portów do publicznego internetu.
* **Secure Gateway (Azure):** Maszyna wirtualna Azure pełni rolę bezpiecznej bramy wejściowej. Działa tam Nginx Proxy Manager, który przyjmuje ruch publiczny (HTTP/HTTPS) i bezpiecznym tunelem Tailscale przekierowuje go do usług wewnętrznych na RPi (np. Passbolt).

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
```

### 🛡️ Omówienie Architektury Hybrydowej

Zaprojektowana przeze mnie infrastruktura opiera się na zasadzie Zero-Trust i ścisłej separacji usług:

* **Zarządzanie Flotą Kontenerów:** Serce systemu stanowi `Portainer CE` działający na Raspberry Pi, który poprzez bezpieczne połączenie (port 9001) zarządza agentami na instancjach chmurowych (Azure, GCP). Umożliwia to wdrażanie stacków na dowolnym węźle z jednego, centralnego panelu.
* **Obserwowalność i Bezpieczeństwo (Observability & Security):** Na każdym węźle w chmurze działają zoptymalizowane agenty przesyłające dane telemetryczne do serwera `Zabbix` (port 10051) oraz logi bezpieczeństwa do menedżera `Wazuh SIEM` (porty 1514/1515). Pozwala to na błyskawiczne reagowanie na anomalie.
* **Bezpieczna Komunikacja (Mesh VPN):** Żadna usługa wewnętrzna (w tym bazy danych czy panele zarządzania) nie jest wystawiona bezpośrednio do publicznego internetu. Cały ruch między chmurami a serwerownią domową odbywa się przez szyfrowaną sieć `Tailscale`.
* **Skalowalny Storage:** Zamiast obciążać ograniczony dysk lokalny lub tworzyć drogie dyski maszyn wirtualnych, podpiąłem zewnętrzny udział plikowy Azure File Share (SMB, port 445), który służy węzłom jako wspólne repozytorium na backupy (np. zrzuty bazy Passbolta).
