# Assignment 52 – OSPF Route Redistribution på Juniper SRX

Dette repository indeholder de komplette Junos-konfigurationer til de tre SRX-routere i assignment 52:

- `r2-config.txt` – konfiguration til R2
- `r4-config.txt` – konfiguration til R4 (uændret fra assignment 51)
- `r5-config.txt` – konfiguration til R5 (uændret fra assignment 51)

Pers oprindelige diagram ligger i `topology.jpg`. Opgaven: <https://perper.gitlab.io/networkingpages/assignments/#assignment-52-ospf-route-redistribution>

## Hvad går konfigurationen ud på?

Udgangspunktet er assignment 51: R2, R4 og R5 i en trekant (Lan5, Lan6, Lan10) med OSPF i area `0.0.0.0`, loopbacks i OSPF og USERLAN'erne eksporteret med `route-filter ... exact`.

Nyt i 52 er, at USERLAN'erne får internetadgang via R2:

- R2's `ge-0/0/5` sidder på LabLan (`10.56.16.80/22`) i security-zonen `untrust`. Interfacet er **ikke** med i OSPF.
- R2 har en statisk default route `0.0.0.0/0 next-hop 10.56.16.1`.
- Export-policyen `my-default-static-route-to-internet` (`protocol static` + `route-filter 0.0.0.0/0 exact`) redistribuerer default routen ind i OSPF. R4 og R5 lærer den derfor som en OSPF external-rute uden selv at blive ændret.
- Source NAT (`interface`) fra zone `lab` til `untrust` skjuler de private net bag `10.56.16.80`, og en security policy tillader `lab` → `untrust`.
- R2 er DHCP-server for USERLAN1 på `ge-0/0/4`: PC5 får en adresse i `192.168.13.10–20` med gateway `192.168.13.1` og DNS `8.8.8.8`.

## Adresser (LLD)

| Router | Interface | Netværk | Adresse |
|---|---|---|---|
| R2 | ge-0/0/1 | Lan10 10.10.12.0/28 | 10.10.12.2/28 |
| R2 | ge-0/0/2 | Lan5 10.10.10.0/28 | 10.10.10.1/28 |
| R2 | ge-0/0/4 | USERLAN1 192.168.13.0/24 | 192.168.13.1/24 (PC5 = DHCP .10–.20) |
| R2 | ge-0/0/5 | LabLan 10.56.16.0/22 | 10.56.16.80/22 (gateway 10.56.16.1) |
| R2 | lo0 | – | 192.168.100.1/32 |
| R4 | ge-0/0/2 | Lan5 10.10.10.0/28 | 10.10.10.2/28 |
| R4 | ge-0/0/3 | Lan6 10.10.11.0/28 | 10.10.11.2/28 |
| R4 | ge-0/0/4 | USERLAN2 192.168.14.0/24 | 192.168.14.1/24 (PC10 = .5) |
| R4 | lo0 | – | 192.168.100.2/32 |
| R5 | ge-0/0/2 | Lan10 10.10.12.0/28 | 10.10.12.1/28 |
| R5 | ge-0/0/3 | Lan6 10.10.11.0/28 | 10.10.11.1/28 |
| R5 | ge-0/0/4 | USERLAN3 192.168.15.0/24 | 192.168.15.1/24 (PC11 = .5) |
| R5 | lo0 | – | 192.168.100.3/32 |

PC10 og PC11 har statiske adresser med routerens `.1` som default gateway.

## Indlæsning

Kun R2 skal ændres – R4 og R5 kører allerede assignment 51-konfigurationen. Fra Junos configuration mode på R2:

```text
load override terminal
```

Indsæt hele filen, afslut med `Ctrl+D`, og kontrollér med `show | compare` og `commit check`. Brug gerne `commit confirmed 5`.

> **Bemærk:** `load override` erstatter hele routerens konfiguration. Filerne indeholder de krypterede root-passwordhashes fra labrouterne og bør kun bruges på de tilsigtede enheder.

## Verifikation

```text
R2> show route 0.0.0.0/0
R2> show dhcp server binding
R4> show route
R4> show route 0.0.0.0/0 detail
R4> show ospf database external
PC11$ traceroute -I -n 10.56.16.1
```

På R4 står `0.0.0.0/0` som `OSPF` med preference 150 (external, redistribueret fra R2's statiske rute), ligesom de andre routeres USERLAN'er. Lan- og loopback-ruter er interne (preference 10).

Forventet traceroute fra PC11 med OSPF-metrics fra assignment 51 (R5→R4→R2 koster 1+1=2, Lan10 direkte koster 10):

1. `192.168.15.1` – R5
2. `10.10.11.2` – R4 (Lan6)
3. `10.10.10.1` – R2 (Lan5)
4. `10.56.16.1` – internet-gateway

## Status

Konfigurationerne er skrevet men **endnu ikke indlæst på routerne eller testet**. HLD, R4's routingtabel og traceroute fra PC11 mangler.
