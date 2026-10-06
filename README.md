# Assignment 52 – OSPF Route Redistribution på Juniper SRX

Junos-konfigurationer til de tre SRX-routere i assignment 52. Hver router findes som `.txt` (kan indlæses med `load override terminal`) og som `.json` (samme konfiguration som struktureret reference).

| Fil | Router |
|---|---|
| `r2-config.txt` / `r2-config.json` | R2 – kant-router mod LabLan/internettet, DMZ og USERLAN1 |
| `r4-config.txt` / `r4-config.json` | R4 – USERLAN2 (uændret fra assignment 51) |
| `r5-config.txt` / `r5-config.json` | R5 – USERLAN3 (uændret fra assignment 51) |

Pers diagram ligger i `topology.jpg`. Opgaven: <https://perper.gitlab.io/networkingpages/assignments/#assignment-52-ospf-route-redistribution>

## Konfigurationerne kort

**Fælles (fra assignment 51):** R2, R4 og R5 er forbundet i en trekant (Lan5, Lan6, Lan10) og kører OSPF i area `0.0.0.0`. Loopbacks er med i OSPF, og hver routers USERLAN annonceres med en export-policy (`route-filter ... exact`).

**R2** er routeren med det nye i assignment 52:

- **Default route:** en statisk `0.0.0.0/0` mod gatewayen `10.56.16.1` på LabLan (`ge-0/0/5`, zone `untrust`). Den redistribueres ind i OSPF med policyen `my-default-static-route-to-internet`, så R4 og R5 automatisk lærer vejen til internettet.
- **Source NAT:** trafik fra USERLAN'erne og DMZ'en oversættes til R2's LabLan-adresse `10.56.16.85`.
- **DHCP:** R2 deler adresser ud til PC5 i USERLAN1 (`192.168.13.10–20`).
- **DMZ:** PC3 er Local Web Server på `192.168.12.55`. Den kan nås udefra på `10.56.16.85:80` via destination NAT. Brugerne må tilgå DMZ'en, men DMZ'en må ikke tilgå brugernes net (`deny-DMZ-to-lab`).

**R4 og R5** er ikke ændret. De får default routen fra R2 via OSPF som en external-rute.

## Adresser (LLD)

| Router | Interface | Netværk | Adresse |
|---|---|---|---|
| R2 | ge-0/0/1 | Lan10 10.10.12.0/28 | 10.10.12.2/28 |
| R2 | ge-0/0/2 | Lan5 10.10.10.0/28 | 10.10.10.1/28 |
| R2 | ge-0/0/3 | DMZ 192.168.12.0/24 | 192.168.12.1/24 (PC3 webserver = .55) |
| R2 | ge-0/0/4 | USERLAN1 192.168.13.0/24 | 192.168.13.1/24 (PC5 = DHCP .10–.20) |
| R2 | ge-0/0/5 | LabLan 10.56.16.0/22 | 10.56.16.85/22 (gateway 10.56.16.1) |
| R2 | lo0 | – | 192.168.100.1/32 |
| R4 | ge-0/0/2 | Lan5 10.10.10.0/28 | 10.10.10.2/28 |
| R4 | ge-0/0/3 | Lan6 10.10.11.0/28 | 10.10.11.2/28 |
| R4 | ge-0/0/4 | USERLAN2 192.168.14.0/24 | 192.168.14.1/24 (PC10 = .5) |
| R4 | lo0 | – | 192.168.100.2/32 |
| R5 | ge-0/0/2 | Lan10 10.10.12.0/28 | 10.10.12.1/28 |
| R5 | ge-0/0/3 | Lan6 10.10.11.0/28 | 10.10.11.1/28 |
| R5 | ge-0/0/4 | USERLAN3 192.168.15.0/24 | 192.168.15.1/24 (PC11 = .5) |
| R5 | lo0 | – | 192.168.100.3/32 |

PC3, PC10 og PC11 har statiske adresser med routerens `.1` som default gateway.

> **Bemærk:** `load override` erstatter hele routerens konfiguration. Filerne indeholder de krypterede root-passwordhashes fra labrouterne og bør kun bruges på de tilsigtede enheder.
