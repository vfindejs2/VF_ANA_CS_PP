# CS-MPG-ANALYZY-E1

## Ucel

Business analyza a konzultacni podpora projektu Cyklicke svozy MP SK (Etapa 1). Projekt pokryva integrace, datove modely a prurezovou analytickou podporu pro PP, RP, FOB a vazby na HEN.

## Stav

- Stav: aktivni
- Vlastnik: doplnit
- Posledni aktualizace: 2026-04-07

## Rozsah

- Ve scope je analyza pozadavku, navrh datovych modelu, mapovani entit a business zadani pro integracni toky.
- Mimo scope je implementace, technicke provedeni a pozdejsi etapy mimo E1.

## Struktura slozky

- `01-vstupni-data-zadni/` = vstupni materialy a vytahy (Cilovy koncept apod.)
- `02-Datovy-model-PP/` = analyticke prace datoveho modelu PP
  - `PP-datovy-model-navrh-reseni-v1/` = hlavni vystupni dokument (BA navrh reseni, ER diagram)
  - `gap-analyza-navrhu-reseni.md` = porovnani navrhu vs. HEN realita
  - `hen-nove-entity-analyza.md` = rozbor entit z HEN (StudieRealizovatelnosti)
- `_archive/` = stare verze a pomocne analyzy (neni soucasti aktualniho stavu)

## Klicove soubory

- `PROGRESS.md` = aktualni stav a dalsi kroky
- `SPEC.md` = zadani, scope a akceptacni kriteria
- `02-Datovy-model-PP/PP-datovy-model-navrh-reseni-v1/datovy-model-navrh-reseni-123372122.md` = hlavni vystup — BA navrh datoveho modelu PP pro CS

## Dalsi krok

Review BA navrhu PP se stakeholdery a potvrzeni otevrenych bodu.