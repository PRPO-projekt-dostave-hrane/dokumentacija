# ADR-0002: Paginacija zbirk z odmikom

Status: sprejeto

Datum: 2026-11-13

## Kontekst

API ima dve zbirki, ki lahko vrneta veliko elementov: seznam restavracij (`GET /v1/restavracije`) in seznam naročil (`GET /v1/narocila`). Brez omejitve bi en klic lahko vrnil celotno tabelo, kar je počasno za strežnik in odjemalca.

Restavracije so vnaprej pripravljeni testni podatki, zato jih je malo. Naročil je več, a odjemalec vedno gleda naročila ene stranke ali ene restavracije, filtrirana po statusu. Pregled vseh preteklih naročil restavracije je redek.

## Odločitev

Zbirke paginiramo z odmikom: parametra `offset` (položaj prvega elementa, privzeto 0) in `limit` (število elementov, privzeto 20, največ 100).

Odgovor zbirke je objekt s poljem `items`, ne golo polje:

    {"items": [ ... ]}

Skupnega števila elementov ne vračamo.

## Posledice

- (+) preprosto za implementacijo (`LIMIT` in `OFFSET` v SQL) in za uporabo v odjemalcu
- (+) odjemalec lahko skoči na poljubno stran
- (-) pri velikih odmikih je poizvedba počasna
- (-) če se med listanjem doda ali izbriše element, se lahko element na meji strani podvoji ali izpusti

## Alternative

- **Kurzor**: hiter tudi pri velikih zbirkah in stabilen ob spremembah. Zavrnjeno, ker so zbirke majhne, implementacija pa zahtevnejša; poleg tega ni mogoč skok na poljubno stran.
- **Brez paginacije**: najpreprostejše, a zbirka naročil sčasoma raste brez omejitve, zato bi en klic vračal vse več podatkov.
- **Golo polje kot odgovor** (`[...]`): krajše, vendar zakomplicira kasnejše dodajanje metapodatkov.