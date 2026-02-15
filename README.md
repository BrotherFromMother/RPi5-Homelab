# RPi5-Homelab
Enterprise-grade monitoring and security stack on Raspberry Pi 5


👋 Cześć, tu Rafał!
🎓 Student Informatyki (V semestr) | Junior System Administrator | Pasjonat Homelabów


Jestem na ostatniej prostej studiów inżynierskich, a moją pasją jest budowanie skalowalnej i bezpiecznej infrastruktury IT. Zamiast uczyć się tylko teorii, zarządzam własnym laboratorium opartym na architekturze hybrydowej, gdzie testuję rozwiązania klasy korporacyjnej.

Celem projektu jest zbudowanie skalowalnego i bezpiecznego środowiska serwerowego służącemu do testowania rozwiązań enterprise. 

Cała infrastruktura jest postawiona na Raspberry Pi 5 8GB z dyskiem M.2 NVMe. 

Wszytskie usługi postawiłem na Dockerze, do łatwiejszego zarządzania i większego bezpieczeństwa. Aktualne usługi (stacki) wraz z kontenerami wyglądają następująco:
Zabbix 
Wazuh single-node
Passbolt
Nginx-Proxy-Manager
Gitea
LGM ( Loki, Grafana Prometheus)

Wszystko jest zarządzane poprzez Porteiner oraz Gitea, a do łączenia się zdalnie do infrasktury używam narzędzia TailScale 



<details>
<summary>🐳 Kliknij, aby zobaczyć aktualny status kontenerów (dps)</summary>

<br>

```bash
pi@pi:~ $ dps
NAMES                           STATUS
monitoring-node-exporter        Up 16 hours
monitoring-promtail             Up 16 hours
monitoring-grafana              Up 16 hours
monitoring-loki                 Up 16 hours
monitoring-prometheus           Up 16 hours
single-node-wazuh.dashboard-1   Up About a minute
single-node-wazuh.manager-1     Up About a minute
single-node-wazuh.indexer-1     Up About a minute
zbx-web                         Up 16 hours (healthy)
zbx-agent                       Up 16 hours
zbx-server                      Up 16 hours
zbx-mysql                       Up 16 hours
gitea-runner                    Up 16 hours
gitea-srv                       Up 16 hours
gitea-tailscale                 Up 16 hours
gitea-db                        Up 16 hours
passbolt-app                    Up 16 hours
passbolt-db                     Up 16 hours
nginx-proxy-manager             Up 16 hours
portainer                       Up 16 hours
```

</details>




W planach mam kupno aktywnego chłodzenia do malinki, gdyż przy włączonym wazuhu temeperatura procesora wzrasta.



<details>
<summary>Kliknij by zobaczyć zużycie zasoób</summary>

<br>

```bash
pi@pi:~ $ vcgencmd measure_temp
temp=67.0'C
pi@pi:~ $ vcgencmd measure_temp
temp=67.5'C
pi@pi:~ $ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.9Gi       3.9Gi       239Mi        74Mi       4.0Gi       4.0Gi
Swap:          1.0Gi       638Mi       385Mi
```
</details>
