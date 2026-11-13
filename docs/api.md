# Načrt REST API-ja

Tukaj je načrt za API storitev katalog, narocila in placila. Je osnova za OpenAPI specifikacijo v repozitoriju vsake storitve.

## Katalog

### Viri

| Vir | Kanonični URI | Opis |
|---|---|---|
| restavracija | `/v1/restavracije/{id}` | ime, vrsta kuhinje, naslov, povprečna ocena |
| jed | `/v1/restavracije/{id}/jedi/{id}` | ime, opis, cena, ali je na voljo |

### Operacije

| Operacija | Metoda in URI | Uspeh | Napake |
|---|---|---|---|
| seznam restavracij | `GET /v1/restavracije` | 200 | 400 |
| ena restavracija | `GET /v1/restavracije/{id}` | 200 | 404 |
| seznam jedi (meni) restavracije | `GET /v1/restavracije/{id}/jedi` | 200 | 404 |
| ena jed | `GET /v1/restavracije/{id}/jedi/{id}` | 200 | 404 |
| dodaj jed v meni | `POST /v1/restavracije/{id}/jedi` | 201 + Location | 404, 422 |
| spremeni jed (z `If-Match`) | `PATCH /v1/restavracije/{id}/jedi/{id}` | 200 | 404, 412, 422, 428 |
| odstrani jed | `DELETE /v1/restavracije/{id}/jedi/{id}` | 204 | 404 |

## Naročila

### Viri

| Vir | Kanonični URI | Opis |
|---|---|---|
| naročilo | `/v1/narocila/{id}` | podrobnosti naročila |
| sprejem | `/v1/narocila/{id}/sprejem` | restavracija sprejme naročilo |
| zavrnitev | `/v1/narocila/{id}/zavrnitev` | restavracija zavrne naročilo |
| preklic | `/v1/narocila/{id}/preklic` | stranka prekliče naročilo |
| placilo | `/v1/narocila/{id}/placilo` | stranka plača po sprejemu |
| prevzem | `/v1/narocila/{id}/prevzem` | jed je pripravljena in na poti |
| dostava | `/v1/narocila/{id}/dostava` | jed je prevzela stranka |
| ocena | `/v1/narocila/{id}/ocena` | ocena 1–5 in komentar po dostavi |

### Statusi in prehodi

Prikaz prehodov stanj enega naročila.

```
CAKA_NA_SPREJEM --sprejem--> SPREJETO --placilo--> PLACANO --prevzem--> V_DOSTAVI --dostava--> DOSTAVLJENO
                --sprejem--> SPREJETO --preklic----> PREKLICANO
                --zavrnitev--> ZAVRNJENO
                --preklic----> PREKLICANO
```

### Operacije

| Operacija | Metoda in URI | Uspeh | Napake |
|---|---|---|---|
| ustvari naročilo | `POST /v1/narocila` | 201 + Location | 404, 409, 422, 503 |
| seznam naročil | `GET /v1/narocila` | 200 | 400 |
| eno naročilo | `GET /v1/narocila/{id}` | 200 | 404 |
| prekliči naročilo | `POST /v1/narocila/{id}/preklic` | 200 | 404, 409 |
| zavrni naročilo | `POST /v1/narocila/{id}/zavrnitev` | 200 | 404, 409 |
| sprejmi naročilo | `POST /v1/narocila/{id}/sprejem` | 200 | 404, 409 |
| plačaj | `POST /v1/narocila/{id}/placilo` z `Idempotency-Key` | 200 | 400, 404, 409, 503 |
| prevzem za dostavo | `POST /v1/narocila/{id}/prevzem` | 200 | 404, 409 |
| dostavljeno | `POST /v1/narocila/{id}/dostava` | 200 | 404, 409 |
| oceni | `POST /v1/narocila/{id}/ocena` | 201 + Location | 404, 409, 422 |

Pri ustvarjanju naročila storitev narocila za vsako postavko pokliče `GET /v1/restavracije/{id}/jedi/{id}` v katalogu in shrani ime in ceno. Če jed ne obstaja, vrne `404`, če ni na voljo, `409`.

Pri plačilu storitev narocila pokliče `POST /v1/placila` in k naročilu shrani ID transakcije. Če storitev placila ne odgovori, vrne `503`.

Ocena je mogoča samo za naročilo s statusom `DOSTAVLJENO` in samo enkrat. Po oceni storitev narocila asinhrono sporoči katalogu novo oceno restavracije.

## Plačila

### Viri

| Vir | Kanonični URI | Opis |
|---|---|---|
| plačilo | `/v1/placila/{id}` | ID naročila, znesek, status, ID transakcije |

### Operacije

| Operacija | Metoda in URI | Uspeh | Napake |
|---|---|---|---|
| izvedi plačilo | `POST /v1/placila` z `Idempotency-Key` | 201 + Location | 400, 422 |
| eno plačilo | `GET /v1/placila/{id}` | 200 | 404 |

Plačilo je simulirano in vedno uspe. Ponovljena zahteva z istim `Idempotency-Key` ne ustvari novega plačila, ampak vrne odgovor prvega.

## Primeri

### Seznam restavracij

```http
GET /v1/restavracije?kuhinja=italijanska&limit=2
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "items": [
    {
      "id": "r-1",
      "ime": "Pizzeria Napoli",
      "kuhinja": "italijanska",
      "naslov": "Italijanska ulica 23",
      "povprecnaOcena": "4.6"
    },
    {
      "id": "r-4",
      "ime": "Trattoria Roma",
      "kuhinja": "italijanska",
      "naslov": "Italijanska ulica 24",
      "povprecnaOcena": "4.2"
    }
  ]
}
```

### Ustvari naročilo

```http
POST /v1/narocila
Content-Type: application/json

{
  "restavracijaId": "r-1",
  "postavke": [
    {"jedId": "j-12", "kolicina": 2},
    {"jedId": "j-15", "kolicina": 1}
  ],
  "naslovDostave": "Večna pot 113, Ljubljana"
}
```

```http
HTTP/1.1 201 Created
Location: /v1/narocila/n-981
Content-Type: application/json

{
  "id": "n-981",
  "restavracijaId": "r-1",
  "status": "CAKA_NA_SPREJEM",
  "postavke": [
    {"jedId": "j-12", "ime": "Margherita", "cena": {"znesek": "9.50", "valuta": "EUR"}, "kolicina": 2},
    {"jedId": "j-15", "ime": "Tiramisu", "cena": {"znesek": "5.00", "valuta": "EUR"}, "kolicina": 1}
  ],
  "skupaj": {"znesek": "24.00", "valuta": "EUR"},
  "ustvarjeno": "2026-11-14T18:30:00Z"
}
```

### Izvedi plačilo (narocila → placila)

```http
POST /v1/placila
Content-Type: application/json
Idempotency-Key: 8e03978e-40d5-4c1a-9f3b-2a7d1e6c0b42

{
  "narociloId": "n-981",
  "znesek": {"znesek": "24.00", "valuta": "EUR"}
}
```

```http
HTTP/1.1 201 Created
Location: /v1/placila/p-55
Content-Type: application/json

{
  "id": "p-55",
  "narociloId": "n-981",
  "znesek": {"znesek": "24.00", "valuta": "EUR"},
  "status" : "USPESNO",
  "transakcijaId": "tx-3f9c2e1a"
}
```

## Problemi

| Tip | Status | Kdaj |
|---|---|---|
| `.../zahteva` | 400 | zahteve ni mogoče razčleniti (neveljaven JSON, napačen parameter) |
| `.../ni-najden` | 404 | vir ne obstaja |
| `.../jed-ni-na-voljo` | 409 | jed v naročilu trenutno ni na voljo |
| `.../slab-prehod` | 409 | prehod statusa naročila iz trenutnega stanja ni dovoljen |
| `.../ze-ocenjeno` | 409 | naročilo je že ocenjeno |
| `.../ni-dostavljeno` | 409 | ocena naročila, ki še ni dostavljeno |
| `.../verzija` | 412 | `If-Match` se ne ujema s trenutno verzijo vira |
| `.../vnos` | 422 | podatki kršijo pravila vsebine - vse napake v polju `errors` |
| `.../obvezen-pogoj` | 428 | v zahtevi ni bilo pogoja, ki je bil obvezen (`If-Match`) |
| `.../placila-nedosegljiva` | 503 | storitev placila ne odgovori |
| `.../katalog-nedosegljiv` | 503 | storitev katalog ne odgovori |

## Parametri zbirk

- `GET /v1/restavracije`: `kuhinja`, `offset`, `limit`
- `GET /v1/narocila`: `status`, `restavracijaId`, `offset`, `limit`
