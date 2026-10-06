# Assignment 52 – OSPF Route Redistribution

Konfigurationer til de tre SRX-routere i JSON: `r2-config.json`, `r4-config.json` og `r5-config.json`.

## Routerne

- **R2** – forbindelsen til internettet. Har en default route mod `10.56.16.1`, som sendes ud til de andre routere via OSPF. Laver NAT, deler IP-adresser ud med DHCP til USERLAN1 og har DMZ'en med webserveren (PC3).
- **R4** – har USERLAN2. Får default routen fra R2 via OSPF.
- **R5** – har USERLAN3. Får default routen fra R2 via OSPF.

Alle tre kører OSPF i area 0 og er forbundet i en trekant.

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
