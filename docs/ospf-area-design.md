# Projekt obszarów OSPF

Opis podziału topologii na obszary OSPF. Projekt został wykonany jako sieć multi-area OSPF w środowisku GNS3 z wykorzystaniem routerów Cisco C7200.

## Założenie projektu

Celem projektu było przygotowanie topologii, która pokazuje działanie protokołu OSPF w strukturze wieloobszarowej. W sieci zastosowano jeden główny obszar backbone oraz kilka dodatkowych obszarów podłączonych do routerów brzegowych.

## Area 0 - Backbone

`Area 0` jest głównym obszarem OSPF. Pełni rolę backbone, czyli centralnej części domeny routingu.

Do `Area 0` należą połączenia:

- R1 - R2
- R2 - R3

Routery pracujące w tej części topologii odpowiadają za podstawową komunikację między pozostałymi obszarami.

## Area 1

`Area 1` obejmuje połączenie:

- R2 - R4

Router R2 pełni w tym przypadku rolę routera łączącego `Area 1` z backbone.

## Area 2

`Area 2` obejmuje połączenie:

- R3 - R5

Router R3 pełni rolę routera brzegowego dla tego obszaru.

## Area 3

`Area 3` jest najbardziej rozbudowanym obszarem w projekcie. Znajdują się w nim routery:

- R3
- R6
- R7
- R8

W tym obszarze zastosowano połączenia FastEthernet oraz mechanizm redundantnego linku. Bezpośrednie połączenie R6 - R8 pełni funkcję `backup link`.

Połączenia w `Area 3`:

- R3 - R6
- R6 - R7
- R7 - R8
- R6 - R8

## Area 4

`Area 4` obejmuje połączenie:

- R1 - R3

Ten obszar tworzy dodatkową trasę pomiędzy routerami R1 i R3.

## Routery pełniące rolę ABR

W tej topologii wybrane routery łączą więcej niż jeden obszar OSPF, dlatego pełnią rolę `ABR`, czyli Area Border Router.

Przykładowe routery ABR:

- R1 - połączenie Area 0 i Area 4
- R2 - połączenie Area 0 i Area 1
- R3 - połączenie Area 0, Area 2, Area 3 i Area 4

Dzięki temu topologia pokazuje praktyczne działanie routingu między różnymi obszarami OSPF.