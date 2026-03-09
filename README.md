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


Poniższe zrzuty ekranu stanowią dowód namacalny, że zadeklarowane powyżej zabezpieczenia są aktywne i skuteczne. Rekruter/Audytor może zweryfikować stan faktyczny.

***

<details>
<summary>🔒 Audit: Weryfikacja zabezpieczeń (Terminal Logs)</summary>
<br>

Poniższe logi stanowią techniczny dowód, że mechanizmy hardeningu są aktywne. Zamiast polegać na obietnicach, weryfikujemy stan faktyczny bezpośrednio z konsoli.

***

### 🛡️ Dowód 1: Wyłączone Uwierzytelnianie Hasłem (Passwordless Auth)
Próba połączenia z nowej stacji roboczej (Fedora), która nie posiada jeszcze autoryzowanego klucza. Zwróć uwagę, że serwer **nie prosi o hasło**, lecz natychmiast odrzuca sesję, informując o braku klucza publicznego.

```bash
keinshur@fedora:~$ ssh pi@192.168.8.178
The authenticity of host '192.168.8.178 (192.168.8.178)' can't be established.
ED25519 key fingerprint is SHA256:rpT36czV+9WAerc4a9j4IYtyhIki3TTXYT8Ds8I/pvE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.8.178' (ED25519) to the list of known hosts.
pi@192.168.8.178: Permission denied (publickey).
```
***

🚫 Dowód 2: Zablokowane Logowanie Direct-Root
Nawet przy próbie logowania na konto administracyjne root, serwer odrzuca połączenie na poziomie protokołu SSH. Wymusza to na administratorze zalogowanie się jako standardowy użytkownik i eskalację uprawnień przez sudo.

```PowerShell
PS C:\Users\rafcie> ssh root@192.168.8.178
root@192.168.8.178: Permission denied (publickey).
```
✅ Dowód 3: Autoryzowany dostęp (Klucz + Passphrase)
Prawidłowy proces logowania z głównej stacji Windows. Zwróć uwagę na wymóg podania Passphrase do klucza prywatnego – to dodatkowa warstwa zabezpieczeń (2FA) na wypadek kradzieży fizycznego urządzenia.

```PowerShell
PS C:\Users\rafcie> ssh pi@192.168.8.178
Enter passphrase for key 'C:\Users\rafcie/.ssh/id_ed25519':
Linux pi 6.12.62+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.12.62-1+rpt1~bookworm (2026-01-19) aarch64

Last login: Mon Mar  9 18:48:41 2026 from 100.72.53.112
pi@pi:~ $
```
Inżynierska wskazówka dla audytora: Powyższe dowody wykazują wdrożenie strategii Defense in Depth. Nawet przejęcie kontroli nad siecią lokalną nie pozwala na nieautoryzowany dostęp bez posiadania unikalnego klucza i znajomości hasła do jego odszyfrowania.

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
