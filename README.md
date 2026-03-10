# 🛡️ RPi5-Homelab: Enterprise Security & Monitoring Stack
> **High-availability internal infrastructure project based on Raspberry Pi 5.**

![OS](https://img.shields.io/badge/OS-Debian%20Bookworm-blue?logo=debian)
![Docker](https://img.shields.io/badge/Containerization-Docker-blue?logo=docker)
![Security](https://img.shields.io/badge/Security-Hardened-success?logo=skyliner)
![Architecture](https://img.shields.io/badge/Arch-ARM64-orange)

---


<details>
<summary>  🛡️ Bezpieczne SSH  </summary>
 

<br>


> Bezpiecześtwo jest na pierwszym miejscu, postanowiłem zabezpieczyć moją Malinkę o nastepującę parametry:

<br> 

### 🛡️ Key Implementations:
* **Kryptografia Asymetryczna:** Całkowita rezygnacja z haseł na rzecz par kluczy **Ed25519** (krzywe eliptyczne).
* **Identity Verification:** Dostęp do powłoki systemu ograniczony wyłącznie do autoryzowanych hostów (Windows/Fedora) z aktywnym kluczem prywatnym.
* **Root Access Policy:** Implementacja `PermitRootLogin no` – wymuszenie eskalacji uprawnień poprzez `sudo`.
* **SSH Configuration:** Optymalizacja parametru `MaxAuthTries 5` oraz wyłączenie `PasswordAuthentication`.
* **Fail2Ban:**: Zastosowanie prostego IPS (Intrusion Prevention System) w celu banowania adresu IP po wykryciu Brute Force
---





<details>
<summary>🔒🛡️ Dowód 1: Wyłączone Uwierzytelnianie Hasłem (Passwordless Auth)</summary>
<br>

Próba połączenia z nowej stacji roboczej (Fedora), która nie posiada jeszcze autoryzowanego klucza. Zwróć uwagę, że serwer **nie prosi o hasło**, lecz natychmiast odrzuca sesję, informując o braku klucza publicznego.

```bash
keinshur@fedora:~$ ssh pi@192.168.8.178
The authenticity of host '192.168.8.178 (192.168.8.178)' can't be established.
ED25519 key fingerprint is SHA256:rpT36czV+9WAerc4a9j4IYtyhIki3TTXYT8Ds8I/pvE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.8.178' (ED25519) to the list of known hosts.
pi@192.168.8.178: Permission denied (publickey).
```
</details>


***


<details>
<summary>🚫 Dowód 2: Zablokowane Logowanie Direct-Root</summary>
<br>

Nawet przy próbie logowania na konto administracyjne root, serwer odrzuca połączenie na poziomie protokołu SSH. Wymusza to na administratorze zalogowanie się jako standardowy użytkownik i eskalację uprawnień przez sudo.

```PowerShell
PS C:\Users\rafcie> ssh root@192.168.8.178
root@192.168.8.178: Permission denied (publickey).
```

</details>


***


<details>
<summary>✅ Dowód 3: Autoryzowany dostęp (Klucz + Passphrase)</summary>
<br>

Prawidłowy proces logowania z głównej stacji Windows. Zwróć uwagę na wymóg podania Passphrase do klucza prywatnego – to dodatkowa warstwa zabezpieczeń (2FA) na wypadek kradzieży fizycznego urządzenia.

```PowerShell
PS C:\Users\rafcie> ssh pi@192.168.8.178
Enter passphrase for key 'C:\Users\rafcie/.ssh/id_ed25519':
Linux pi 6.12.62+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.12.62-1+rpt1~bookworm (2026-01-19) aarch64

Last login: Mon Mar  9 18:48:41 2026 from 100.72.53.112
pi@pi:~ $
```
Nawet przejęcie kontroli nad siecią lokalną nie pozwala na nieautoryzowany dostęp bez posiadania unikalnego klucza i znajomości hasła do jego odszyfrowania.
</details>


***


<details>
<summary>🛡️ Phase 4: Active Incident Response & Monitoring (IPS)</summary>
<br>

 ***
 
> W tej fazie projekt przeszedł z pasywnej ochrony (hardening) do aktywnej obrony. Zaimplementowano system **IPS (Intrusion Prevention System)**, który w czasie rzeczywistym analizuje próby naruszenia bezpieczeństwa i podejmuje autonomiczne działania obronne.

***
<details>
<summary>⚙️ Kluczowe parametry systemu oraz konfiguracja strażnika </summary>
<br>
 
* **Silnik detekcji:** Fail2Ban zintegrowany bezpośrednio z szyną zdarzeń `systemd`.
* **Polityka agresywności:** Tryb `mode = extra` — monitorowanie nie tylko błędnych haseł, ale i odrzuconych prób negocjacji kluczy publicznych (preauth).
* **Threshold (Próg):** Limit **5 nieudanych prób** w określonym czasie skutkuje natychmiastową blokadą adresu IP na firewallu.
* **Integracja SMTP:** Wykorzystanie `msmtp` do przesyłania raportów o incydentach (Real-time Alerting).

## 📄 Konfiguracja Strażnika (`/etc/fail2ban/jail.local`):
```ini
[sshd]
enabled  = true
port     = ssh
backend  = systemd
mode     = extra
maxretry = 5
bantime  = 1h
ignoreip = 127.0.0.1/8 ::1 192.168.8.176  # Biała lista administratora
action   = %(action_mwl)s[lines=20]       # Ban + Mail z raportem WHOIS i logami
```
</details>

***



<details>
<summary>🔍 Audit: Dowody aktywnej obrony (Real-time Logs)</summary>


<br>

Poniżej prezentuję działanie Fail2Ban oraz powidomień SMTP

<br>

🛡️ Dowód 1: Wykrycie ataku i nałożenie blokady (Fail2Ban Log)
Wycinek z /var/log/fail2ban.log pokazujący proces zliczania prób i finalny ban dla hosta 192.168.8.186

<br>

```Plaintext
2026-03-09 20:11:16,792 fail2ban.filter : INFO [sshd] Found 192.168.8.186
2026-03-09 20:11:18,034 fail2ban.filter : INFO [sshd] Found 192.168.8.186
2026-03-09 20:11:19,386 fail2ban.actions: NOTICE [sshd] Ban 192.168.8.186
```
<br>

📩 Dowód 2: Skuteczna wysyłka alertu przez bramkę SMTP
Logi z usługi msmtp potwierdzające, że serwer Gmail przyjął i przetworzył powiadomienie o incydencie (Status 250 OK).

```Plaintext
Mar 09 20:11:21 host=smtp.gmail.com tls=on auth=on user=rafal.ciereszko.2abt@gmail.com 
recipients=rafal.important.information@gmail.com mailsize=3241 
smtpstatus=250 smtpmsg='250 2.0.0 OK 1773083481 ... - gsmtp' exitcode=EX_OK
```
Połączenie narzędzi Fail2Ban oraz msmtp tworzy niezawodną pętlę sprzężenia zwrotnego. Administrator jest informowany o ataku w ciągu < 3 sekund od jego wystąpienia, co skraca czas reakcji na incydent do minimum.


</details>

</details>

*** 

</details>

***

### Kolejny rodział


***

