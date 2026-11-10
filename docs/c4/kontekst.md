# C4 diagrami

## Nivo 1: kontekst sistema

```mermaid
flowchart TB
    stranka["<b>Stranka</b><br/>[uporabnik]"]
    restavracija["<b>Restavracija</b><br/>[uporabnik]"]

    sistem["<b>Dostava hrane</b><br/>[sistem]<br/>katalog restavracij, izvedba ter sledenje naročila, obdelava plačil"]

    ponudnik["<b>Plačilni ponudnik</b><br/>[zunanji sistem, simuliran]<br/>simulira uspešno plačilo naročila"]

    stranka -- "pregleduje menije, naroča,<br/>plača, sledi statusu, ocenjuje" --> sistem
    restavracija -- "ureja meni, sprejema/zavrača<br/>naročila, označi dostavo" --> sistem
    sistem -- "zahteva plačilo" --> ponudnik

    classDef uporabnik fill:#08427b,stroke:#052e56,color:#fff
    classDef sistem fill:#1168bd,stroke:#0b4884,color:#fff
    classDef zunanji fill:#999999,stroke:#6b6b6b,color:#fff
    class stranka,restavracija uporabnik
    class sistem sistem
    class ponudnik zunanji
```

## Nivo 2: storitve

```mermaid
flowchart TB
    stranka["<b>Stranka</b><br/>[uporabnik]"]
    restavracija["<b>Restavracija</b><br/>[uporabnik]"]

    subgraph sistem["Dostava hrane"]
        spa["<b>Spletna aplikacija</b><br/>[Angular]<br/>vmesnik za stranke in restavracije"]

        katalog["<b>katalog</b><br/>[Quarkus]<br/>restavracije, meniji, povprečne ocene"]
        narocila["<b>narocila</b><br/>[Quarkus]<br/>naročila, statusi, ocene"]
        placila["<b>placila</b><br/>[Quarkus]<br/>obdelava plačila"]

        dbp[("<b>placila-db</b><br/>[PostgreSQL]")]
        dbk[("<b>katalog-db</b><br/>[PostgreSQL]")]
        dbn[("<b>narocila-db</b><br/>[PostgreSQL]")]
    end

    ponudnik["<b>Plačilni ponudnik</b><br/>[zunanji sistem, simuliran]<br/>simulirano plačilo"]

    stranka --> spa
    restavracija --> spa

    spa -- "REST: pogled na restavracije in menije" --> katalog
    spa -- "REST: oddaj naročilo in stanje naročila" --> narocila

    narocila -- "REST: preveri jedi in cene" --> katalog
    narocila -. "nova ocena restavracije" .-> katalog
    narocila -- "REST: plačaj naročilo" --> placila

    placila -- "zahteva plačilo" --> ponudnik

    katalog --> dbk
    narocila --> dbn
    placila --> dbp

    classDef uporabnik fill:#08427b,stroke:#052e56,color:#fff
    classDef storitev fill:#1168bd,stroke:#0b4884,color:#fff
    classDef baza fill:#438dd5,stroke:#2e6295,color:#fff
    classDef zunanji fill:#999999,stroke:#6b6b6b,color:#fff
    class stranka,restavracija uporabnik
    class spa,katalog,placila,narocila storitev
    class ponudnik zunanji
    class dbk,dbn,dbp baza
```

## Legenda

- **temno modro**: uporabniki sistema
- **modro**: naš sistem oziroma njegove storitve
- **sivo**: zunanji sistemi
- **svetlo modro**: baze
- **polna črta**: sinhron klic (pošiljatelj čaka na odgovor)
- **črtkana črta**: asinhrono sporočilo



