# Scenariusz failover w OSPF

Ten dokument opisuje scenariusz wykorzystania zapasowego połączenia w protokole OSPF. Celem testu jest pokazanie, że OSPF potrafi automatycznie przeliczyć trasę po awarii głównego linku i przekierować ruch na trasę zapasową.

Projekt został wykonany w środowisku GNS3 z wykorzystaniem routerów Cisco C7200. Scenariusz dotyczy obszaru `Area 3`, w którym znajdują się routery R3, R6, R7 oraz R8.

## Cel scenariusza

Celem scenariusza jest sprawdzenie działania mechanizmu `failover` w topologii OSPF.

W projekcie zastosowano dwa możliwe kierunki komunikacji między routerami R6 i R8:

- ścieżka podstawowa przez router R7,
- ścieżka zapasowa przez bezpośredni link R6 - R8.

Dzięki manipulacji parametrem `OSPF cost` protokół OSPF powinien w normalnych warunkach wybierać trasę przez R7. Bezpośrednie połączenie R6 - R8 powinno być używane dopiero wtedy, gdy ścieżka podstawowa przestanie działać.

## Fragment topologii

Scenariusz dotyczy następującego fragmentu sieci:

```txt
        R7
       /  \
      /    \
     R6----R8
```

W tej części topologii występują trzy połączenia:

| Połączenie | Sieć | Typ linku | OSPF area |
|---|---|---|---|
| R6 - R7 | `192.168.37.0/30` | FastEthernet | Area 3 |
| R7 - R8 | `192.168.38.0/30` | FastEthernet | Area 3 |
| R6 - R8 | `192.168.39.0/30` | FastEthernet | Area 3 |

## Ścieżka podstawowa

W normalnych warunkach ruch pomiędzy R6 i R8 powinien przechodzić przez R7:

```txt
R6 -> R7 -> R8
```

Ta ścieżka jest preferowana, ponieważ ma niższy łączny `OSPF cost`.

## Ścieżka zapasowa

Bezpośredni link pomiędzy R6 i R8 pełni funkcję `backup link`:

```txt
R6 -> R8
```

Ten link ma ustawiony wyższy `OSPF cost`, dlatego nie powinien być wybierany jako główna trasa, dopóki działa ścieżka przez R7.

## Manipulacja kosztami OSPF

W projekcie ręcznie ustawiono koszty OSPF na wybranych interface, aby wymusić konkretną logikę wyboru trasy.

| Router | Interface | Link | OSPF cost | Rola |
|---|---|---|---|---|
| R6 | FastEthernet1/0 | R6 - R7 | 1 | Link podstawowy |
| R6 | FastEthernet1/1 | R6 - R8 | 100 | Backup link |
| R8 | FastEthernet0/0 | R8 - R7 | 1 | Link podstawowy |
| R8 | FastEthernet1/0 | R8 - R6 | 100 | Backup link |

Dzięki temu OSPF powinien wybrać trasę przez R7, ponieważ jej koszt jest niższy niż koszt bezpośredniego połączenia R6 - R8.

## Założenia testu

Przed wykonaniem testu należy upewnić się, że:

- wszystkie routery w `Area 3` są uruchomione,
- interface między R6, R7 i R8 są w stanie `up/up`,
- sąsiedztwa OSPF zostały poprawnie zestawione,
- routery posiadają wpisy OSPF w `routing table`,
- backup link R6 - R8 ma wyższy koszt niż trasa przez R7.

## Komendy weryfikacyjne przed awarią

Na routerach R6, R7 i R8 warto wykonać poniższe komendy.

### Sprawdzenie stanu interface

```bash
show ip interface brief
```

Ta komenda pozwala sprawdzić, czy interface są aktywne i czy posiadają poprawnie przypisane adresy IP.

### Sprawdzenie sąsiedztw OSPF

```bash
show ip ospf neighbor
```

Ta komenda pokazuje routery sąsiednie OSPF oraz stan relacji. Poprawnie działające sąsiedztwo powinno osiągnąć stan `FULL`.

### Sprawdzenie tras OSPF

```bash
show ip route ospf
```

Ta komenda pokazuje trasy nauczone dynamicznie przez OSPF.

### Sprawdzenie pełnej routing table

```bash
show ip route
```

Ta komenda pozwala sprawdzić, która trasa została wybrana jako najlepsza.

### Sprawdzenie konfiguracji OSPF

```bash
show ip protocols
```

Ta komenda pokazuje informacje o uruchomionym procesie OSPF, rozgłaszanych sieciach oraz parametrach routingu.

## Test ścieżki przed awarią

Aby sprawdzić, którędy przechodzi ruch, można wykonać test `traceroute` z R6 do adresu routera R8.

Przykładowo:

```bash
traceroute 192.168.38.2
```

Oczekiwany przebieg trasy przed awarią:

```txt
R6 -> R7 -> R8
```

Oznacza to, że OSPF wybrał ścieżkę o niższym koszcie przez router R7.

## Symulacja awarii głównego linku

Aby zasymulować awarię ścieżki podstawowej, można wyłączyć jeden z interface znajdujących się na trasie przez R7.

Przykładowo na R6 można wyłączyć interface prowadzący do R7:

```bash
configure terminal
interface FastEthernet1/0
shutdown
end
```

Alternatywnie można wyłączyć interface po stronie R7 prowadzący do R6 albo R8.

Po wykonaniu tej czynności OSPF powinien wykryć zmianę topologii i rozpocząć proces konwergencji.

## Konwergencja OSPF

Po awarii linku OSPF aktualizuje informacje o stanie łączy i przelicza najlepsze trasy za pomocą algorytmu SPF.

W tym momencie router powinien usunąć niedostępną trasę przez R7 i wybrać alternatywną trasę przez backup link R6 - R8.

## Komendy weryfikacyjne po awarii

Po wyłączeniu linku należy ponownie wykonać:

```bash
show ip ospf neighbor
```

```bash
show ip route ospf
```

```bash
show ip route
```

```bash
traceroute 192.168.38.2
```

Oczekiwany przebieg trasy po awarii:

```txt
R6 -> R8
```

Jeżeli trasa zmieniła się na bezpośredni link R6 - R8, oznacza to, że mechanizm `failover` działa poprawnie.

## Przywrócenie głównego linku

Po zakończeniu testu można ponownie uruchomić wyłączony interface.

Przykład dla R6:

```bash
configure terminal
interface FastEthernet1/0
no shutdown
end
```

Po przywróceniu linku OSPF powinien ponownie zestawić sąsiedztwo i po konwergencji wrócić do preferowanej ścieżki przez R7.

## Oczekiwany wynik końcowy

Oczekiwanym wynikiem testu jest potwierdzenie, że:

- w normalnych warunkach OSPF wybiera trasę R6 - R7 - R8,
- bezpośredni link R6 - R8 działa jako `backup link`,
- po awarii głównego połączenia OSPF automatycznie przelicza trasę,
- ruch zostaje przekierowany przez link zapasowy,
- po przywróceniu głównego linku OSPF może ponownie wybrać trasę o niższym koszcie.