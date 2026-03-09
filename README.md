# 🛡️ RPi5-Homelab: Enterprise Security & Monitoring Stack
> **High-availability internal infrastructure project based on Raspberry Pi 5.**

![OS](https://img.shields.io/badge/OS-Debian%20Bookworm-blue?logo=debian)
![Docker](https://img.shields.io/badge/Containerization-Docker-blue?logo=docker)
![Security](https://img.shields.io/badge/Security-Hardened-success?logo=skyliner)
![Architecture](https://img.shields.io/badge/Arch-ARM64-orange)

---

## 🔒 Phase 1: Infrastructure Hardening
> Bezpieczeństwo sesji administracyjnych jest krytycznym elementem projektu. Wdrożone mechanizmy eliminują podatności na ataki typu *Credential Stuffing* i *Brute-force*.

### 🛡️ Key Implementations:
* **Kryptografia Asymetryczna:** Całkowita rezygnacja z haseł na rzecz par kluczy **Ed25519** (krzywe eliptyczne).
* **Identity Verification:** Dostęp do powłoki systemu ograniczony wyłącznie do autoryzowanych hostów (Windows/Fedora) z aktywnym kluczem prywatnym.
* **Root Access Policy:** Implementacja `PermitRootLogin no` – wymuszenie eskalacji uprawnień poprzez `sudo`.
* **SSH Configuration:** Optymalizacja parametru `MaxAuthTries 3` oraz wyłączenie `PasswordAuthentication`.

---

<details>
<summary>🔒 Audit: Weryfikacja działających zabezpieczeń (Screany)</summary>
<br>

Poniższe zrzuty ekranu stanowią dowód namacalny, że zadeklarowane powyżej zabezpieczenia są aktywne i skuteczne. Rekruter/Audytor może zweryfikować stan faktyczny.

***

###  Dowód 1: Wyłączone Uwierzytelnianie Hasłem
Próba połączenia z zewnętrznego hosta (bez unikalnego klucza prywatnego) kończy się natychmiastowym odrzuceniem sesji przez serwer przed prośbą o hasło. Serwer informuje, że akceptuje tylko klucze publiczne.

`<img src="placeholders/ssh-denied-password.png" alt="Dowód wyłączonych haseł - Permission Denied (publickey)" width="700">`

* **Opis obrazu:** Na screenie widać komendę `ssh student@10.64.104.XX` i wynik `student@10.64.104.XX: Permission denied (publickey).`. Nie pojawia się monit o hasło. To dowodzi, że `PasswordAuthentication no` działa.

***

### Dowód 2: Zablokowane Logowanie Direct-Root
Próba bezpośredniego logowania na konto `root` z autoryzowanego hosta również kończy się błędem `Permission denied`, ponieważ polityka `PermitRootLogin no` jest nadrzędna względem obecności klucza w `authorized_keys` dla roota.

`<img src="placeholders/ssh-denied-root.png" alt="Dowód zablokowanego roota - PermitRootLogin no" width="700">`

* **Opis obrazu:** Na screenie widać komendę `ssh root@10.64.104.XX` i wynik `root@10.64.104.XX: Permission denied (publickey).`. Nawet z unikalnym kluczem, bezpośredni dostęp do roota jest odcięty. Logowanie możliwe tylko na konto zwykłego użytkownika z eskalacją `sudo`.

> **Inżynierska wskazówka dla audytora:** Powyższe dowody wykazują wdrożenie strategii **Defense in Depth** w obrębie najwrażliwszego portu administracyjnego serwera.

</details>


---

## 🐳 Docker Stack Overview
Usługi działają w odizolowanych kontenerach, zarządzanych przez **Portainer CE**.

### Monitoring & Security
* **Wazuh (SIEM/XDR):** Centralne zarządzanie zdarzeniami bezpieczeństwa.
* **Zabbix:** Monitoring dostępności i wydajności zasobów sprzętowych.
* **LGM Stack:** Agregacja logów (Loki) i wizualizacja metryk (Grafana).

### DevSecOps & Tools
* **Gitea:** Lokalny serwer Git z obsługą CI/CD (Gitea Runner).
* **Passbolt:** Rozwiązanie klasy Enterprise do bezpiecznego przechowywania haseł.
* **Nginx Proxy Manager:** Zarządzanie certyfikatami SSL i Reverse Proxy.

---

## 📊 System Health Statistics

<details>
<summary>⚡ Kliknij, aby zobaczyć aktualne metryki zasobów</summary>

### Temperatura procesora (Load: Active)
```bash
pi@pi:~ $ vcgencmd measure_temp
temp=67.5'C  # Status: Wymaga montażu aktywnego chłodzenia
