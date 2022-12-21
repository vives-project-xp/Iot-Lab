# Network IP from device

* Home assistant : 172.30.245.10

* Camera 1 : 172.30.245.20
* Camera 2 : 172.30.245.30

## statisch IP via CLI

Om de statische IP aan te passen of in te stellen via de CLI kan gebruik gemaakt worden van nmcli.
Voer de volgende commands uit.

1. login
2. nmcli con edit "connection name" (bv. wlan0)
3. set ipv4 addresses 172.30.245.10/16
4. indien je gevraagd wordt of je ipv4.method op manual wil zetten antwoord yes
5. set ipv4.dns 1.1.1.1
6. set ipv4.gateway "je gateway"
7. save
8. quit
9. reboot
