# ADR-0001: Razrez sistema na storitve katalog, narocila in placila

Status: sprejeto

Datum: 2026-11-10 

## Kontekst 

Sistem za dostavo hrane mora strankam omogočiti pregled restavracij, naročanje, plačilo, sledenje dostavi in ocenjevanje, restavracijam pa upravljanje menijev in sprejemanje naročil. Zahtevani sta vsaj dve mikrostoritvi, vsako v svojem repozitoriju in s svojo bazo.

Isti pojmi imajo v različnih delih sistema različen pomen. Jed v meniju ima trenutno ceno in razpoložljivost; jed v naročilu mora hraniti ID jedi ter ime in ceno ob času naročila, sicer bi podražitev jedi spremenila že oddana naročila.

Plačilo poteka preko (simuliranega) zunanjega ponudnika, ki je hard-coded da se odzove uspešno.

Projekt razvija en sam razvijalec, zato mora biti obseg obvladljiv.

## Odločitev 

Sistem razdelimo na tri storitve po poslovnih zmožnostih, vsako v svojem repozitoriju:

- **katalog**: restavracije, njihove povprečne ocene in meniji; ti podatki so vnaprej pripravljeni (mock data)
- **narocila**: življenjski cikel naročila (od oddano do dostavljeno, poleg tega še zavrnjeno in preklicano) in ocene, ki jih pošilja storitvi "katalog" za izračun povprečne ocene
- **placila**: plačilo naročila prek simuliranega zunanjega ponudnika; vedno vrne uspeh in ID transakcije, ki ga naročila shranijo k naročilu

## Posledice 

- (+) meje storitev sledijo mejam pomena
- (+) vsaka storitev ima svoj repozitorij in bazo ter jo lahko spreminjam neodvisno
- (+) zamenjava simuliranega plačila s pravim ponudnikom ne vpliva na ostale storitve
- (-) narocila potrebujejo podatke o jedeh iz kataloga, zato je med njima sinhron klic
- (-) med narocila in placila je dodatna komunikacija za isto naročilo
- (-) ni skupne transakcije čez storitve
- (-) tri storitve pomenijo tri repozitorije, cevovode, image-e in namestitve za enega razvijalca

## Alternative 

- **Monolit**: enostavnejši za enega razvijalca, a ne izpolnjuje zahteve predmeta po mikrostoritvah.
- **Plačilo kot del storitve narocila**: manj dela in brez komunikacije med dvema storitvama za isto plačilo. Zavrnjeno, ker bi sprememba načina plačila (npr. pravi ponudnik) zahtevala spremembe v storitvi naročil.
- **Storitev za e-poštna obvestila**: obveščala bi stranke o spremembi statusa naročila. Zavrnjeno, ker za enega razvijalca preveč poveča obseg; stranka status spremlja v aplikaciji.