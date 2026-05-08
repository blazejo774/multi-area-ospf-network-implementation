# Plan adresacji

Ten dokument przedstawia plan adresacji IPv4 użyty w projekcie multi-area OSPF wykonanym w środowisku GNS3.

Wszystkie połączenia typu router-router zostały zaadresowane jako sieci punkt-punkt z maską `/30`. Takie rozwiązanie pozwala efektywnie wykorzystać adresację IPv4, ponieważ dla każdego linku dostępne są dokładnie dwa użyteczne adresy IP.

## Tabela adresacji

| Połączenie | Sieć | R1 IP | R2 IP | R3 IP | R4 IP | R5 IP | R6 IP | R7 IP | R8 IP | OSPF area |
|---|---|---|---|---|---|---|---|---|---|---|
| R1 - R2 | 192.168.12.0/30 | 192.168.12.1 | 192.168.12.2 | - | - | - | - | - | - | Area 0 |
| R2 - R3 | 192.168.23.0/30 | - | 192.168.23.1 | 192.168.23.2 | - | - | - | - | - | Area 0 |
| R1 - R3 | 192.168.13.0/30 | 192.168.13.1 | - | 192.168.13.2 | - | - | - | - | - | Area 4 |
| R2 - R4 | 192.168.24.0/30 | - | 192.168.24.1 | - | 192.168.24.2 | - | - | - | - | Area 1 |
| R3 - R5 | 192.168.35.0/30 | - | - | 192.168.35.1 | - | 192.168.35.2 | - | - | - | Area 2 |
| R3 - R6 | 192.168.36.0/30 | - | - | 192.168.36.1 | - | - | 192.168.36.2 | - | - | Area 3 |
| R6 - R7 | 192.168.37.0/30 | - | - | - | - | - | 192.168.37.1 | 192.168.37.2 | - | Area 3 |
| R7 - R8 | 192.168.38.0/30 | - | - | - | - | - | - | 192.168.38.1 | 192.168.38.2 | Area 3 |
| R6 - R8 | 192.168.39.0/30 | - | - | - | - | - | 192.168.39.1 | - | 192.168.39.2 | Area 3 |

## Założenia adresacji

- Każdy link router-router używa osobnej podsieci `/30`.
- Adresacja jest czytelna i zgodna z numerami routerów, np. sieć `192.168.12.0/30` oznacza połączenie R1 - R2.
- Backbone OSPF znajduje się w `Area 0`.
- Dodatkowe obszary OSPF zostały podzielone na `Area 1`, `Area 2`, `Area 3` oraz `Area 4`.