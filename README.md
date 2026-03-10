## 🌐 Phase 5: Distributed Infrastructure Inventory & Network Mesh

> "Single Pane of Glass" – zarządzanie rozproszoną infrastrukturą z jednego punktu, zachowując pełną izolację od publicznego internetu.

Kluczowym elementem projektu jest wykorzystanie **Tailscale (Mesh VPN)**. Dzięki temu wszystkie węzły (On-premise i Cloud) komunikują się wewnątrz prywatnej warstwy 100.x.x.x. Porty usług są otwarte wyłącznie na interfejsach VPN, co drastycznie zmniejsza powierzchnię ataku.

### 📋 Inwentarz Węzłów (Nodes)

| Hostname | Lokalizacja | Adres IP (Tailscale) | Rola w systemie |
| :--- | :--- | :--- | :--- |
| **RPi5-Homelab** | Dom (Białystok) | `100.73.36.110` | Central Management, Zabbix Server, Wazuh Manager |
| **Fedora-Node** | Laptop (Local) | `100.72.53.112` | App Node, Podman Runtime, Workstation |
| **Azure-VM** | Chmura (Azure) | `100.x.x.x` | Reverse Proxy (NPM), Cloud Monitoring |
| **GCP-Node** | Chmura (GCP) | `100.x.x.x` | Backup Node, External Health-Check |


### 🔍 Porty i Usługi (Full Service Blueprint)

Poniższa tabela przedstawia pełną inwentaryzację usług działających na węźle centralnym (RPi5). Każda usługa została przypisana do konkretnej warstwy funkcjonalnej.

| Port | Usługa / Proces | Warstwa | Zastosowanie |
| :--- | :--- | :--- | :--- |
| **8080** | Zabbix Web | **Obserwowalność** | Panel zarządzania i wizualizacji metryk. |
| **10051** | Zabbix Server | **Obserwowalność** | Główny silnik zbierający dane od agentów (TCP). |
| **10050** | Zabbix Agent | **Obserwowalność** | Monitorowanie parametrów lokalnych Malinki. |
| **9443** | Portainer (HTTPS) | **Zarządzanie** | Graficzny interfejs do zarządzania flotą kontenerów. |
| **8081** | Passbolt (App) | **Bezpieczeństwo** | System zarządzania hasłami klasy enterprise. |
| **8082** | Filebrowser | **Pliki** | Webowy menedżer plików (dostęp do zasobów labu). |
| **8000** | Wazuh Dashboard | **Bezpieczeństwo** | Panel analityczny zdarzeń bezpieczeństwa (SIEM). |
| **22** | OpenSSH | **Dostęp** | Bezpieczny dostęp terminalowy (Klucze Ed25519). |
| **5900** | WayVNC | **Dostęp** | Zdalny pulpit (VNC) dla środowiska graficznego. |
| **7575** | Bazarr / App | **Media/Inne** | Automatyzacja pobierania napisów (Node.js). |
| **3002** | Custom Dashboard | **Zarządzanie** | Dodatkowy panel pomocniczy dla usług labu. |
| **41641 / UDP** | Tailscale | **Sieć** | Komunikacja wewnątrz prywatnej sieci Mesh VPN. |
| **5353 / UDP** | Avahi | **Sieć** | Rozwiązywanie nazw lokalnych (mDNS/pi.local). |

> **Nota dot. Bezpieczeństwa:** Usługi takie jak CUPS (port 631) są celowo ograniczone do interfejsu `127.0.0.1`, co uniemożliwia dostęp do nich spoza samej Malinki.

---
